# Моки и заглушки в Python: изоляция тестов с unittest.mock

В предыдущих статьях мы освоили `pytest`, включая его мощные фикстуры и параметризацию. Однако при тестировании реальных приложений часто возникает проблема: наш код зависит от внешних систем (базы данных, API, файловая система, время) или других сложных компонентов, которые трудно или нежелательно использовать напрямую в тестах.

Здесь на помощь приходят **тестовые двойники (test doubles)**, в частности **моки (mocks)** и **заглушки (stubs)**.

## Проблема внешних зависимостей

Представьте, что вы тестируете функцию, которая:

-   Отправляет HTTP-запрос к внешнему API (например, для получения курса валют).
-   Читает или записывает данные в базу данных.
-   Работает с файлами на диске.
-   Зависит от текущего времени.

Прямое использование этих зависимостей в юнит-тестах нежелательно, потому что:

-   **Медленно**: Сетевые запросы или операции с БД могут быть долгими.
-   **Ненадежно**: Внешний сервис может быть недоступен, или данные в БД могут измениться.
-   **Сложно контролировать**: Трудно имитировать специфические ответы или состояния ошибок от внешних систем.
-   **Побочные эффекты**: Тесты могут изменять реальные данные (например, создавать записи в БД или отправлять настоящие email).

> Цель юнит-тестирования — проверить логику _вашего_ кода в изоляции. Тестовые двойники позволяют заменить реальные зависимости контролируемыми имитациями.

## Тестовые двойники: Заглушки (Stubs) vs Моки (Mocks)

Существует несколько типов тестовых двойников, но наиболее часто используются заглушки и моки.

-   **Заглушка (Stub)**: Это простой объект, который предоставляет предопределенные ("зашитые") ответы на вызовы методов во время теста. Заглушки используются, когда тесту нужны какие-то данные от зависимости, но сама зависимость не является объектом проверки.

    -   _Пример_: Заглушка для сервиса, возвращающего погоду, всегда отдает `{"temperature": 25, "condition": "sunny"}`.

-   **Мок (Mock)**: Это более "умный" объект. Моки не только предоставляют ответы (как заглушки), но и позволяют вам **проверять, как ваш код взаимодействовал с этой зависимостью**. Вы можете настроить ожидания (например, какой метод должен быть вызван, с какими аргументами, сколько раз) и затем проверить, были ли эти ожидания выполнены.
    -   _Пример_: Мок для сервиса отправки email, который позволяет проверить, что метод `send_email` был вызван ровно один раз с правильным адресом получателя и темой письма.

| Тип          | Основное назначение                                    | Проверяет ли взаимодействия? | Когда использовать                                    |
| ------------ | ------------------------------------------------------ | ---------------------------- | ----------------------------------------------------- |
| **Заглушка** | Предоставление фиксированных данных для теста          | Нет                          | Когда тесту нужны простые данные от зависимости       |
| **Мок**      | Имитация поведения и **проверка** корректности вызовов | Да                           | Когда важно проверить, как код использует зависимость |

## Библиотека unittest.mock

Python предоставляет мощную стандартную библиотеку `unittest.mock` для создания моков, заглушек и других тестовых двойников. Она прекрасно интегрируется с `pytest`.

Ключевые компоненты `unittest.mock`:

-   `Mock` и `MagicMock`: Основные классы для создания мок-объектов. `MagicMock` расширяет `Mock`, предоставляя реализации для большинства магических методов Python (например, `__str__`, `__len__`).
-   `patch`: Мощный инструмент (декоратор или контекстный менеджер) для временной замены объектов в вашем коде моками во время выполнения теста.

### Базовое использование Mock

```python
# test_basic_mock.py
from unittest.mock import Mock

def process_payment(payment_service, amount):
    # print(f"Пытаюсь списать {amount}...") # Упрощаем, убирая print
    success = payment_service.charge(amount)
    if success:
        # print("Оплата прошла успешно") # Упрощаем
        payment_service.log_transaction(details=f"Charged {amount}")
        return "SUCCESS"
    else:
        # print("Ошибка оплаты") # Упрощаем
        return "FAILED"


def test_process_payment_success():
    # 1. Создаем мок для payment_service
    mock_payment_service = Mock()

    # 2. Настраиваем поведение мока
    # Метод charge() должен вернуть True
    mock_payment_service.charge.return_value = True

    # 3. Вызываем тестируемую функцию с моком
    result = process_payment(mock_payment_service, 100)

    # 4. Проверяем результат работы функции
    assert result == "SUCCESS"

    # 5. Проверяем взаимодействия с моком
    mock_payment_service.charge.assert_called_once_with(100) # Проверяем, что charge был вызван 1 раз с суммой 100
    mock_payment_service.log_transaction.assert_called_once_with(details="Charged 100") # Проверяем вызов log_transaction

def test_process_payment_failure():
    mock_payment_service = Mock()
    mock_payment_service.charge.return_value = False # Имитируем неудачную оплату

    result = process_payment(mock_payment_service, 50)

    assert result == "FAILED"
    mock_payment_service.charge.assert_called_once_with(50)
    mock_payment_service.log_transaction.assert_not_called() # Убеждаемся, что log_transaction не был вызван
```

