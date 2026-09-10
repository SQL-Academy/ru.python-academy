---
meta:
    title: "Потоки и процессы в Python: threading и multiprocessing"
    description: "Модуль threading для I/O-bound задач и multiprocessing для CPU-bound. Создание, синхронизация через Lock и Queue, выбор подхода."
---

# Потоки и процессы в Python

Восемь тяжёлых вычислений через `ThreadPoolExecutor` считаются секунд двенадцать. Меняете в нём одно слово, `Thread` на `Process`, и те же восемь укладываются в три-четыре. В этом зазоре и живёт вся разница между потоками и процессами.

Расклад вы помните из вводной статьи: **потоки** (`threading`) для I/O, **процессы** (`multiprocessing`) для вычислений. Здесь разберём оба модуля. API у них почти одинаковый: выучили один — считайте, что выучили оба.

![Иллюстрация: слева панель threading с одним Process, внутри которого Thread 1-4 с разделяемой памятью и иконкой GIL; справа multiprocessing с тремя отдельными процессами, у каждого свой Python и память, под ними ядра CPU; подпись: потоки делят память и GIL, процессы изолированы и истинно параллельны](https://python-academy.org/static/guidePage/threading-and-multiprocessing/threads-vs-processes-ru.webp "Потоки делят память внутри одного процесса (GIL), процессы изолированы и могут идти на разных ядрах CPU")

## threading: потоки внутри одного процесса

Создаём поток через `threading.Thread`, передавая ему функцию-цель и аргументы:

```python
import threading
import time

def worker(name, sleep_time):
    print(f"Поток {name}: засыпаю на {sleep_time} с")
    time.sleep(sleep_time)
    print(f"Поток {name}: завершён")

t1 = threading.Thread(target=worker, args=("A", 2))
t2 = threading.Thread(target=worker, args=("B", 1))

t1.start()                # запускаем
t2.start()
t1.join()                 # ждём завершения
t2.join()
print("Все потоки завершены")
```

- `target` — функция, которую выполняет поток
- `args` — кортеж аргументов
- `start()` — запускает поток
- `join()` — блокирует основной поток до завершения этого

Если запустить, увидим примерно такой вывод:

```text
Поток A: засыпаю на 2 с
Поток B: засыпаю на 1 с
Поток B: завершён
Поток A: завершён
Все потоки завершены
```

Поток B стартовал вторым, но завершился первым — его `sleep` короче, и `t1.join()` ждёт именно завершения A. Это и есть конкурентное выполнение: потоки идут одновременно, порядок завершения определяется их работой, а не порядком запуска.

## Защита общих данных: Lock

Потоки делят память. Если два потока меняют одну переменную без защиты, **результат непредсказуем** (race condition). Посмотрим: пять потоков увеличивают счётчик по миллиону раз каждый, значит должно получиться 5 000 000.

```python
import threading

counter = 0

def increment():
    global counter
    for _ in range(1_000_000):
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)        # например 3137095 — и при каждом запуске другое число
```

Ждали 5 000 000, а получили меньше — и в следующий раз число будет другим. Дело в том, что `counter += 1` не одно действие, а три (прочитать значение, прибавить единицу, записать обратно), и потоки успевают влезть между шагами, затирая чужие инкременты. Защита — `Lock`: пока один поток внутри `with lock`, остальные ждут своей очереди.

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(1_000_000):
        with lock:                # автоматический acquire/release
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)        # 5000000 — теперь всегда верно
```

`with lock:` — стандартный способ работы: он гарантирует освобождение блокировки даже при исключении. Кроме `Lock` в `threading` есть `Event`, `Semaphore`, `Condition`, `RLock`. На практике в 90% случаев хватает `Lock` и `Queue` (см. ниже). Остальное нужно для нетривиальной координации.

## Обмен данными между потоками: queue.Queue

Менять общие переменные напрямую опасно: приходится расставлять `Lock` везде. Чище и безопаснее передавать данные через **потокобезопасную очередь** из модуля `queue`:

```python
import threading
import queue
import time

q = queue.Queue()

def producer():
    for i in range(5):
        q.put(f"item-{i}")
        time.sleep(0.1)
    q.put(None)               # сигнал «больше не будет»

def consumer():
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Получил {item}")

t_prod = threading.Thread(target=producer)
t_cons = threading.Thread(target=consumer)
t_prod.start()
t_cons.start()
t_prod.join()
t_cons.join()
```

`q.put()` блокируется если очередь переполнена (для ограниченных), `q.get()` блокируется если пуста. Внутри уже есть все нужные блокировки.

`q.put(None)` в конце producer-а это **соглашение** между producer и consumer: «данных больше не будет». В `Queue` нет встроенного сигнала «конец потока», поэтому договариваются вручную, и `None` это просто наиболее частый выбор. Можно использовать любое значение, которое не может прийти как валидные данные.

## multiprocessing: настоящий параллелизм

API почти идентичен `threading`, но вместо потоков создаются отдельные процессы. Каждый со своим интерпретатором, своей памятью, своим GIL. Несколько процессов реально работают одновременно на разных ядрах.

```python
import multiprocessing
import time

