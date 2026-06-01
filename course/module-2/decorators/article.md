# Декораторы в Python

Допустим, у нас есть несколько функций и мы хотим во всех замерить время выполнения. Можно скопировать код тайминга в каждую:

```python
def slow_function():
    start = time.time()
    # ... основная работа
    elapsed = time.time() - start
    print(f"slow_function: {elapsed:.4f} c")

def another_function():
    start = time.time()
    # ... основная работа
    elapsed = time.time() - start
    print(f"another_function: {elapsed:.4f} c")
```

Работает, но через неделю мы захотим поменять формат логирования, и придётся редактировать **все** функции по очереди. Плюс основная логика теряется в служебном коде.

Декораторы решают именно это: вы пишете обёртку один раз, и навешиваете её на любую функцию короткой строкой `@обёртка` над определением.

## Базовый синтаксис

Декоратор — функция, которая берёт другую функцию и возвращает её «обёрнутую» версию (с добавленным поведением):

```python-executable
def my_decorator(func):
    def wrapper():
        print("До вызова функции")
        func()
        print("После вызова функции")
    return wrapper

@my_decorator
def say_hello():
    print("Привет, мир!")

say_hello()
# Вывод:
# До вызова функции
# Привет, мир!
# После вызова функции
```

Запись `@my_decorator` над `say_hello` это сахар над таким эквивалентом:

```python
say_hello = my_decorator(say_hello)
```

То есть мы переопределяем `say_hello` на новую функцию (которую вернул декоратор). Никакой магии, обычное переприсваивание.

## Декоратор с аргументами функции

Если оборачиваемая функция принимает аргументы, обёртка должна их пробрасывать. Универсальный приём: `*args, **kwargs`:

```python-executable
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("До вызова")
        result = func(*args, **kwargs)
        print("После вызова")
        return result
    return wrapper

@my_decorator
def add(a, b):
    return a + b

print(add(5, 3))
# Вывод:
# До вызова
# После вызова
# 8
```

`*args, **kwargs` означает «приму любые позиционные и именованные аргументы», и `func(*args, **kwargs)` пробрасывает их дальше. Этим приёмом декоратор становится универсальным — работает с любой функцией.

## Практический декоратор: тайминг

Тот самый тайминг, ради которого мы всё начали:

```python-executable
import time

def timing(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        print(f"{func.__name__}: {elapsed:.4f} c")
        return result
    return wrapper

@timing
def calculate_sum(n):
    return sum(range(n))

calculate_sum(1_000_000)
# Вывод: calculate_sum: 0.0462 c
```

Теперь добавить тайминг к любой функции это одна строка `@timing` сверху. Захотим поменять формат вывода: правим одну функцию `timing`, а не каждую функцию в проекте.

## functools.wraps: сохранение имени и docstring

У наивного декоратора есть незаметный побочный эффект: обёрнутая функция «теряет» своё имя и документацию, потому что снаружи вы видите уже `wrapper`, а не оригинал:

```python-executable
def timing(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@timing
def calculate_sum(n):
    """Считает сумму чисел от 0 до n."""
    return sum(range(n))

print(calculate_sum.__name__)
# Вывод: wrapper
print(calculate_sum.__doc__)
# Вывод: None
```

В реальном коде это ломает отладку, логирование и работу IDE. Лечится одной строкой — декоратором `@functools.wraps(func)` на `wrapper`:

```python-executable
from functools import wraps

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@timing
def calculate_sum(n):
    """Считает сумму чисел от 0 до n."""
    return sum(range(n))

print(calculate_sum.__name__)
# Вывод: calculate_sum
print(calculate_sum.__doc__)
# Вывод: Считает сумму чисел от 0 до n.
```

Правило: пишете свой декоратор, всегда оборачивайте внутреннюю функцию через `@wraps(func)`. Это бесплатно и сохраняет интроспекцию.

## Декоратор с параметрами

Иногда хочется передать настройки самому декоратору, например «повтори вызов N раз». Это требует ещё одного уровня: внешняя функция принимает параметр, внутри возвращает «настоящий» декоратор:

```python-executable
from functools import wraps

def repeat(n=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(n):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(n=3)
def say_hi(name):
    print(f"Привет, {name}!")

say_hi("Анна")
# Вывод:
# Привет, Анна!
# Привет, Анна!
# Привет, Анна!
```

Три уровня вложенности кажутся пугающими, но логика простая:

1. `repeat(n)` принимает параметр декоратора и возвращает обычный декоратор.
2. `decorator(func)` принимает функцию и возвращает обёртку.
3. `wrapper(*args, **kwargs)` обрабатывает реальный вызов.

## Цепочка декораторов

Декораторы можно применять несколько. Они применяются **снизу вверх**: ближайший к функции идёт первым:

```python-executable
def bold(func):
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def format_text(text):
    return text

print(format_text("Привет, мир!"))
# Вывод: <b><i>Привет, мир!</i></b>
```

`@italic` применяется к `format_text` первым, получается «italic-функция». Потом `@bold` оборачивает её снаружи, получается «bold(italic(format_text))». Поэтому сначала закрывается `<i>`, потом `<b>`.

## Вы уже встречали декораторы

Когда мы разбирали инкапсуляцию, мы использовали `@property` и `@balance.setter` — это и есть декораторы из стандартной библиотеки. `@property` берёт функцию-геттер и превращает её в «вычисляемый атрибут». Никакой магии: тот же механизм, что мы сейчас разбираем, только применённый в контексте классов.

## Где декораторы живут в реальности

Несколько мест, где вы их встретите чаще всего:

**Web-фреймворки (Flask, FastAPI, Django).** Привязка URL к функции-обработчику:

```python
@app.route('/home')
def home():
    return "Главная страница"
```

`@app.route` регистрирует функцию в роутере фреймворка — без декоратора пришлось бы вручную писать `app.add_url_rule(...)` для каждого эндпоинта.

**Кэширование.** Сохранение результатов чтобы не пересчитывать одно и то же:

```python-executable
from functools import wraps

def memoize(func):
    cache = {}
    @wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(30))
# Вывод: 832040
```

Без `@memoize` `fib(30)` пересчитывал бы одно и то же миллион раз и подвисал. С кэшем работает мгновенно. В стандартной библиотеке такой декоратор уже есть: `from functools import lru_cache`.

**Тесты.** В pytest `@pytest.fixture`, `@pytest.mark.parametrize` — это декораторы, которые превращают обычную функцию в фикстуру или параметризованный тест.

## Что дальше?

Декораторы это рабочая лошадка Python: библиотеки и фреймворки строятся на них (Flask-роуты, pytest-фикстуры, dataclasses, type-checkers). Когда вы понимаете, что `@что-то` это просто `func = что-то(func)`, никакой магии в этих библиотеках больше нет — только композиция функций.

---

**В чём основное назначение декораторов в Python?**

