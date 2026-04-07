# pytest: фикстуры, параметризация и продвинутые тесты

В предыдущей статье мы познакомились с основами `pytest`: научились писать и запускать простые тесты. Теперь пришло время углубиться в две самые мощные и часто используемые концепции `pytest`, которые делают его таким гибким и удобным: **фикстуры** и **параметризация**.

## Фикстуры: подготовка и управление тестовым окружением ⚙️

Часто для проведения теста необходимо выполнить некоторые предварительные действия: подготовить данные, настроить тестовый объект, установить соединение с базой данных и т.д. После теста может потребоваться выполнить очистку: удалить временные файлы, закрыть соединения.

> **Фикстуры** в `pytest` — это функции, которые служат для настройки и предоставления данных или объектов, необходимых для ваших тестов. Они помогают сделать тесты более чистыми, структурированными и избежать дублирования кода.

### Создание простой фикстуры

Фикстура определяется с помощью декоратора `@pytest.fixture`.

```python
# test_fixtures_example.py
import pytest

@pytest.fixture
def sample_list():
    print("\n(Фикстура sample_list: создаю список)")
    return [1, 2, 3, 4, 5]

@pytest.fixture
def sample_dict():
    print("\n(Фикстура sample_dict: создаю словарь)")
    return {"name": "Alice", "age": 30}

def test_list_length(sample_list): # Имя фикстуры передается как аргумент
    print("(Тест test_list_length)")
    assert len(sample_list) == 5

def test_dict_name(sample_dict): # Другая фикстура
    print("(Тест test_dict_name)")
    assert sample_dict["name"] == "Alice"

def test_list_and_dict_usage(sample_list, sample_dict):
    print("(Тест test_list_and_dict_usage)")
    assert len(sample_list) > 0
    assert "age" in sample_dict
```

Если запустить `pytest -v -s` (флаг `-s` нужен, чтобы увидеть выводы `print` из фикстур и тестов):

-   `pytest` выполнит каждую фикстуру перед тестом, который ее запрашивает.
-   Результат выполнения фикстуры (то, что она возвращает) передается в тест как аргумент.

### Завершение работы фикстуры: yield для setup и teardown

Если вам нужно выполнить какие-то действия _после_ того, как тест завершился (например, закрыть соединение с базой данных или удалить временный файл), используйте `yield` в фикстуре.

```python
# test_fixture_yield.py
import pytest
import os

@pytest.fixture
def temp_file_fixture():
    file_path = "temp_test_file.txt"
    print(f"\n(Фикстура: создаю файл {file_path})")
    with open(file_path, "w") as f:
        f.write("Hello, pytest!")

    yield file_path # Тест получает это значение

    print(f"\n(Фикстура: удаляю файл {file_path})")
    os.remove(file_path)

def test_read_temp_file(temp_file_fixture):
    file_path = temp_file_fixture # Получаем путь к файлу от фикстуры
    print(f"(Тест: читаю файл {file_path})")
    with open(file_path, "r") as f:
        content = f.read()
    assert content == "Hello, pytest!"
```

Код до `yield` выполняется перед тестом (setup), а код после `yield` — после теста (teardown).

### Области действия фикстур (scope)

По умолчанию фикстура выполняется для каждой тестовой функции, которая ее запрашивает. Это поведение можно изменить с помощью параметра `scope` в декораторе `@pytest.fixture`.

Доступные области действия:

-   `function` (по умолчанию): Фикстура выполняется один раз для каждой тестовой функции.
-   `class`: Фикстура выполняется один раз для каждого тестового класса.
-   `module`: Фикстура выполняется один раз для каждого модуля.
-   `package`: Фикстура выполняется один раз для каждого пакета (Python 3).
-   `session`: Фикстура выполняется один раз за всю тестовую сессию.

```python
# test_scopes.py
import pytest

@pytest.fixture(scope="session")
def session_db_connection():
    print("\n(Фикстура session_db_connection: устанавливаю соединение с БД - ОДИН РАЗ ЗА СЕССИЮ)")
    connection = {"status": "connected"} # Имитация соединения
    yield connection
    print("\n(Фикстура session_db_connection: закрываю соединение с БД - ОДИН РАЗ ЗА СЕССИЮ)")

@pytest.fixture(scope="module")
def module_data_setup(session_db_connection):
    # Эта фикстура зависит от session_db_connection
    assert session_db_connection["status"] == "connected"
    print("\n(Фикстура module_data_setup: настраиваю данные для модуля - ОДИН РАЗ ЗА МОДУЛЬ)")
    data = {"module_id": 123}
    yield data
    print("\n(Фикстура module_data_setup: очищаю данные модуля)")

class TestUserOperations:
    def test_user_create(self, module_data_setup, session_db_connection):
        print("(Тест test_user_create)")
        assert session_db_connection["status"] == "connected"
        assert module_data_setup["module_id"] == 123

    def test_user_view(self, module_data_setup, session_db_connection):
        print("(Тест test_user_view)")
        assert session_db_connection["status"] == "connected"
        assert module_data_setup["module_id"] == 123

def test_another_operation(module_data_setup, session_db_connection):
    print("(Тест test_another_operation в том же модуле)")
    assert session_db_connection["status"] == "connected"
    assert module_data_setup["module_id"] == 123
```

