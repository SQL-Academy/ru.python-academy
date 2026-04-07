# Модуль unittest: встроенный фреймворк тестирования Python

В предыдущих статьях мы подробно изучили `pytest` — популярный и мощный фреймворк для тестирования в Python, а также рассмотрели концепции мокирования с помощью `unittest.mock`. Однако в стандартной библиотеке Python есть собственный, давно существующий фреймворк для написания тестов — модуль `unittest`.

Знание `unittest` важно по нескольким причинам:

-   **Понимание основ**: `unittest` следует классическому xUnit-стилю, который лег в основу многих других фреймворков тестирования.
-   **Работа с существующим кодом**: Вы можете столкнуться с проектами, где тесты уже написаны с использованием `unittest`.
-   **Отсутствие внешних зависимостей**: Так как `unittest` встроен, его можно использовать без установки дополнительных пакетов.

## Введение в unittest

Модуль `unittest`, иногда называемый "PyUnit", был вдохновлен JUnit (фреймворком для Java).

Ключевые особенности `unittest`:

-   **Объектно-ориентированный подход**: Тесты организуются в классы, наследующие от `unittest.TestCase`.
-   **Специальные методы утверждений**: Вместо простого `assert` используются методы вида `self.assertEqual()`, `self.assertTrue()` и т.д.
-   **Структура для настройки и очистки**: Предоставляет методы `setUp()`, `tearDown()` и их аналоги на уровне класса.

## Основы unittest

Давайте рассмотрим базовый пример.

```python
# test_simple_unittest.py
import unittest

def add(a, b):
    return a + b

class TestAddFunction(unittest.TestCase): # Класс должен наследоваться от unittest.TestCase
    def test_add_positive_numbers(self): # Тестовые методы должны начинаться с test_
        result = add(3, 5)
        self.assertEqual(result, 8) # Используем метод утверждения

    def test_add_negative_numbers(self):
        result = add(-1, -1)
        self.assertEqual(result, -2)

    def test_add_mixed_numbers(self):
        result = add(-1, 1)
        self.assertEqual(result, 0)

if __name__ == '__main__':
    unittest.main() # Стандартный способ запуска тестов из файла
```

Если сохранить этот код в файл `test_simple_unittest.py` и запустить его (`python test_simple_unittest.py`), вы увидите примерно такой вывод:

```text
...
----------------------------------------------------------------------
Ran 3 tests in 0.001s

OK
```

Структура теста `unittest`:

1.  Импортируется модуль `unittest`.
2.  Создается класс теста, наследующийся от `unittest.TestCase`.
3.  Внутри класса определяются тестовые методы, имена которых начинаются с `test_`.
4.  В тестовых методах используются специальные методы `self.assert...()` для проверки условий.
5.  Блок `if __name__ == '__main__': unittest.main()` позволяет запускать тесты при непосредственном выполнении файла.

## Методы утверждений (Assertion Methods)

Класс `TestCase` предоставляет множество методов для различных проверок. Вот некоторые из наиболее часто используемых:

| Метод                                      | Проверяет, что...                                                     |
| ------------------------------------------ | --------------------------------------------------------------------- |
| `assertEqual(a, b)`                        | `a == b`                                                              |
| `assertNotEqual(a, b)`                     | `a != b`                                                              |
| `assertTrue(x)`                            | `bool(x) is True`                                                     |
| `assertFalse(x)`                           | `bool(x) is False`                                                    |
| `assertIs(a, b)`                           | `a is b`                                                              |
| `assertIsNot(a, b)`                        | `a is not b`                                                          |
| `assertIsNone(x)`                          | `x is None`                                                           |
| `assertIsNotNone(x)`                       | `x is not None`                                                       |
| `assertIn(member, container)`              | `member in container`                                                 |
| `assertNotIn(member, container)`           | `member not in container`                                             |
| `assertIsInstance(obj, cls)`               | `isinstance(obj, cls)`                                                |
| `assertRaises(exception, callable, ...)`   | `callable` вызывает `exception`                                       |
| `assertRaisesRegex(exception, regex, ...)` | `callable` вызывает `exception` с сообщением, соответствующим `regex` |

