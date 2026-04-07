# Продвинутое asyncio в Python: Ключевые инструменты

В предыдущей статье мы познакомились с основами `asyncio`. Теперь рассмотрим ключевые продвинутые концепции, которые часто используются при написании реальных асинхронных приложений: асинхронные генераторы и контекстные менеджеры, очереди для обмена данными, базовые примитивы синхронизации и интеграция с блокирующим кодом.

## Асинхронные генераторы (async for)

Подобно обычным генераторам, **асинхронные генераторы** позволяют итерировать по последовательности данных асинхронно, не загружая всю ее в память. Они определяются с `async def` и используют `yield`. Перебираются с помощью `async for`.

```python-executable
import asyncio

async def async_number_generator(limit):
    for i in range(limit):
        await asyncio.sleep(0.5) # Имитация асинхронной операции
        yield i

async def main_gen():
    print("Начинаем перебор асинхронного генератора:")
    async for number in async_number_generator(5):
        print(f"Получено число: {number}")

if __name__ == "__main__":
    # Убедитесь, что пример запускается из основного потока
    # или используйте соответствующий метод запуска asyncio для вашей среды.
    try:
        asyncio.run(main_gen())
    except RuntimeError as e:
        # В некоторых средах (как Jupyter) может потребоваться get_event_loop()
        if "cannot run current event loop" in str(e):
             print("Запуск через asyncio.run() не удался. Попробуйте другой способ запуска цикла событий.")
        else:
             raise e

```

`async for` будет ожидать (`await`) получения каждого следующего элемента от асинхронного генератора.

## Асинхронные контекстные менеджеры (async with)

Контекстные менеджеры (`with`) полезны для управления ресурсами. **Асинхронные контекстные менеджеры** расширяют это для асинхронных операций. Они реализуют методы `__aenter__` и `__aexit__` (которые могут быть `async`) и используются с `async with`.

```python-executable
import asyncio

class AsyncResource:
    def __init__(self, name):
        self.name = name

    async def __aenter__(self): # Асинхронный вход
        print(f"Ресурс '{self.name}': вход (получение ресурса...)")
        await asyncio.sleep(0.5) # Имитация асинхронной операции
        print(f"Ресурс '{self.name}': получен.")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb): # Асинхронный выход
        print(f"Ресурс '{self.name}': выход (освобождение ресурса...)")
        await asyncio.sleep(0.5) # Имитация асинхронной операции
        print(f"Ресурс '{self.name}': освобожден.")
        if exc_type:
            print(f"Произошло исключение: {exc_val}")
        # return True # Если True, исключение будет подавлено

async def use_async_resource():
    async with AsyncResource("DB_Connection") as resource:
        print(f"Используем ресурс '{resource.name}'...")
        await asyncio.sleep(1)
        print("Завершили использование ресурса.")

# Блок if __name__ == "__main__" и try/except аналогичен предыдущему примеру
if __name__ == "__main__":
    try:
        asyncio.run(use_async_resource())
    except RuntimeError as e:
        if "cannot run current event loop" in str(e):
             print("Запуск через asyncio.run() не удался. Попробуйте другой способ запуска цикла событий.")
        else:
             raise e
```

## Работа с сетевыми протоколами и Streams

`asyncio` предоставляет низкоуровневый API для работы с сетевыми потоками данных (TCP) через `StreamReader` и `StreamWriter`, которые можно получить с помощью `asyncio.open_connection` и `asyncio.start_server`. Это позволяет создавать асинхронных TCP-клиентов и серверов для любых протоколов.

Однако для стандартных протоколов, таких как HTTP/HTTPS, обычно удобнее использовать высокоуровневые библиотеки, построенные на основе `asyncio`, например:

-   **`aiohttp`**: Популярная библиотека для создания асинхронных HTTP клиентов и серверов.
-   **`httpx`**: Современный HTTP-клиент, который поддерживает как синхронные, так и асинхронные запросы.

Эти библиотеки абстрагируют детали работы со Streams, предоставляя более простой интерфейс для веб-взаимодействий.

## Асинхронные очереди (asyncio.Queue)

`asyncio.Queue` — это основной способ безопасного обмена данными между различными асинхронными задачами (`Task`) в рамках одного цикла событий. API похож на `queue.Queue`, но использует `await`.

-   `await queue.put(item)`: Добавить элемент.
-   `await queue.get()`: Извлечь элемент (ожидает, если очередь пуста).
-   `queue.task_done()` / `await queue.join()`: Для координации завершения обработки элементов.

