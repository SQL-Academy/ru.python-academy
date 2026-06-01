# Модуль unittest: классический фреймворк

В стандартной библиотеке Python есть свой фреймворк для тестов — `unittest`. Стиль классический ООП: тесты живут в классах, наследуются от `TestCase`, проверки делаются через специальные методы `assert*` (тем, кто знаком с JUnit в Java, всё это будет привычно). Большинство новых проектов выбирают `pytest`, но `unittest` всё ещё встречается:

-   В legacy-коде (фреймворк существует с Python 2.1).
-   В проектах со строгим запретом на внешние зависимости.
-   В тестах самой стандартной библиотеки.

Знать его базу полезно — рано или поздно вы откроете чужой репозиторий с `unittest` и нужно будет разобраться.

## Базовая структура

```python
import unittest

def add(a, b):
    return a + b

class TestAddFunction(unittest.TestCase):
    def test_add_positive(self):
        self.assertEqual(add(3, 5), 8)

    def test_add_negative(self):
        self.assertEqual(add(-1, -1), -2)

if __name__ == "__main__":
    unittest.main()
```

Что отличается от `pytest`:

-   Тесты живут в **классе**, который наследуется от `unittest.TestCase`.
-   Имена методов начинаются с `test_`, как и в pytest.
-   Вместо обычного `assert` используются **специальные методы**: `self.assertEqual(a, b)` вместо `assert a == b`.
-   `unittest.main()` в конце файла — точка запуска при `python test_file.py`.

Запуск через стандартный `python` или через `python -m unittest`:

```bash
python test_addition.py
# или
python -m unittest test_addition.py
```

## Главные assert-методы

`TestCase` предоставляет много специальных проверок. На практике хватает 6 самых частых:

| Метод                        | Проверяет                  |
| ---------------------------- | -------------------------- |
| `assertEqual(a, b)`          | `a == b`                   |
| `assertNotEqual(a, b)`       | `a != b`                   |
| `assertTrue(x)`              | `bool(x) is True`          |
| `assertFalse(x)`             | `bool(x) is False`         |
| `assertIn(item, container)`  | `item in container`        |
| `assertRaises(Exception)`    | блок выбрасывает исключение |

Пример `assertRaises` через контекстный менеджер:

```python
import unittest

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

class TestDivide(unittest.TestCase):
    def test_zero_division(self):
        with self.assertRaises(ValueError):
            divide(10, 0)

    def test_normal(self):
        self.assertEqual(divide(10, 2), 5)
```

## setUp и tearDown

В `unittest` нет фикстур — есть `setUp()` (запускается перед каждым тестом) и `tearDown()` (после каждого, даже если тест упал):

```python
import unittest

class TestUserStorage(unittest.TestCase):
    def setUp(self):
        self.users = {"id": 1, "name": "Anna"}

    def tearDown(self):
        self.users = None

    def test_has_name(self):
        self.assertIn("name", self.users)

    def test_has_id(self):
        self.assertEqual(self.users["id"], 1)
```

Для каждого теста цикл `setUp → test_X → tearDown` запускается заново — это гарантирует, что тесты независимы друг от друга. Есть ещё `setUpClass`/`tearDownClass` для подготовки **один раз на класс**, но это уже частные случаи.

## unittest vs pytest

| Что                | unittest                       | pytest                            |
| ------------------ | ------------------------------ | --------------------------------- |
| Тесты              | методы класса `TestCase`       | обычные функции                   |
| Проверки           | `self.assertEqual()` и т.п.    | стандартный `assert`              |
| Подготовка         | `setUp` / `tearDown`           | фикстуры с DI и scope             |
| Параметризация    | вручную или через расширения  | `@pytest.mark.parametrize`        |
| Зависимости        | встроен                        | `pip install pytest`              |

`pytest` лаконичнее, гибче и в новых проектах используется по умолчанию. Но `unittest` встроен и не требует ничего ставить — для скриптов и стандартной библиотеки это плюс.

## Моки в unittest

`unittest.mock` (про который мы говорили в прошлой статье) — часть того же модуля. Тот же `@patch`, тот же `Mock` работают в `unittest`-тестах **точно так же**, как в pytest. Тестовый метод просто получает `mock_*` параметром после `self`:

```python
import unittest
from unittest.mock import patch

class TestUserAPI(unittest.TestCase):
    @patch("requests.get")
    def test_get_user(self, mock_get):
        mock_get.return_value.json.return_value = {"id": 1}
        # ... тестируем код, использующий requests.get
```

## Что дальше?

В следующей (заключительной для модуля тестирования) статье — как измерять **покрытие** тестами через `pytest-cov` и автоматизировать запуск через **CI** (на примере GitHub Actions).

---

**Что верно про модуль `unittest`?**