Пример использования `assertRaises`:

```python
import unittest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

class TestDivideFunction(unittest.TestCase):
    def test_divide_by_zero(self):
        with self.assertRaises(ValueError) as cm:
            divide(10, 0)
        self.assertEqual(str(cm.exception), "Cannot divide by zero")

    def test_divide_success(self):
        self.assertEqual(divide(10, 2), 5)

if __name__ == '__main__':
    unittest.main()
```

## Настройка и очистка тестового окружения (Fixtures)

`unittest` предоставляет методы для настройки окружения перед тестами и очистки после них. Это аналог фикстур в `pytest`.

-   `setUp()`: Выполняется перед каждым тестовым методом в классе.
-   `tearDown()`: Выполняется после каждого тестового метода, даже если он вызвал исключение.
-   `setUpClass()`: Метод класса, выполняется один раз перед всеми тестами в классе.
-   `tearDownClass()`: Метод класса, выполняется один раз после всех тестов в классе.

```python
import unittest

class TestDatabaseOperations(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        # print("setUpClass: Подключение к тестовой базе данных...")
        cls.db_connection = "< simulated_db_connection >" # Имитация

    @classmethod
    def tearDownClass(cls):
        # print("tearDownClass: Закрытие подключения к базе данных.")
        cls.db_connection = None

    def setUp(self):
        # print("\nsetUp: Создание тестовых данных...")
        self.user_data = {"id": 1, "name": "Test User"}

    def tearDown(self):
        # print("tearDown: Удаление тестовых данных...")
        self.user_data = None

    def test_user_creation(self):
        # print("Выполняется test_user_creation")
        self.assertIn("name", self.user_data)
        self.assertTrue(self.db_connection)

    def test_user_property_access(self):
        # print("Выполняется test_user_property_access")
        self.assertEqual(self.user_data["id"], 1)
        self.assertTrue(self.db_connection)

# Закомментировано, т.к. print убраны для компактности
# if __name__ == '__main__':
#     unittest.main(verbosity=2)
```

## Запуск тестов unittest

Есть несколько способов запустить тесты, написанные с `unittest`:

1.  **Из файла**: `python your_test_file.py` (если в файле есть `unittest.main()`).
2.  **Через модуль `unittest` из командной строки**:
    -   `python -m unittest your_test_file.py`
    -   `python -m unittest test_module.TestClass`
    -   `python -m unittest test_module.TestClass.test_method`
    -   `python -m unittest discover`: Автоматически находит и запускает все тесты (файлы `test*.py`) в текущей директории и ее поддиректориях.
        ```bash
        python -m unittest discover -s tests_directory -p "test_*.py"
        ```
        (`-s` задает директорию для поиска, `-p` — паттерн для имен файлов).

## Пропуск тестов и ожидаемые сбои

`unittest` предоставляет декораторы для управления выполнением тестов:

-   `@unittest.skip(reason)`: Всегда пропускает тест. `reason` — строка с объяснением.
-   `@unittest.skipIf(condition, reason)`: Пропускает тест, если `condition` истинно.
-   `@unittest.skipUnless(condition, reason)`: Пропускает тест, если `condition` ложно.
-   `@unittest.expectedFailure`: Отмечает тест как "ожидаемо неудачный". Если тест падает, он считается успешно пройденным (в специальной категории). Если он внезапно проходит, это считается ошибкой.