```python-executable
import asyncio
import random

async def producer_async(q, n_items):
    for i in range(n_items):
        item = f"AsyncItem-{i}"
        await asyncio.sleep(random.uniform(0.1, 0.3))
        await q.put(item)
        print(f"Продюсер: добавил {item} (очередь: {q.qsize()})")

async def consumer_async(name, q):
    while True:
        item = await q.get()
        print(f"Потребитель {name}: получил {item}")
        await asyncio.sleep(random.uniform(0.2, 0.5)) # Имитация обработки
        q.task_done()
        print(f"Потребитель {name}: обработал {item}")

async def main_queue():
    q = asyncio.Queue(maxsize=3) # Очередь с ограничением размера

    # Запускаем продюсеров и потребителей как задачи
    producers = [asyncio.create_task(producer_async(q, 5)) for _ in range(2)]
    consumers = [asyncio.create_task(consumer_async(f"C-{i}", q)) for i in range(3)]

    # Ждем, пока продюсеры добавят все элементы
    await asyncio.gather(*producers)
    print("-- Продюсеры завершили --")

    # Ждем, пока потребители обработают все элементы в очереди
    await q.join()
    print("-- Все элементы обработаны --")

    # Аккуратно останавливаем потребителей (т.к. они в бесконечном цикле)
    for c in consumers:
        c.cancel()

    # Даем возможность задачам обработать отмену
    await asyncio.gather(*consumers, return_exceptions=True)
    print("-- Потребители остановлены --")

# Блок if __name__ == "__main__" и try/except аналогичен предыдущим примерам
if __name__ == "__main__":
    try:
        asyncio.run(main_queue())
    except RuntimeError as e:
        if "cannot run current event loop" in str(e):
             print("Запуск через asyncio.run() не удался. Попробуйте другой способ запуска цикла событий.")
        else:
             raise e
```

## Примитивы синхронизации в asyncio

Для координации работы корутин и защиты общих ресурсов `asyncio` предоставляет аналоги примитивов из `threading`, но адаптированные для асинхронности (они уступают управление циклу событий, а не блокируют поток).

Самый базовый — **`asyncio.Lock`**. Он гарантирует, что только одна корутина может выполнять код внутри блока `async with lock_async:`. Используется для защиты критических секций.

```python-executable
import asyncio

shared_counter_async = 0
lock_async = asyncio.Lock()

async def increment_async(n_times):
    global shared_counter_async
    for _ in range(n_times):
        async with lock_async: # Захватываем блокировку
            # Критическая секция: только одна корутина может быть здесь одновременно
            current_val = shared_counter_async
            await asyncio.sleep(0.001) # Имитация работы внутри секции
            shared_counter_async = current_val + 1
        # Блокировка освобождается автоматически при выходе из async with

async def main_lock_example():
    tasks = [increment_async(1000) for _ in range(5)] # Уменьшим количество для скорости
    await asyncio.gather(*tasks)
    print(f"Итоговый счетчик (ожидается 5000): {shared_counter_async}")

# Блок if __name__ == "__main__" и try/except аналогичен предыдущим примерам
if __name__ == "__main__":
    try:
        asyncio.run(main_lock_example())
    except RuntimeError as e:
        if "cannot run current event loop" in str(e):
             print("Запуск через asyncio.run() не удался. Попробуйте другой способ запуска цикла событий.")
        else:
             raise e
```

Другие примитивы, такие как `asyncio.Event` (для сигнализации между корутинами), `asyncio.Semaphore` (для ограничения одновременного доступа к ресурсу) и `asyncio.Condition` (для более сложной синхронизации), также доступны, но используются реже, чем `Lock` и `Queue`.

## Выполнение блокирующего кода в asyncio

Что делать, если нужно вызвать функцию, которая блокирует поток (например, старая библиотека или CPU-bound расчет), из асинхронного кода? Прямой вызов заблокирует весь цикл событий. Решение — `loop.run_in_executor()`.

Эта функция позволяет выполнить блокирующую функцию в отдельном потоке (по умолчанию используется `ThreadPoolExecutor`) или процессе (`ProcessPoolExecutor`), не останавливая цикл событий `asyncio`.

```python-executable
import asyncio
import time
import concurrent.futures

def blocking_io_operation(duration):
    print(f"[Поток {threading.current_thread().name}] Блокирующая операция: начинаю, сплю {duration} сек...")
    time.sleep(duration) # Обычный, блокирующий sleep
    print(f"[Поток {threading.current_thread().name}] Блокирующая операция: завершена.")
    return f"Результат от {duration} сек."

async def main_blocking():
    loop = asyncio.get_running_loop() # Получаем текущий цикл событий

    print("Запускаем блокирующую операцию в executor'е...")

    # Запускаем blocking_io_operation в стандартном ThreadPoolExecutor
    # Первый аргумент None означает использование executor'а по умолчанию
    result_future = loop.run_in_executor(None, blocking_io_operation, 2)

    # Пока блокирующая операция выполняется в другом потоке,
    # асинхронный код может продолжать работу:
    print("Асинхронный код выполняется ПАРАЛЛЕЛЬНО с блокирующей операцией...")
    await asyncio.sleep(1)
    print("Асинхронный код все еще работает...")

    # Ожидаем результат от executor'а
    result = await result_future
    print(f"Получен результат из executor'а: {result}")

# Блок if __name__ == "__main__" и try/except аналогичен предыдущим примерам
# Добавим импорт threading для вывода имени потока
import threading

if __name__ == "__main__":
    try:
        asyncio.run(main_blocking())
    except RuntimeError as e:
        if "cannot run current event loop" in str(e):
             print("Запуск через asyncio.run() не удался. Попробуйте другой способ запуска цикла событий.")
        else:
             raise e
```

## Что дальше?

Мы рассмотрели ключевые продвинутые инструменты `asyncio`: `async for`, `async with`, `asyncio.Queue`, `asyncio.Lock` и `run_in_executor`. Они составляют основу для построения большинства реальных асинхронных приложений на Python.

В заключительной статье мы сравним все рассмотренные подходы к конкурентности (`threading`, `multiprocessing`, `asyncio`), обсудим `concurrent.futures` и рассмотрим общие лучшие практики.

---

**Какая функция asyncio используется для безопасного выполнения блокирующего кода без остановки цикла событий?**
