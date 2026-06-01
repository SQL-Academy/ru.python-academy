# Потоки и процессы в Python

Из вводной статьи: для I/O-bound задач нужны **потоки** (`threading`), для CPU-bound — **процессы** (`multiprocessing`). Здесь разберём оба модуля. API у них почти одинаковый: выучили один — считайте, что выучили оба.

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

-   `target` — функция, которую выполняет поток
-   `args` — кортеж аргументов
-   `start()` — запускает поток
-   `join()` — блокирует основной поток до завершения этого

Если запустить, увидим примерно такой вывод:

```
Поток A: засыпаю на 2 с
Поток B: засыпаю на 1 с
Поток B: завершён
Поток A: завершён
Все потоки завершены
```

Поток B стартовал вторым, но завершился первым — его `sleep` короче, и `t1.join()` ждёт именно завершения A. Это и есть конкурентное выполнение: потоки идут одновременно, порядок завершения определяется их работой, а не порядком запуска.

## Защита общих данных: Lock

Потоки делят память. Если два потока меняют одну переменную, **результат непредсказуем** (race condition). Защита — `Lock`:

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100_000):
        with lock:                # автоматический acquire/release
            counter += 1

threads = [threading.Thread(target=increment) for _ in range(5)]
for t in threads:
    t.start()
for t in threads:
    t.join()

print(counter)        # 500000 — корректно
```

Без `lock` итог будет случайным числом меньше 500_000: потоки «перетирают» инкременты друг друга. Контекстный менеджер `with lock:` это стандартный способ работы, он гарантирует освобождение даже при исключении.

Кроме `Lock` в `threading` есть `Event`, `Semaphore`, `Condition`, `RLock`. На практике в 90% случаев хватает `Lock` и `Queue` (см. ниже). Остальное нужно для нетривиальной координации.

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

# Для I/O-bound: потоки
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(task, range(10)))

# Для CPU-bound: процессы
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(task, range(10)))
```

Это самый практичный способ распараллелить простые задачи. На современных проектах `concurrent.futures` встречается чаще, чем прямые `threading.Thread` или `multiprocessing.Process`.

## Когда что брать

| Задача | Инструмент |
| --- | --- |
| Простые I/O-bound в существующем синхронном коде | `ThreadPoolExecutor` |
| Когда нужно вручную управлять состоянием (Lock, Queue) | `threading` напрямую |
| CPU-bound вычисления | `ProcessPoolExecutor` или `multiprocessing.Pool` |
| Тысячи сетевых соединений | **asyncio** (следующие статьи) |

`asyncio` лучше для I/O-bound на больших объёмах: один поток, минимальные накладные расходы на переключение. Но требует переписать код в `async`-стиле.

## Что дальше?

В следующих двух статьях — **asyncio**: третий и самый эффективный для I/O способ организовать конкурентное выполнение. Особенно для веб-серверов, ботов и работы с API.

---

**Почему `multiprocessing` подходит для CPU-bound задач, а `threading` нет?**

