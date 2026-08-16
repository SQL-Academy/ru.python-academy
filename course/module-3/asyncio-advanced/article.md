---
meta:
    title: "Продвинутый asyncio: очереди, синхронизация, executor"
    description: "asyncio.Queue для обмена между корутинами, asyncio.Lock для синхронизации, и главное это run_in_executor для блокирующего кода без остановки event loop."
---

# Продвинутый asyncio в Python

Асинхронный сервис держал тысячи соединений и вдруг замер весь, для всех разом. Причина в одной строке: кто-то вызвал синхронную `requests.get()`, и она на секунду заблокировала единственный поток event loop, а с ним и все остальные корутины.

Базовых средств asyncio из прошлой статьи (`async def`, `await`, `gather`, Tasks) здесь недостаточно. Для реальных приложений нужны ещё три инструмента: очереди между корутинами, синхронизация и, главное, запуск блокирующего кода без остановки event loop.

## asyncio.Queue: обмен данными между корутинами

В asyncio все корутины работают в одном потоке и в принципе могут делиться состоянием напрямую. Но для **производитель-потребитель** паттерна удобнее очередь:

```python
import asyncio

async def producer(q):
    for i in range(5):
        await q.put(f"item-{i}")
        await asyncio.sleep(0.1)
    await q.put(None)            # сигнал остановки

async def consumer(q):
    while True:
        item = await q.get()
        if item is None:
            break
        print(f"Получил {item}")

async def main():
    q = asyncio.Queue()
    await asyncio.gather(producer(q), consumer(q))

asyncio.run(main())
```

API такой же, как у `queue.Queue`, но методы здесь корутины (`await q.put`, `await q.get`). Очередь блокирует на пустом `get()` или переполненном `put()` (если задан `maxsize`), но не сам поток — она уступает управление event loop.

## asyncio.Lock: защита общего состояния

В asyncio переключение корутин происходит только на `await`. Если между двумя `await` есть критическая секция (где меняется общее состояние), переключение туда не вклинится. Но если внутри критической секции есть `await`, другая корутина может вмешаться.

```python
import asyncio

counter = 0
lock = asyncio.Lock()

async def increment():
    global counter
    async with lock:
        current = counter
        await asyncio.sleep(0.01)     # await ВНУТРИ критической секции
        counter = current + 1

async def main():
    await asyncio.gather(*(increment() for _ in range(100)))
    print(counter)        # 100 — корректно благодаря lock

asyncio.run(main())
```

Без `lock` несколько корутин прочитали бы одно и то же значение `current`, и итог был бы меньше 100. С `async with lock:` только одна корутина может находиться в критической секции одновременно.

В реальном asyncio-коде блокировки нужны **редко**, потому что большинство переменных живут внутри одной корутины. `Lock` пригодится, когда несколько корутин читают/пишут одну общую структуру или ресурс — например, общий счётчик активных подключений или кэш.

Кроме `Lock` есть `asyncio.Event`, `asyncio.Semaphore`, `asyncio.Condition` (API копирует `threading`, но операции через `await`).

## Блокирующий код в asyncio: run_in_executor

Вернёмся к аварии из начала главы. Правило, которое там нарушили: **в event loop нельзя вызывать блокирующие функции напрямую**. `time.sleep(2)`, `requests.get()`, тяжёлый расчёт останавливают его целиком.

Но иногда деваться некуда: нужна старая синхронная библиотека или CPU-bound расчёт. На этот случай есть `loop.run_in_executor()`: запустить блокирующую функцию в **отдельном потоке** (или процессе), пока event loop спокойно продолжает работу.

