# Моки и заглушки: изоляция тестов

Реальный код часто общается с внешним миром: HTTP-запросы, БД, файлы, время. В тестах это плохо: сеть может упасть, БД может быть медленной, время неуправляемо. Тест должен проверять **вашу логику**, а не работоспособность чужих сервисов.

Решение — **тестовый двойник**: подсовываем коду фейковый объект, который ведёт себя как нужно. У этого двойника есть две роли:

-   **Stub** (заглушка) — возвращает заранее заданные ответы, проверять ничего не нужно.
-   **Mock** — то же самое плюс умеет фиксировать вызовы (с какими аргументами и сколько раз), чтобы тест мог это проверить.

На практике в Python оба делаются одним инструментом: `unittest.mock`.

## Mock-объект

Простейший случай: создаём `Mock` и настраиваем, что должны вернуть его методы.

```python
from unittest.mock import Mock

def process_payment(service, amount):
    if service.charge(amount):
        service.log(f"charged {amount}")
        return "OK"
    return "FAIL"

def test_payment_success():
    service = Mock()
    service.charge.return_value = True       # настроили мок

    result = process_payment(service, 100)

    assert result == "OK"
    service.charge.assert_called_once_with(100)
    service.log.assert_called_once_with("charged 100")
```

Функция `process_payment` принимает `service` аргументом. В production-коде туда передают реальный платёжный сервис; в тесте мы подсовываем Mock через тот же аргумент. В этом и смысл «зависимость через параметр». Что происходит дальше:

-   `Mock()` создаёт объект, у которого **любой** атрибут или метод существует автоматически.
-   `service.charge.return_value = True` говорит «когда тест вызовет `service.charge(...)`, верни `True`».
-   `assert_called_once_with(100)` проверяет: «метод `charge` был вызван ровно один раз с аргументом `100`». Это и есть отличие Mock от Stub: тест не просто получает данные, он проверяет **как** код взаимодействовал с зависимостью.

## patch: подменить уже существующий объект

`Mock()` хорош, если зависимость передаётся в функцию аргументом. Но часто код использует импортированный объект напрямую (например, `requests.get`). Тогда нужен `patch`: он временно подменяет что-то в коде на мок:

```python
from unittest.mock import patch, Mock

# тестируемый код
import requests

def get_user(user_id):
    response = requests.get(f"https://api.example.com/users/{user_id}")
    if response.status_code == 200:
        return response.json()
    return None

# тест
@patch("requests.get")
def test_get_user(mock_get):
    # настраиваем, что вернёт requests.get(...)
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 1, "name": "Anna"}
    mock_get.return_value = mock_response

    user = get_user(1)

    assert user == {"id": 1, "name": "Anna"}
    mock_get.assert_called_once_with("https://api.example.com/users/1")
```

Что важно:

-   `@patch("requests.get")` подменяет `requests.get` на мок только на время теста, после теста всё возвращается обратно.
-   Аргумент `mock_get` (имя любое) — это автоматически созданный мок, который заменил оригинал. Через него настраиваем поведение и проверяем вызовы.
-   `response.json()` это **метод** (с круглыми скобками), поэтому настраиваем `mock_response.json.return_value` — у вложенного мока `json` свой собственный `return_value`. Любой атрибут или метод мока тоже мок, по цепочке.

`patch` можно использовать и как **контекстный менеджер** (`with patch(...) as mock_get:`) — пригодится, когда подмена нужна только для части теста.

## Главная грабля: куда указывать путь в patch

Самая частая ошибка с `patch` это выбор пути. Правило: **патчить там, где объект используется, а не там, где он определён**.

Допустим, наш `get_user` живёт в файле `app.py`. Если код импортирует модуль целиком:

```python
# app.py
import requests

def get_user(user_id):
    return requests.get(f"https://api.example.com/users/{user_id}")
```

То в тесте патчим `"requests.get"`, и это работает, потому что в `app.py` обращение идёт через сам модуль `requests`:

```python
@patch("requests.get")   # OK
def test_get_user(mock_get):
    ...
```

А если в коде использован `from requests import get`:

```python
# app.py
from requests import get

def get_user(user_id):
    return get(f"https://api.example.com/users/{user_id}")
```

То патчить `"requests.get"` **бесполезно**. После `from ... import` в `app.py` появилась локальная ссылка `get`, которая указывает на оригинальную функцию. Подмена в модуле `requests` её не касается. Патчить нужно по месту использования:

```python
@patch("app.get")    # патчим там, где get вызывается
def test_get_user(mock_get):
    ...
```

Мнемоника: **«patch where it's looked up»**, куда тестируемый код смотрит за функцией, там и патчите.

## side_effect: имитация ошибок

Иногда нужно проверить, что код корректно реагирует на сбой зависимости (например, на `ConnectionError`). Для этого у мока есть `side_effect`:

```python
from unittest.mock import Mock

mock_api = Mock()
mock_api.connect.side_effect = ConnectionError("network down")

# теперь mock_api.connect() выбросит ConnectionError
```

Этот же `side_effect` принимает функцию или список значений (для разных ответов на последовательные вызовы) — но это уже более редкие случаи.

## Главное правило

Моки удобны, но коварны: легко начать мокать **внутренности** собственного кода, и тогда тесты проверяют не поведение, а реализацию. Любой рефакторинг ломает их, хотя код работает.

Правило: **мокайте границы системы** — внешние API, БД, файловую систему, время. Свой код тестируйте напрямую, без моков.

## Что дальше?

В следующей статье — встроенный модуль **`unittest`**: классический xUnit-стиль с классами `TestCase` и методами `setUp`/`tearDown`. Это альтернатива pytest, которую вы встретите в legacy-коде.

---

**Что верно про моки и заглушки в Python?**