def heavy_calc(n):
    print(f"Процесс считает для n={n}")
    total = sum(i * i for i in range(n))
    return total

if __name__ == "__main__":                     # обязательная защита
    p1 = multiprocessing.Process(target=heavy_calc, args=(10_000_000,))
    p2 = multiprocessing.Process(target=heavy_calc, args=(10_000_000,))

    p1.start()
    p2.start()
    p1.join()
    p2.join()
    print("Оба процесса завершились")
```

**`if __name__ == "__main__":`** обязательна на Windows и macOS. Без неё дочерние процессы будут пытаться запустить весь модуль заново и упадут в бесконечную рекурсию создания процессов. Привыкайте сразу.

## Обмен данными между процессами: Queue

Процессы изолированы: у каждого своя память, общие переменные не работают. Передача данных идёт через специальный механизм. Самый удобный это `multiprocessing.Queue` с тем же API, что и `queue.Queue`:

```python
import multiprocessing

def producer(q):
    for i in range(3):
        q.put(f"item-{i}")
    q.put(None)

def consumer(q):
    while True:
        item = q.get()
        if item is None:
            break
        print(f"Получил {item}")

if __name__ == "__main__":
    q = multiprocessing.Queue()
    p1 = multiprocessing.Process(target=producer, args=(q,))
    p2 = multiprocessing.Process(target=consumer, args=(q,))
    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

Кроме `Queue` есть `Pipe` (для двух процессов), `Value`/`Array` (общие примитивные данные) и `Manager` (общие списки/словари через серверный процесс). Это всё нужно редко: `Queue` + `Pool` (ниже) покрывают большинство случаев.

## Пул процессов для CPU-bound задач

Создавать процессы вручную для каждой задачи накладно. `multiprocessing.Pool` создаёт пул из N процессов и распределяет между ними задачи:

```python
import multiprocessing

def heavy_square(x):
    # имитация тяжёлых вычислений, нагружающих CPU
    return sum(i * i for i in range(x * 100_000))

if __name__ == "__main__":
    with multiprocessing.Pool(processes=4) as pool:
        results = pool.map(heavy_square, range(1, 9))
    print(results)
```

`pool.map(func, items)` применяет функцию к каждому элементу из `items`, распределяя работу между процессами в пуле. Контекстный менеджер сам закроет пул и дождётся всех процессов.

## concurrent.futures: одинаковый API для потоков и процессов

Модуль `concurrent.futures` даёт высокоуровневую обёртку над обоими подходами. Один и тот же код работает и с потоками, и с процессами, меняется только класс executor:

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def task(x):
    return x * x

# if __name__ нужен из-за ProcessPoolExecutor: он запускает процессы
if __name__ == "__main__":
    # Для I/O-bound: потоки
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(task, range(10)))

    # Для CPU-bound: процессы — меняется только класс executor
    with ProcessPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(task, range(10)))
```

Это самый практичный способ распараллелить простые задачи. На современных проектах `concurrent.futures` встречается чаще, чем прямые `threading.Thread` или `multiprocessing.Process`.

## Когда что брать

| Задача                                                 | Инструмент                                       |
| ------------------------------------------------------ | ------------------------------------------------ |
| Простые I/O-bound в существующем синхронном коде       | `ThreadPoolExecutor`                             |
| Когда нужно вручную управлять состоянием (Lock, Queue) | `threading` напрямую                             |
| CPU-bound вычисления                                   | `ProcessPoolExecutor` или `multiprocessing.Pool` |
| Тысячи сетевых соединений                              | **asyncio** (следующие статьи)                   |

`asyncio` лучше для I/O-bound на больших объёмах: один поток, минимальные накладные расходы на переключение. Но требует переписать код в `async`-стиле.

## Проверка понимания

**Почему `multiprocessing` подходит для CPU-bound задач, а `threading` нет?**

1. Процессы быстрее запускаются, чем потоки — Наоборот, создание процесса дороже, чем создание потока. Преимущество multiprocessing в другом.

2. **Правильный ответ:** У каждого процесса свой Python-интерпретатор и свой GIL, поэтому процессы реально работают параллельно на разных ядрах CPU — Верно. GIL разрешает выполнять байткод Python только одному потоку в рамках процесса. Несколько процессов запускают несколько интерпретаторов, каждый со своим GIL и своим ядром: это и есть истинный параллелизм.

3. multiprocessing использует асинхронность, а threading нет — Ни один из них не использует асинхронность. Это совершенно отдельный подход (asyncio).

4. threading нельзя использовать на многоядерных процессорах — threading работает на любом процессоре. Ограничение в том, что из-за GIL потоки не выполняют Python-код одновременно, но для I/O-bound задач это не мешает, так как GIL отпускается во время ожидания.

В следующих двух статьях — **asyncio**: третий и самый эффективный для I/O способ организовать конкурентное выполнение. Особенно для веб-серверов, ботов и работы с API.
