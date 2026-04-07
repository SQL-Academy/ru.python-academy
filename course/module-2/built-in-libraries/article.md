# Встроенные библиотеки Python

Представьте, что вы получили новый смартфон. Сразу после покупки на нём уже установлены календарь, калькулятор, камера и другие полезные приложения. Встроенные библиотеки Python работают похожим образом! 🧰

Встроенные библиотеки — это модули, которые поставляются вместе с Python и доступны сразу после его установки. Они предоставляют готовые решения для самых распространённых задач программирования.

## Что такое встроенные библиотеки?

> Встроенные библиотеки (стандартная библиотека) — это набор модулей, которые включены в дистрибутив Python и могут использоваться без дополнительной установки.

Стандартная библиотека Python включает:

-   Модули для работы с системой
-   Инструменты для обработки данных
-   Утилиты для работы с сетью
-   Средства для создания пользовательских интерфейсов
-   И многое другое

## Основные встроенные библиотеки

Давайте рассмотрим несколько наиболее полезных встроенных библиотек Python.

### math — математические функции

Модуль `math` предоставляет доступ к математическим функциям, определенным в стандарте языка C:

```python
import math

# Константы
print(f"Число π: {math.pi}")
print(f"Число e: {math.e}")

# Тригонометрические функции
angle = math.pi / 4  # 45 градусов в радианах
print(f"Синус 45°: {math.sin(angle):.4f}")
print(f"Косинус 45°: {math.cos(angle):.4f}")

# Другие функции
print(f"Факториал 5: {math.factorial(5)}")
print(f"Наибольший общий делитель 12 и 18: {math.gcd(12, 18)}")
```

### random — генерация случайных чисел

Модуль `random` предоставляет функции для генерации случайных чисел и выбора случайных элементов:

```python
import random

# Генерация случайного целого числа в диапазоне
print(f"Случайное число от 1 до 10: {random.randint(1, 10)}")

# Случайное число с плавающей точкой от 0 до 1
print(f"Случайное число от 0 до 1: {random.random():.4f}")

# Выбор случайного элемента из последовательности
fruits = ["яблоко", "банан", "апельсин", "груша"]
print(f"Случайный фрукт: {random.choice(fruits)}")

# Перемешивание последовательности
numbers = [1, 2, 3, 4, 5]
random.shuffle(numbers)
print(f"Перемешанные числа: {numbers}")
```

### datetime — работа с датами и временем

Модуль `datetime` предоставляет классы для работы с датами и временем:

```python
import datetime

# Текущая дата и время
now = datetime.datetime.now()
print(f"Текущая дата и время: {now}")

# Создание конкретной даты
specific_date = datetime.date(2023, 12, 31)
print(f"Заданная дата: {specific_date}")

# Разница между датами
today = datetime.date.today()
new_year = datetime.date(today.year + 1, 1, 1)
days_until_new_year = (new_year - today).days
print(f"Дней до Нового года: {days_until_new_year}")

# Форматирование даты
formatted_date = now.strftime("%d.%m.%Y %H:%M")
print(f"Отформатированная дата: {formatted_date}")
```

### os — взаимодействие с операционной системой

Модуль `os` предоставляет функции для взаимодействия с операционной системой:

```python
import os

# Получение текущей рабочей директории
print(f"Текущая директория: {os.getcwd()}")

# Список файлов и папок в директории
files = os.listdir('.')
print(f"Первые 3 файла в текущей директории: {files[:3]}")

# Информация о системе
print(f"Имя операционной системы: {os.name}")

# Существует ли файл или директория
file_exists = os.path.exists('example.txt')
print(f"Файл example.txt существует: {file_exists}")
```

### json — работа с форматом JSON

Модуль `json` предоставляет функции для работы с данными в формате JSON:

```python
import json

# Словарь Python
person = {
    "name": "Иван",
    "age": 30,
    "city": "Москва",
    "languages": ["Python", "JavaScript", "SQL"]
}

# Преобразование словаря в строку JSON
person_json = json.dumps(person, ensure_ascii=False, indent=4)
print("JSON строка:")
print(person_json)

# Преобразование строки JSON в объект Python
json_string = '{"name": "Мария", "age": 25, "city": "Санкт-Петербург"}'
person_dict = json.loads(json_string)
print(f"Имя: {person_dict['name']}, Возраст: {person_dict['age']}")
```

### collections — специализированные типы данных

Модуль `collections` предоставляет альтернативные структуры данных для Python:

```python
from collections import Counter, defaultdict, namedtuple

# Counter - подсчет элементов
text = "Программирование на Python это интересно и приятно"
character_count = Counter(text.lower())
print("Три самых частых символа:")
for char, count in character_count.most_common(3):
    print(f"'{char}': {count}")

# defaultdict - словарь со значением по умолчанию
fruit_categories = defaultdict(list)
fruit_categories["желтые"].append("банан")
fruit_categories["красные"].append("яблоко")
fruit_categories["красные"].append("клубника")
print(f"Желтые фрукты: {fruit_categories['желтые']}")
print(f"Зеленые фрукты: {fruit_categories['зеленые']}")  # Пустой список

# namedtuple - именованные кортежи
Person = namedtuple('Person', ['name', 'age', 'job'])
alice = Person('Алиса', 30, 'инженер')
print(f"{alice.name}, {alice.age} лет, работает как {alice.job}")
```

## Проверка понимания

Давайте проверим, насколько хорошо вы усвоили тему встроенных библиотек:

**Какая библиотека лучше всего подходит для работы с датами в Python?**
