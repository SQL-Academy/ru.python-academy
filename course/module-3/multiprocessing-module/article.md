# Многопроцессорность в Python: Модуль multiprocessing

В предыдущей статье мы исследовали многопоточность с модулем `threading` и выяснили, что из-за Global Interpreter Lock (GIL) она не дает истинного параллелизма для CPU-bound задач в CPython. Чтобы по-настоящему задействовать несколько ядер процессора для вычислительно-интенсивных операций, Python предлагает модуль `multiprocessing`.

Модуль `multiprocessing` позволяет создавать и управлять процессами так же, как модуль `threading` управляет потоками. Ключевое отличие в том, что каждый процесс имеет свой собственный интерпретатор Python и свое пространство памяти, что позволяет им выполняться параллельно на разных ядрах CPU, обходя ограничения GIL.

## Введение в модуль multiprocessing

API модуля `multiprocessing` во многом схож с API `threading`. Это делает переход между ними относительно простым.

**Основные преимущества multiprocessing:**

-   Позволяет достичь настоящего параллелизма для CPU-bound задач.
-   Каждый процесс работает в своем изолированном пространстве памяти, что снижает риск конфликтов данных по сравнению с потоками.

**Основные недостатки:**

-   Создание процессов более ресурсоемко, чем создание потоков.
-   Обмен данными между процессами сложнее и медленнее, чем между потоками (требует сериализации/десериализации и специальных механизмов IPC - Inter-Process Communication).

## Создание и управление процессами

Класс `multiprocessing.Process` используется для создания новых процессов.

```python-executable
import multiprocessing
import os
import time

def worker_process(name):
    """Функция, выполняемая в отдельном процессе."""
    print(f"Процесс {name} (PID: {os.getpid()}): начинаю работу.")
    # time.sleep(2) # Имитация задержки (можно раскомментировать для наглядности)
    print(f"Процесс {name} (PID: {os.getpid()}): завершаю работу.")

if __name__ == "__main__":  # Важно для multiprocessing на некоторых платформах
    print(f"Основной процесс (PID: {os.getpid()})")

    # Создаем процессы
    process1 = multiprocessing.Process(target=worker_process, args=("Worker-A",))
    process2 = multiprocessing.Process(target=worker_process, args=("Worker-B",))

    # Запускаем процессы
    process1.start()
    process2.start()

    print("Все дочерние процессы запущены.")

    # Ожидаем завершения процессов
    process1.join()
    process2.join()

    print("Все дочерние процессы завершили свою работу.")
```

-   **`if __name__ == "__main__":`**: Эта проверка крайне важна при использовании `multiprocessing` на платформах, где дочерние процессы наследуют (или импортируют) родительский модуль (например, Windows). Без нее код верхнего уровня модуля будет выполняться в каждом дочернем процессе, что может привести к рекурсивному созданию процессов и ошибкам.
-   `os.getpid()`: Возвращает идентификатор текущего процесса.

## Обмен данными между процессами

Поскольку процессы имеют изолированную память, для обмена данными между ними требуются специальные механизмы.

### 1. Канал (Pipe)

`multiprocessing.Pipe()` создает пару объектов `Connection`, представляющих два конца канала. По умолчанию канал является двунаправленным. Каждый объект `Connection` имеет методы `send()` и `recv()`.

```python-executable
import multiprocessing
import time

def sender(conn):
    print("Отправитель: отправляю данные.")
    conn.send("Привет от отправителя") # Одно сообщение
    conn.close()
    print("Отправитель: данные отправлены и канал закрыт.")

def receiver(conn):
    print("Получатель: ожидаю данные...")
    msg = conn.recv()
    print(f"Получатель: получил \"{msg}\"")
    conn.close()
    print("Получатель: канал закрыт.")

if __name__ == "__main__":
    parent_conn, child_conn = multiprocessing.Pipe()

    p_sender = multiprocessing.Process(target=sender, args=(parent_conn,))
    p_receiver = multiprocessing.Process(target=receiver, args=(child_conn,))

    p_sender.start()
    p_receiver.start()

    p_sender.join()
    p_receiver.join()
    print("Обмен через Pipe завершен.")
```

`Pipe` хорошо подходит для простой двусторонней связи между двумя процессами.

### 2. Очередь (Queue)

`multiprocessing.Queue` очень похожа на `queue.Queue` из модуля `queue`, но предназначена для работы с процессами. Она потоко- и процессо-безопасна.

```python-executable
import multiprocessing
import time
import random

def producer_proc(q):
    for i in range(3):
        item = f"Элемент-{i}"
        time.sleep(random.uniform(0.1, 0.2)) # Небольшая случайная задержка
        q.put(item)
        print(f"Производитель: добавил {item} в очередь.")
    q.put(None) # Сигнал окончания для потребителя

def consumer_proc(name, q):
    while True:
        item = q.get()
        if item is None:
            print(f"{name}: получил сигнал None, завершаю.")
            break
        print(f"{name}: обработал {item}")
        time.sleep(random.uniform(0.1, 0.3))

if __name__ == "__main__":
    q = multiprocessing.Queue()

    p_prod = multiprocessing.Process(target=producer_proc, args=(q,))
    p_cons1 = multiprocessing.Process(target=consumer_proc, args=("Потребитель-A", q))

    p_prod.start()
    p_cons1.start()

    p_prod.join()
    p_cons1.join()
    print("Обмен через Queue завершен.")
```

### 3. Разделяемая память (Value и Array)

`Value` и `Array` позволяют разделять простые данные (числа, строки, массивы) между процессами. Доступ к ним должен синхронизироваться с помощью блокировок (`multiprocessing.Lock`), если несколько процессов могут их изменять.