В этих примерах `mock_payment_service.charge` и `mock_payment_service.log_transaction` автоматически становятся моками, как только вы к ним обращаетесь. Атрибут `return_value` позволяет задать, что вернет вызов мок-метода. Методы `assert_called_once_with()`, `assert_not_called()` и другие позволяют проверить, как именно использовался мок.

### patch: Замена объектов на лету

`patch` позволяет вам заменить объекты в определенном модуле моками на время выполнения теста. Это очень удобно, когда вы не можете легко передать мок в качестве аргумента (например, если объект создается внутри тестируемой функции или импортируется глобально).

`patch` можно использовать как декоратор или как контекстный менеджер.

```python
# test_patch_example.py
from unittest.mock import patch, Mock

# --- Тестируемый код (обычно в другом файле) ---
import requests
def get_external_data(item_id):
    # Эта функция делает реальный сетевой запрос
    print(f"\nВызов requests.get для {item_id}...") # Оставим print для демонстрации, что он НЕ выполнится в тесте
    response = requests.get(f"https://api.example.com/items/{item_id}")
    if response.status_code == 200:
        return response.json()
    return None
# --- Конец тестируемого кода ---

#### Использование patch как декоратора

@patch('__main__.requests.get') # Патчим requests.get в текущем модуле, где он используется
def test_get_external_data_with_decorator(mock_requests_get):
    # Настраиваем мок, который будет передан в mock_requests_get
    print("\nЗапуск test_get_external_data_with_decorator")
    mock_response = Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 1, "name": "Test Item"}
    mock_requests_get.return_value = mock_response

    data = get_external_data(1)

    assert data == {"id": 1, "name": "Test Item"}
    mock_requests_get.assert_called_once_with("https://api.example.com/items/1")

#### Использование patch как контекстного менеджера

def test_get_external_data_with_context_manager():
    print("\nЗапуск test_get_external_data_with_context_manager")
    with patch('__main__.requests.get') as mock_requests_get_cm:
        mock_response = Mock()
        mock_response.status_code = 404
        mock_requests_get_cm.return_value = mock_response

        data = get_external_data(2)

        assert data is None
        mock_requests_get_cm.assert_called_once_with("https://api.example.com/items/2")

#### Патчинг методов объекта с patch.object

# --- Тестируемый код ---
class ReportGenerator:
    def _get_user_stats(self, user_id):
        # Имитация сложного/медленного вызова
        raise NotImplementedError("Не вызывайте реальный метод в тесте!")

    def generate_report(self, user_id):
        stats = self._get_user_stats(user_id)
        return f"User {user_id}: Logins - {stats['logins']}, Spent - {stats['spent']}"
# --- Конец тестируемого кода ---

@patch.object(ReportGenerator, '_get_user_stats') # Патчим метод конкретного класса
def test_generate_report_patches_object_method(mock_get_stats):
    print("\nЗапуск test_generate_report_patches_object_method")
    mock_get_stats.return_value = {"logins": 10, "spent": 50} # Упрощенные данные для теста

    generator = ReportGenerator()
    report = generator.generate_report(user_id=123)

    assert report == "User 123: Logins - 10, Spent - 50"
    mock_get_stats.assert_called_once_with(123)
```

**Важно при использовании `patch`**: Путь к объекту, который вы патчите, должен быть тем местом, где объект _используется_ (где он импортирован и вызывается), а не где он определен.

### side_effect: Имитация ошибок и последовательных вызовов

Атрибут `side_effect` у мока позволяет имитировать более сложное поведение:

-   **Вызов исключения**: Если присвоить `side_effect` исключение (например, `ConnectionError`), мок вызовет это исключение при обращении.
-   **Возврат разных значений при последовательных вызовах**: Если присвоить `side_effect` итерируемый объект (например, список), мок будет возвращать элементы этого списка по очереди при каждом вызове.
-   **Выполнение функции**: Можно присвоить `side_effect` функцию, которая будет вызвана вместо мока.