Выбор правильной области видимости помогает оптимизировать выполнение тестов, избегая ненужных повторных настроек.

### Общие фикстуры: conftest.py

Если у вас есть фикстуры, которые вы хотите использовать в нескольких тестовых файлах, вы можете определить их в специальном файле с именем `conftest.py`. `pytest` автоматически обнаруживает этот файл.

-   Файл `conftest.py` должен находиться в директории с тестами или в родительской директории.
-   Фикстуры, определенные в `conftest.py`, становятся доступными для всех тестов в этой директории и ее поддиректориях без необходимости их импортировать.

**Пример:**

Содержимое `conftest.py` (в корневой папке тестов или подпапке):

```python
# conftest.py
import pytest

@pytest.fixture(scope="session")
def global_app_config():
    print("\n(conftest.py: Загружаю глобальную конфигурацию приложения - ОДИН РАЗ ЗА СЕССИЮ)")
    return {"api_url": "https://test.api.example.com", "timeout": 5}
```

Содержимое `test_module_one.py`:

```python
# test_module_one.py
def test_api_url(global_app_config): # Фикстура из conftest.py доступна здесь
    print(f"(Тест test_api_url: использую {global_app_config['api_url']})")
    assert "example.com" in global_app_config["api_url"]
```

### Автоматическое использование фикстур (autouse)

Иногда фикстура должна быть выполнена для всех тестов в определенной области видимости, даже если тесты явно ее не запрашивают. Для этого используется параметр `autouse=True`.

```python
# conftest.py
import pytest

@pytest.fixture(autouse=True, scope="session")
def print_start_end_session():
    print("\n--- НАЧАЛО ТЕСТОВОЙ СЕССИИ ---")
    yield
    print("\n--- КОНЕЦ ТЕСТОВОЙ СЕССИИ ---")

@pytest.fixture(autouse=True, scope="function")
def track_test_duration():
    import time
    start_time = time.time()
    yield
    duration = time.time() - start_time
    print(f"(Тест выполнялся: {duration:.4f} сек.)")
```

Используйте `autouse` с осторожностью, так как это может сделать зависимости тестов менее явными.

## Параметризация: один тест, много запусков 🔄

Часто возникает необходимость проверить одну и ту же логику с разными наборами входных данных и ожидаемых результатов. Вместо написания нескольких почти идентичных тестов `pytest` предлагает **параметризацию** с помощью маркера `@pytest.mark.parametrize`.

```python
# test_parametrize_example.py
import pytest

def get_discount(age, is_member):
    if age >= 65:
        return 0.15 # 15% скидка для пожилых
    if is_member:
        return 0.10 # 10% скидка для участников
    if age < 18:
        return 0.05 # 5% скидка для несовершеннолетних
    return 0.0

@pytest.mark.parametrize(
    "age, is_member, expected_discount", # Имена аргументов в тестовой функции
    [
        (70, False, 0.15),      # Тестовый случай 1
        (30, True, 0.10),       # Тестовый случай 2
        (16, False, 0.05),      # Тестовый случай 3
        (25, False, 0.0),       # Тестовый случай 4
        (65, True, 0.15),       # Пожилой участник все равно получает скидку для пожилых
        pytest.param(10, True, 0.05, id="JuniorMember"), # Пример с id для теста
    ]
)
def test_get_discount_parametrized(age, is_member, expected_discount):
    assert get_discount(age, is_member) == expected_discount
```

В этом примере:

-   Тест `test_get_discount_parametrized` будет запущен несколько раз, по одному разу для каждой тройки значений в списке.
-   `pytest.param(...)` можно использовать для присвоения пользовательского ID тестовому случаю, что улучшает читаемость вывода при большом количестве параметров.

### Комбинирование фикстур и параметризации

Фикстуры и параметризация могут использоваться вместе для создания очень гибких тестовых сценариев.

```python
# test_fixtures_and_parametrize.py
import pytest

@pytest.fixture
def user_profile_factory():
    def _create_profile(name, role="viewer"):
        return {"username": name, "role": role, "permissions": []}
    return _create_profile

@pytest.mark.parametrize(
    "role_to_assign, expected_permission_count",
    [
        ("admin", 5),
        ("editor", 3),
        ("viewer", 1)
    ]
)
def test_user_permissions_after_role_assignment(
    user_profile_factory,
    role_to_assign,
    expected_permission_count
):
    profile = user_profile_factory(name="testuser")

    # Имитируем логику присвоения прав в зависимости от роли
    if role_to_assign == "admin":
        profile["permissions"] = ["read", "write", "delete", "manage_users", "publish"]
    elif role_to_assign == "editor":
        profile["permissions"] = ["read", "write", "publish"]
    elif role_to_assign == "viewer":
        profile["permissions"] = ["read"]

    profile["role"] = role_to_assign

    assert profile["role"] == role_to_assign
    assert len(profile["permissions"]) == expected_permission_count
```

## Что дальше?

Фикстуры и параметризация — это хлеб с маслом для эффективного тестирования с `pytest`. Освоив их, вы сможете писать чистые, поддерживаемые и всесторонние тесты.

В следующей статье мы рассмотрим, как изолировать наши тесты от внешних зависимостей с помощью моков и заглушек.

**Какое утверждение о фикстурах и параметризации в `pytest` является верным?**