```python-executable
import multiprocessing

def worker_value(num, lock):
    for _ in range(500):
        with lock:
            num.value += 1

def worker_array(arr, index, lock):
    with lock:
        arr[index] -= index * 0.5 # Пример операции над элементом массива

if __name__ == "__main__":
    lock = multiprocessing.Lock()
    shared_num = multiprocessing.Value('i', 0) # 'i' - тип integer
    shared_array = multiprocessing.Array('d', [10.0, 20.0, 30.0]) # 'd' - тип double

    processes = []
    # Процессы для Value
    for _ in range(2):
        p = multiprocessing.Process(target=worker_value, args=(shared_num, lock))
        processes.append(p)
        p.start()

    # Процессы для Array
    for i in range(len(shared_array)):
        p = multiprocessing.Process(target=worker_array, args=(shared_array, i, lock))
        processes.append(p)
        p.start()

    for p in processes:
        p.join()

    print(f"Shared number: {shared_num.value}") # Ожидается 1000
    print(f"Shared array: {list(shared_array)}")
```

Типы данных для `Value` и `Array` указываются с помощью кодов типов (typecodes), аналогичных модулю `array`.

### 4. Менеджеры (Manager)

Менеджеры предоставляют способ создания разделяемых объектов, которые могут быть более сложными, чем простые `Value` или `Array`. Менеджер запускает серверный процесс, который управляет этими объектами, а другие процессы получают прокси-объекты для доступа к ним.

Поддерживаются списки, словари, пространства имен, блокировки, очереди и т.д.

```python-executable
import multiprocessing

def worker_manager_dict(shared_dict, key, value):
    shared_dict[key] = value
    print(f"Процесс {key}: установил {key}={value}")

def worker_manager_list(shared_list, value):
    shared_list.append(value)
    print(f"Процесс добавил {value} в список")

if __name__ == "__main__":
    with multiprocessing.Manager() as manager:
        shared_dict = manager.dict()
        shared_list = manager.list()

        processes = []
        # Демонстрация для словаря
        for i in range(2):
            p = multiprocessing.Process(target=worker_manager_dict, args=(shared_dict, f"key{i}", i*10))
            processes.append(p)
            p.start()

        # Демонстрация для списка
        for i in range(2):
            p = multiprocessing.Process(target=worker_manager_list, args=(shared_list, f"item_{i}"))
            processes.append(p)
            p.start()

        for p in processes:
            p.join()

        print(f"Разделяемый словарь: {dict(shared_dict)}")
        print(f"Разделяемый список: {list(shared_list)}")
```

Менеджеры более гибкие, но и более медленные по сравнению с `Value`/`Array` из-за накладных расходов на IPC.

## Пулы процессов (Pool)

Класс `multiprocessing.Pool` предоставляет удобный способ распределения задач между несколькими рабочими процессами.

-   `pool.map(func, iterable)`: Применяет функцию `func` к каждому элементу `iterable` и возвращает список результатов. Блокирует до завершения всех задач.
-   `pool.apply_async(func, args)`: Асинхронно выполняет `func(*args)`. Возвращает объект `AsyncResult`, у которого можно вызвать `get()` для получения результата (блокирующий вызов).
-   `pool.close()`: Запрещает добавление новых задач в пул.
-   `pool.join()`: Ожидает завершения всех рабочих процессов.

```python-executable
import multiprocessing
import time

def square(x):
    time.sleep(0.1) # Имитация вычислений
    return x * x

if __name__ == "__main__":
    # Используем контекстный менеджер для автоматического close() и join()
    with multiprocessing.Pool(processes=2) as pool:
        numbers = list(range(5))

        # Пример с map
        print("Используем pool.map():")
        results_map = pool.map(square, numbers)
        print(f"Результаты (map): {results_map}")

        # Пример с apply_async
        print("\nИспользуем pool.apply_async():")
        async_results = [pool.apply_async(square, (num,)) for num in numbers]
        results_apply_async = [res.get(timeout=1) for res in async_results]
        print(f"Результаты (apply_async): {results_apply_async}")

    print("Работа с пулом завершена.")
```

## Сравнение threading и multiprocessing

| Аспект                 | `threading`                               | `multiprocessing`                              |
| ---------------------- | ----------------------------------------- | ---------------------------------------------- |
| **Параллелизм CPU**    | Ограничен GIL (нет для CPU-bound)         | Истинный параллелизм (обходит GIL)             |
| **I/O-bound задачи**   | Эффективен (GIL освобождается)            | Также эффективен, но больше накладных расходов |
| **Память**             | Разделяемая память                        | Изолированная память для каждого процесса      |
| **Обмен данными**      | Простой (через общие переменные + синхр.) | Сложнее (Pipe, Queue, Value, Array, Manager)   |
| **Накладные расходы**  | Низкие (создание потока дешевле)          | Высокие (создание процесса дороже)             |
| **Отказоустойчивость** | Сбой в потоке может уронить весь процесс  | Сбой в процессе обычно не влияет на другие     |

**Когда что выбирать:**

-   **`threading`**: Для I/O-bound задач, где важны низкие накладные расходы и простой обмен данными, и где GIL не является узким местом.
-   **`multiprocessing`**: Для CPU-bound задач, требующих интенсивных вычислений, которые можно распараллелить на несколько ядер.

## Что дальше?

Мы изучили, как модуль `multiprocessing` позволяет достичь настоящего параллелизма в Python, обойдя ограничения GIL. В следующей статье мы перейдем к совершенно другому подходу к конкурентности — асинхронному программированию с использованием `asyncio`.

---

**Какое основное преимущество модуля `multiprocessing` перед `threading` для CPU-bound задач?**