```python
import unittest
import sys

class TestSkipping(unittest.TestCase):
    @unittest.skip("Причина: демонстрация пропуска")
    def test_always_skipped(self):
        self.fail("Этот тест не должен был запуститься")

    @unittest.skipIf(sys.version_info < (3, 10), "Требуется Python 3.10+")
    def test_python_310_feature(self):
        self.assertTrue(True) # Логика для Python 3.10+

    @unittest.expectedFailure
    def test_broken_feature(self):
        self.assertEqual(1, 2, "Эта функция пока не работает корректно")

if __name__ == '__main__':
    unittest.main()
```

## unittest в сравнении с pytest

Теперь, когда вы знакомы с основами `pytest` и `unittest`, давайте сравним их:

| Особенность        | `unittest`                                        | `pytest`                                         |
| ------------------ | ------------------------------------------------- | ------------------------------------------------ |
| **Стиль**          | ООП (классы `TestCase`), xUnit-стиль              | Функциональный, меньше шаблонного кода           |
| **Утверждения**    | Специальные методы (`self.assertEqual()` и т.д.)  | Стандартный `assert` Python (более читаемо)      |
| **Фикстуры**       | `setUp/tearDown` методы                           | Мощные фикстуры с `@pytest.fixture`, DI, `scope` |
| **Параметризация** | Требует ручной реализации или расширений          | Встроенная (`@pytest.mark.parametrize`)          |
| **Обнаружение**    | Файлы `test*.py`, классы `Test*`, методы `test_*` | Более гибкое, меньше строгих правил именования   |
| **Плагины**        | Ограниченная экосистема                           | Обширная экосистема плагинов                     |
| **Сообщество**     | Стандартная библиотека, широко используется       | Очень активное сообщество, де-факто стандарт     |
| **Зависимости**    | Встроен, нет внешних зависимостей                 | Требует установки (`pip install pytest`)         |

**Когда может быть предпочтительнее `unittest`?**

-   В старых проектах, где он уже используется.
-   Если есть строгое требование не использовать внешние зависимости.
-   Для написания тестов для самой стандартной библиотеки Python.
-   Если вы предпочитаете классический xUnit-стиль.

В большинстве новых проектов Python-сообщество склоняется к использованию `pytest` из-за его гибкости, простоты и мощных возможностей.

## Использование unittest.mock с unittest

Как мы уже обсуждали в статье про моки и заглушки, библиотека `unittest.mock` является частью стандартной библиотеки и отлично работает с фреймворком `unittest`. Механизмы `patch` и создания объектов `Mock` используются точно так же.

```python
import unittest
from unittest.mock import patch, Mock

# --- Тестируемый код ---
class ExternalService:
    def get_data(self):
        # Реальный вызов, который мы хотим избежать в тесте
        raise NotImplementedError("Don't call real service in test!")

class MyClass:
    def __init__(self):
        self.service = ExternalService()

    def process_data(self):
        data = self.service.get_data()
        # Какая-то обработка данных
        return f"Processed: {data.get('key', 'no_key')}"
# --- Конец тестируемого кода ---

class TestMyClassWithUnittest(unittest.TestCase):
    # Патчим метод get_data у ExternalService
    @patch('__main__.ExternalService.get_data')
    def test_process_data_with_mock(self, mock_get_data):
        # Настраиваем мок
        mock_get_data.return_value = {"key": "mocked_value"}

        instance = MyClass()
        result = instance.process_data()

        self.assertEqual(result, "Processed: mocked_value")
        mock_get_data.assert_called_once() # Проверяем, что мок был вызван

if __name__ == '__main__':
    unittest.main()
```

Здесь `patch` используется для замены метода `get_data` у `ExternalService` моком во время выполнения теста `test_process_data_with_mock`.

## Что дальше?

Мы рассмотрели ключевые аспекты фреймворка `unittest`. Понимание его принципов полезно для любого Python-разработчика.

В заключительной статье этой серии мы поговорим о том, как измерять качество нашего тестового покрытия и как автоматизировать процесс тестирования с помощью систем непрерывной интеграции (CI).

---

**Какое утверждение о модуле `unittest` является верным?**
