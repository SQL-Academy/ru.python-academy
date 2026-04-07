# Форматы JSON и CSV в Python: чтение, запись и анализ данных

Мы уже рассмотрели основы работы с файлами и текстовыми файлами в Python.
Теперь мы переходим к структурированным данным — форматам, которые используются для хранения и обмена данными в структурированном виде. 🧩

Мы сосредоточимся на двух наиболее популярных форматах:

-   **JSON** (JavaScript Object Notation) — формат для обмена данными, широко используемый в веб-приложениях и API
-   **CSV** (Comma-Separated Values) — простой формат для представления табличных данных

Эти форматы очень распространены и используются во всех областях программирования — от веб-разработки до анализа данных.

## JSON: JavaScript Object Notation

> JSON (JavaScript Object Notation) — это текстовый формат обмена данными, похожий на словари и списки в Python. Он легко читается как человеком, так и машиной.

### Структура данных JSON

JSON поддерживает следующие типы данных:

-   Объекты (словари): `{"name": "Alice", "age": 30}`
-   Массивы (списки): `[1, 2, 3, 4]`
-   Строки: `"Hello, world!"`
-   Числа: `42`, `3.14`
-   Логические значения: `true`, `false`
-   `null` (в Python соответствует `None`)

### Модуль json в Python

Python предоставляет встроенный модуль `json` для работы с этим форматом:

```python
import json

# Простой словарь Python
person = {
    "name": "Анна",
    "age": 28,
    "city": "Москва",
    "languages": ["Python", "JavaScript"]
}

# Преобразование словаря Python в строку JSON
json_string = json.dumps(person, ensure_ascii=False, indent=2)
print("JSON строка:")
print(json_string)

# Преобразование строки JSON обратно в объект Python
parsed_data = json.loads(json_string)
print("\nПреобразовано обратно в Python:")
print(f"Имя: {parsed_data['name']}")
print(f"Возраст: {parsed_data['age']}")
print(f"Языки программирования: {', '.join(parsed_data['languages'])}")
```

### Запись JSON в файл и чтение из файла

Вот как можно сохранить данные в JSON-файл и затем прочитать их:

```python
import json

# Данные о студентах
students = [
    {"id": 1, "name": "Иван", "scores": [85, 90, 78]},
    {"id": 2, "name": "Мария", "scores": [92, 88, 95]}
]

# Запись в файл
with open('students.json', 'w', encoding='utf-8') as file:
    json.dump(students, file, ensure_ascii=False, indent=2)
    print("Данные записаны в файл students.json")

# Чтение из файла
with open('students.json', 'r', encoding='utf-8') as file:
    loaded_students = json.load(file)
    print(f"\nЗагружено {len(loaded_students)} студентов:")

    for student in loaded_students:
        avg_score = sum(student['scores']) / len(student['scores'])
        print(f"  {student['name']}: средний балл {avg_score:.1f}")
```

### Основные методы модуля json

| Метод                  | Описание                                |
| ---------------------- | --------------------------------------- |
| `json.dumps(obj)`      | Преобразует объект Python в строку JSON |
| `json.loads(str)`      | Преобразует строку JSON в объект Python |
| `json.dump(obj, file)` | Записывает объект Python в JSON-файл    |
| `json.load(file)`      | Читает JSON из файла в объект Python    |

Параметр `ensure_ascii=False` позволяет корректно сохранять русские буквы и другие символы Unicode, а `indent` делает вывод более читаемым.

## CSV: Comma-Separated Values

> CSV (Comma-Separated Values) — это простой текстовый формат для представления табличных данных, где строки таблицы — это строки файла, а столбцы разделены запятыми (или другими разделителями).

CSV выглядит примерно так:

```python
Имя,Возраст,Город
Анна,28,Москва
Иван,35,Санкт-Петербург
```

### Модуль csv в Python

Python предоставляет встроенный модуль `csv` для работы с этим форматом:

```python
import csv

# Данные для записи
data = [
    ['Имя', 'Возраст', 'Город'],  # Заголовки
    ['Анна', '28', 'Москва'],
    ['Иван', '35', 'Санкт-Петербург'],
    ['Мария', '22', 'Казань']
]

# Запись в CSV файл
with open('people.csv', 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerows(data)
    print("Данные записаны в файл people.csv")

# Чтение из CSV файла
with open('people.csv', 'r', encoding='utf-8') as file:
    reader = csv.reader(file)

    # Чтение заголовков (первая строка)
    headers = next(reader)
    print(f"\nЗаголовки: {headers}")

    # Чтение данных
    print("\nДанные:")
    for row in reader:
        print(f"  {row[0]}, {row[1]} лет, город {row[2]}")
```

### Использование DictReader и DictWriter

Для более удобной работы с CSV можно использовать `DictReader` и `DictWriter`,
которые позволяют работать с данными как со словарями:

```python
import csv

# Запись в CSV с использованием DictWriter
data = [
    {'Имя': 'Алексей', 'Профессия': 'Инженер', 'Зарплата': 85000},
    {'Имя': 'Екатерина', 'Профессия': 'Дизайнер', 'Зарплата': 75000},
    {'Имя': 'Сергей', 'Профессия': 'Программист', 'Зарплата': 110000}
]

with open('employees.csv', 'w', newline='', encoding='utf-8') as file:
    fieldnames = ['Имя', 'Профессия', 'Зарплата']
    writer = csv.DictWriter(file, fieldnames=fieldnames)

    writer.writeheader()  # Запись заголовков
    writer.writerows(data)  # Запись данных
    print("Данные сотрудников записаны в файл")

# Чтение из CSV с использованием DictReader
with open('employees.csv', 'r', encoding='utf-8') as file:
    reader = csv.DictReader(file)

    print("\nСотрудники:")
    for row in reader:
        print(f"  {row['Имя']} - {row['Профессия']}, зарплата: {row['Зарплата']} руб.")
```

### Основные особенности работы с CSV

1. **Разделители**: Хотя CSV расшифровывается как "значения, разделённые запятыми", на практике могут использоваться и другие разделители (точка с запятой, табуляция)
2. **Кавычки**: Если значение содержит разделитель или кавычки, оно заключается в кавычки
3. **Экранирование**: Если внутри значения есть кавычки, они экранируются

```python
import csv

# Пример с другим разделителем
data = [
    ['Товар', 'Цена', 'Наличие'],
    ['Ноутбук', '45000', 'Да'],
    ['Смартфон', '25000', 'Нет']
]

# Запись с использованием точки с запятой
with open('products.csv', 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file, delimiter=';')
    writer.writerows(data)
    print("Данные записаны с разделителем ';'")

# Чтение с указанием правильного разделителя
with open('products.csv', 'r', encoding='utf-8') as file:
    reader = csv.reader(file, delimiter=';')
    for row in reader:
        print('  '.join(row))
```

## Практический пример: Анализ данных о продажах

Рассмотрим пример, где мы сначала сохраняем данные о продажах в CSV, а затем анализируем их и сохраняем результаты в JSON:

```python
import csv
import json

# Создаем данные о продажах
sales = [
    ['Дата', 'Товар', 'Категория', 'Цена', 'Количество'],
    ['2023-01-05', 'Ноутбук HP', 'Электроника', '45000', '2'],
    ['2023-01-10', 'Смартфон Apple', 'Электроника', '85000', '3'],
    ['2023-01-15', 'Книга "Python"', 'Книги', '1200', '5'],
    ['2023-02-10', 'Микроволновка', 'Бытовая техника', '7000', '1']
]

# Шаг 1: Сохраняем данные в CSV
with open('sales.csv', 'w', newline='', encoding='utf-8') as file:
    writer = csv.writer(file)
    writer.writerows(sales)
    print("Данные о продажах сохранены в CSV")

# Шаг 2: Читаем и анализируем данные
with open('sales.csv', 'r', encoding='utf-8') as file:
    reader = csv.reader(file)
    headers = next(reader)  # Пропускаем заголовки

    # Подготовка переменных для анализа
    total_revenue = 0
    sales_by_category = {}

    # Анализ данных
    for row in reader:
        date, product, category, price, quantity = row
        revenue = float(price) * int(quantity)

        # Общая выручка
        total_revenue += revenue

        # Выручка по категориям
        if category in sales_by_category:
            sales_by_category[category] += revenue
        else:
            sales_by_category[category] = revenue

    # Вывод результатов анализа
    print(f"\nОбщая выручка: {total_revenue} руб.")
    print("\nВыручка по категориям:")
    for category, rev in sales_by_category.items():
        print(f"  {category}: {rev} руб.")

# Шаг 3: Сохраняем результаты анализа в JSON
results = {
    "total_revenue": total_revenue,
    "sales_by_category": sales_by_category
}

with open('sales_analysis.json', 'w', encoding='utf-8') as file:
    json.dump(results, file, ensure_ascii=False, indent=2)
    print("\nРезультаты анализа сохранены в JSON")

# Шаг 4: Проверяем сохраненный JSON
with open('sales_analysis.json', 'r', encoding='utf-8') as file:
    saved_results = json.load(file)
    print("\nСодержимое JSON файла с результатами:")
    print(json.dumps(saved_results, ensure_ascii=False, indent=2))
```

В этом примере мы:

1. Создали CSV-файл с данными о продажах
2. Прочитали данные и рассчитали выручку по категориям
3. Сохранили результаты анализа в JSON-файл
4. Прочитали сохраненный JSON, чтобы убедиться в корректности

## Проверка понимания

**Какой код правильно считывает данные из JSON файла в Python?**