```python
from unittest.mock import Mock

# 1. Имитация исключения
mock_api = Mock()
mock_api.connect.side_effect = ConnectionError("Failed to connect to API")

try:
    mock_api.connect()
except ConnectionError as e:
    print(f"Перехвачена ошибка: {e}")

# 2. Последовательные возвращаемые значения
mock_db_reader = Mock()
mock_db_reader.read_row.side_effect = [
    {"id": 1, "data": "A"},
    {"id": 2, "data": "B"},
    None # Имитация конца данных
]

print(mock_db_reader.read_row()) # Вернет {"id": 1, "data": "A"}
print(mock_db_reader.read_row()) # Вернет {"id": 2, "data": "B"}
print(mock_db_reader.read_row()) # Вернет None

# 3. Выполнение функции
def custom_side_effect(*args, **kwargs):
    user_id = kwargs.get("user_id", 0)
    print(f"-> custom_side_effect вызван для user_id={user_id}")
    return "Admin" if user_id == 1 else "Guest"

mock_user_service = Mock()
mock_user_service.get_user_type.side_effect = custom_side_effect

print(mock_user_service.get_user_type(user_id=1))
print(mock_user_service.get_user_type(user_id=2))
```

### Использование с pytest и фикстурой mocker

Хотя `unittest.mock` является частью стандартной библиотеки, для более удобной интеграции с `pytest` часто используется плагин `pytest-mock`. Он предоставляет фикстуру `mocker`, которая является оберткой вокруг `unittest.mock.patch` и упрощает некоторые сценарии.

```bash
# pip install pytest-mock (если еще не установлен)
```

```python
# test_pytest_mocker.py
# (предполагаем, что get_external_data определена как в предыдущем примере patch)

# --- Код из module_to_test.py для самодостаточности примера ---
import requests
def get_external_data(item_id):
    response = requests.get(f"https://api.example.com/items/{item_id}")
    if response.status_code == 200:
        return response.json()
    return None
# --- Конец кода из module_to_test.py ---

def test_get_external_data_with_mocker(mocker): # mocker - это фикстура pytest-mock
    # Используем mocker.patch вместо unittest.mock.patch
    mock_requests_get = mocker.patch('__main__.requests.get')

    mock_response = Mock() # Можно использовать и unittest.mock.Mock напрямую
    mock_response.status_code = 200
    mock_response.json.return_value = {"id": 10, "name": "Mocked Item"}
    mock_requests_get.return_value = mock_response

    data = get_external_data(10)
    assert data == {"id": 10, "name": "Mocked Item"}
    mock_requests_get.assert_called_once_with("https://api.example.com/items/10")

# mocker также имеет удобные методы, например, mocker.stub()
class EmailSender:
    def send(self, to, body):
        # Реальная отправка письма
        raise NotImplementedError("Don't send real emails in tests!")

def test_email_sending_logic(mocker):
    sender_instance = EmailSender()
    # mocker.patch.object() аналогичен unittest.mock.patch.object()
    mock_send_method = mocker.patch.object(sender_instance, 'send')
    mock_send_method.return_value = True # Настроим, чтобы send возвращал True

    # Тестируемый код, который использует sender_instance.send(...)
    # result = process_user_notification(user_id=1, email_sender=sender_instance)
    # assert result is True

    # Для примера просто вызовем его
    assert sender_instance.send("test@example.com", "Hello") == True
    mock_send_method.assert_called_once_with("test@example.com", "Hello")
```

Фикстура `mocker` обеспечивает удобный доступ к функциональности `patch` без необходимости импортировать `patch` напрямую в каждом тестовом файле.

## Лучшие практики использования моков и заглушек

1.  **Мокайте границы, а не внутренности**: Старайтесь мокать только то, что находится на границе вашей системы или вашего модуля (внешние API, прямой доступ к БД, файловая система). Не мокайте слишком много внутренних деталей вашего собственного кода, иначе тесты станут хрупкими и будут проверять реализацию, а не поведение.
2.  **Тестируйте поведение, а не реализацию**: Моки помогают проверить, что ваш код _правильно использует_ свои зависимости, а не то, как эти зависимости реализованы внутри.
3.  **Один мок на одно поведение**: Старайтесь, чтобы один мок имитировал одно конкретное поведение или отвечал за один аспект зависимости.
4.  **Делайте моки как можно проще**: Чем проще мок, тем легче понять тест.
5.  **`patch` с осторожностью**: Используйте `patch` обдуманно. Чрезмерное использование `patch` может сделать тесты сложными для понимания и поддержки, так как становится неясно, какой код на самом деле выполняется.
6.  **Предпочитайте внедрение зависимостей (Dependency Injection)**: Если это возможно, проектируйте свой код так, чтобы зависимости можно было легко передавать в функции или классы (например, через аргументы конструктора или методов). Это часто упрощает тестирование и уменьшает необходимость в `patch`.

## Что дальше?

Теперь, когда вы умеете изолировать зависимости с помощью моков и заглушек, ваши тесты станут более надежными, быстрыми и сфокусированными на проверке конкретной логики.

В следующей статье мы познакомимся со встроенным в Python фреймворком `unittest`, который также активно использует концепции мокирования.

---

**Какое утверждение о моках и заглушках в Python верно?**
