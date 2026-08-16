---
meta:
    title: "Модуль unittest: классический тестовый фреймворк"
    description: "unittest — встроенный в Python xUnit-фреймворк: TestCase, методы assertEqual/assertTrue, setUp/tearDown. Что вы встретите в legacy-коде."
---

# Модуль unittest: классический фреймворк

Открываете проект постарше, а тесты там не похожи на те, что вы только что писали: классы, наследование от `TestCase`, проверки через `self.assertEqual` вместо простого `assert`. Это `unittest` — тестовый фреймворк из стандартной библиотеки Python, классический ООП-стиль (тем, кто видел JUnit в Java, всё будет знакомо). Большинство новых проектов берут `pytest`, но `unittest` никуда не делся и встречается:

- В legacy-коде (фреймворк существует с Python 2.1).
- В проектах со строгим запретом на внешние зависимости.
- В тестах самой стандартной библиотеки.

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

- Тесты живут в **классе**, который наследуется от `unittest.TestCase`.
- Имена методов начинаются с `test_`, как и в pytest.
- Вместо обычного `assert` используются **специальные методы**: `self.assertEqual(a, b)` вместо `assert a == b`.
- `unittest.main()` в конце файла — точка запуска при `python test_file.py`.

Запуск через стандартный `python` или через `python -m unittest`:

```bash
python test_addition.py
# или
python -m unittest test_addition.py
```

## Главные assert-методы

`TestCase` предоставляет много специальных проверок. На практике хватает 6 самых частых:

| Метод                       | Проверяет                   |
| --------------------------- | --------------------------- |
| `assertEqual(a, b)`         | `a == b`                    |
| `assertNotEqual(a, b)`      | `a != b`                    |
| `assertTrue(x)`             | `bool(x) is True`           |
| `assertFalse(x)`            | `bool(x) is False`          |
| `assertIn(item, container)` | `item in container`         |
| `assertRaises(Exception)`   | блок выбрасывает исключение |

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

![Иллюстрация: класс TestUserStorage(unittest.TestCase), внутри последовательность шагов 1.setUp 2.test_has_name 3.tearDown 4.setUp 5.test_has_id 6.tearDown — для каждого теста setUp и tearDown запускаются заново](https://python-academy.org/static/guidePage/unittest-module/setup-teardown-ru.webp "setUp и tearDown запускаются для каждого теста")

Для каждого теста цикл `setUp → test_X → tearDown` запускается заново — это гарантирует, что тесты независимы друг от друга. Есть ещё `setUpClass`/`tearDownClass` для подготовки **один раз на класс**, но это уже частные случаи.

## unittest vs pytest

| Что            | unittest                     | pytest                     |
| -------------- | ---------------------------- | -------------------------- |
| Тесты          | методы класса `TestCase`     | обычные функции            |
| Проверки       | `self.assertEqual()` и т.п.  | стандартный `assert`       |
| Подготовка     | `setUp` / `tearDown`         | фикстуры с DI и scope      |
| Параметризация | вручную или через расширения | `@pytest.mark.parametrize` |
| Зависимости    | встроен                      | `pip install pytest`       |

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

## Проверка понимания

**Что верно про модуль `unittest`?**

1. **Правильный ответ:** Тестовые классы в unittest должны наследоваться от unittest.TestCase — Именно TestCase даёт доступ к методам assert\*, setUp/tearDown и интеграции с раннером.

2. unittest использует стандартный assert Python для проверок — Нет, unittest использует специальные методы (self.assertEqual, self.assertTrue и т.д.). Стандартный assert это стиль pytest.

3. setUp запускается один раз перед всеми тестами в классе — setUp запускается перед КАЖДЫМ тестом. Один раз на класс это setUpClass.

4. Тестовые методы в unittest могут называться как угодно — Раннер находит только методы, чьё имя начинается с test. Метод с другим именем просто не запустится как тест.

В следующей (заключительной для модуля тестирования) статье — как измерять **покрытие** тестами через `pytest-cov` и автоматизировать запуск через **CI** (на примере GitHub Actions).
