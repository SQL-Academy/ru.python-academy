# pytest: фикстуры и параметризация

В прошлой статье мы писали простые тесты-функции. Сейчас разберём две вещи, без которых тестирование быстро становится копипастой: **фикстуры** (общая подготовка для нескольких тестов) и **параметризация** (один тест на много входов).

## Фикстуры: общая подготовка

Часто тестам нужно одно и то же подготовленное состояние: пользователь с заполненными полями, открытый объект, тестовая база. Фикстура это функция-помощник, которая возвращает это состояние и автоматически передаётся в тест по имени. Декоратор `@pytest.fixture` говорит pytest «эта функция — фикстура, вызывай её для тестов, которые её запрашивают»:

```python
import pytest

@pytest.fixture
def alice():
    return {"id": 1, "name": "Alice", "email": "alice@example.com"}

def test_user_has_email(alice):         # имя фикстуры в аргументе
    assert "@" in alice["email"]

def test_user_id_is_int(alice):
    assert isinstance(alice["id"], int)
```

`pytest` сам видит, что `test_user_has_email` запрашивает фикстуру `alice`, вызывает её и передаёт результат в тест. Если фикстура не нужна — её можно просто не указывать в аргументах. Главное удобство: вы один раз описываете подготовку, потом используете в десятках тестов через один аргумент.

## yield: подготовка и очистка

Если нужно не только подготовить что-то, но и **прибрать после теста** (закрыть соединение, удалить временный файл), используйте `yield` вместо `return`:

```python
import os
import pytest

@pytest.fixture
def temp_file():
    path = "temp_test_file.txt"
    with open(path, "w") as f:
        f.write("hello")

    yield path           # тест получает path

    os.remove(path)       # выполнится после теста

def test_read(temp_file):
    with open(temp_file) as f:
        assert f.read() == "hello"
```

Код **до** `yield` выполняется перед тестом (setup), код **после** — после теста (teardown). Работает даже если тест упал.

## Scope: как часто пересоздавать

По умолчанию фикстура вызывается **для каждого теста**. Если подготовка дорогая (открыть подключение к БД, загрузить большой файл), можно сказать pytest «делай один раз на всю сессию» через `scope="session"`:

```python
@pytest.fixture(scope="session")
def db_connection():
    print("\nподключаемся к БД (один раз)")
    conn = {"status": "connected"}
    yield conn
    print("\nзакрываем соединение")
```

На практике 95% времени используется `function` (по умолчанию, один экземпляр на тест) и `session` (один на весь прогон). Есть ещё `class` и `module` — пригодятся в особых случаях, для старта они не нужны.

## conftest.py: общие фикстуры

Если фикстура нужна в нескольких файлах, положите её в `conftest.py` рядом с тестами. Импортировать не нужно — pytest найдёт автоматически:

<CodeProject
    defaultFile="test_api.py"
    files={[
        {
            path: 'conftest.py',
            content: `import pytest

@pytest.fixture(scope="session")
def app_config():
    return {"api_url": "https://test.example.com", "timeout": 5}
`,
        },
        {
            path: 'test_api.py',
            content: `def test_api_url(app_config):     # фикстура из conftest.py
    assert "example.com" in app_config["api_url"]
`,
        },
    ]}
/>

Это стандартный способ делиться фикстурами между тестами одного проекта.

## Параметризация: один тест, много входов

Когда нужно проверить функцию на 5-10 наборов входных данных, не пишите 10 одинаковых тестов. Используйте `@pytest.mark.parametrize`:

```python
import pytest

def get_discount(age, is_member):
    if age >= 65:
        return 0.15
    if is_member:
        return 0.10
    if age < 18:
        return 0.05
    return 0.0

@pytest.mark.parametrize(
    "age, is_member, expected",
    [
        (70, False, 0.15),
        (30, True,  0.10),
        (16, False, 0.05),
        (25, False, 0.0),
        (65, True,  0.15),
    ],
)
def test_get_discount(age, is_member, expected):
    assert get_discount(age, is_member) == expected
```

Обратите внимание на первый аргумент `parametrize`: это одна **строка** с именами параметров через запятую (`"age, is_member, expected"`), а не кортеж. Это специфическое соглашение pytest. Дальше идёт список кортежей — каждый кортеж это один прогон теста.

`pytest` запустит этот тест 5 раз, каждый со своим набором значений. В выводе они идут отдельными тестами:

```text
test_discount.py::test_get_discount[70-False-0.15] PASSED
test_discount.py::test_get_discount[30-True-0.1]   PASSED
...
```

Если один из случаев упал, в имени теста видны его параметры — сразу понятно, какая комбинация сломалась.

## Что дальше?

В следующей статье — **моки и заглушки**: как изолировать тесты от внешних зависимостей (база, API, время), чтобы они оставались быстрыми и предсказуемыми.

---

**Что верно о фикстурах и параметризации в `pytest`?**