![Иллюстрация: event loop с кодом await blocking_task() слева; стрелка run_in_executor к Thread Pool Executor справа, где выполняется time.sleep(2); внизу другие корутины продолжают работать; стрелка возврата результата](https://python-academy.org/static/guidePage/asyncio-advanced/run-in-executor-ru.webp "run_in_executor отдаёт блокирующую функцию в пул потоков, чтобы не блокировать event loop")

```python
import asyncio
import time

def blocking_io():
    print("Блокирующая функция: засыпаю на 2с")
    time.sleep(2)                    # синхронный sleep
    return "готово"

async def main():
    loop = asyncio.get_running_loop()
    print("Запускаем блокирующую задачу в executor")

    # None = executor по умолчанию (ThreadPoolExecutor)
    future = loop.run_in_executor(None, blocking_io)

    # пока блокирующая задача работает, event loop свободен
    await asyncio.sleep(1)
    print("Event loop работает параллельно")

    result = await future
    print(f"Результат: {result}")

asyncio.run(main())
```

`run_in_executor(None, func, *args)` отдаёт `func(*args)` в стандартный `ThreadPoolExecutor` (тот самый, что мы видели в статье про потоки и процессы) и возвращает future, который можно `await`-ить.

Для CPU-bound кода можно передать `ProcessPoolExecutor` первым аргументом — функция уйдёт в отдельный процесс с собственным GIL.

## async-итерация и контекстные менеджеры

Если объект собирает данные постепенно (через сеть, например), он может быть **асинхронным итератором**: итерируется через `async for`:

```python
async for line in aiohttp_response:
    process(line)
```

Если ресурс надо открыть и закрыть асинхронно (соединение с БД), это **асинхронный контекстный менеджер** через `async with`:

```python
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.json()
```

Сами вы их пишете редко, это инструменты библиотек (`aiohttp`, `asyncpg`, `aioredis`). Достаточно знать, что они существуют и узнавать `async for` / `async with` в чужом коде.

## Сравнение трёх подходов

|                   | threading                 | multiprocessing      | asyncio                    |
| ----------------- | ------------------------- | -------------------- | -------------------------- |
| Параллелизм CPU   | нет (GIL)                 | да                   | нет (1 поток)              |
| I/O-bound         | хорошо                    | хорошо, но дорого    | отлично                    |
| Накладные расходы | низкие                    | высокие              | минимальные                |
| Память            | общая                     | изолированная        | общая (1 поток)            |
| Обмен данными     | переменные + Lock / Queue | Queue, Pipe, Manager | переменные / asyncio.Queue |
| Тысячи задач      | плохо                     | очень плохо          | прекрасно                  |

**Правило выбора:**

- Тысячи сетевых соединений, новые проекты → **asyncio**
- I/O в существующем синхронном коде без async-библиотек → **threading** или `ThreadPoolExecutor`
- Тяжёлые вычисления → **multiprocessing** или `ProcessPoolExecutor`
- В одном приложении часто всё это сочетается: asyncio как основной слой + `run_in_executor` с пулом потоков/процессов для блокирующих кусков.

## Несколько подводных камней

- **CPU-bound в asyncio** убивает event loop. Используйте `run_in_executor` с `ProcessPoolExecutor` для тяжёлых вычислений в async-коде.
- **Забытая `await`**: `asyncio.sleep(1)` без `await` ничего не делает (создаёт корутину и выбрасывает её). В современных IDE это подсвечивается.
- **Mix sync/async**: вызов `requests.get()` (синхронный) в asyncio блокирует всё. Используйте `aiohttp` / `httpx` для async-HTTP.
- **`if __name__ == "__main__":`** на Windows и macOS обязательна для `multiprocessing`, иначе процессы будут рекурсивно создавать сами себя.

## Проверка понимания

**Какой инструмент asyncio используется для безопасного запуска блокирующего кода, не останавливая event loop?**

1. asyncio.sleep() — asyncio.sleep() — неблокирующая пауза, она сама уже асинхронная.

2. asyncio.gather() — gather() запускает несколько корутин конкурентно. Для блокирующих синхронных функций он не подходит — они всё равно остановят event loop.

3. **Правильный ответ:** loop.run_in_executor() — Верно. run_in_executor отдаёт блокирующую функцию в отдельный поток (или процесс), и event loop продолжает работать с другими корутинами параллельно.

4. asyncio.create_task() — create_task() планирует корутину к выполнению в event loop. Но он не помогает с СИНХРОННЫМИ блокирующими функциями — те всё равно зависнут event loop.

На этом модуль конкурентности завершён. Карта из вводной статьи и матрица выше уже отвечают на вопрос «что брать», а на практике основная программа чаще всего живёт на asyncio, отдавая CPU-тяжёлые куски в process pool через `run_in_executor`.
