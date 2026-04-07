# Словари в Python

В этой статье мы разберем словари (dict) — один из самых мощных и гибких типов данных
в Python.

## Что такое словарь?

Словарь в Python — это коллекция пар "ключ-значение".
Можно представить словарь как телефонную книгу,
где имена людей (ключи) связаны с их номерами телефонов (значения).

Основные свойства словарей:

-   **Ключи должны быть уникальными** — нельзя иметь два одинаковых ключа в одном словаре
-   **Ключи должны быть неизменяемыми** — в качестве ключей можно использовать строки, числа, кортежи, но не списки или словари
-   **Значения могут быть любого типа** — числа, строки, списки, другие словари и т.д.
-   **Начиная с Python 3.7, словари сохраняют порядок добавления элементов** (раньше порядок не гарантировался)

## Создание словарей

Существует несколько способов создать словарь в Python:

### 1. С помощью фигурных скобок {}

```python
# Пустой словарь
empty_dict = {}

# Словарь с данными
person = {"name": "John", "age": 30, "city": "New York"}
print(person)

# Вложенные словари
nested = {
    "person": {"name": "Mary", "age": 25},
    "contacts": {"email": "mary@example.com", "phone": "+1234567890"}
}
print(nested["person"])
```

### 2. С помощью конструктора dict()

```python
# Пустой словарь
empty_dict = dict()

# Из списка кортежей (пар ключ-значение)
pairs = [("name", "Anna"), ("age", 28), ("city", "Berlin")]
person = dict(pairs)
print(person)

# С использованием именованных аргументов (только для строковых ключей)
settings = dict(theme="dark", font_size=12, notifications=True)
print(settings)
```

### 3. Генераторы словарей (dict comprehensions)

```python
# Словарь квадратов чисел (число: квадрат числа)
squares = {x: x**2 for x in range(1, 6)}
print(squares)
```

### 4. Метод fromkeys() для создания словаря с одинаковыми значениями

```python
# Создание словаря с заданными ключами и одинаковым значением для всех
keys = ["name", "age", "city"]
defaults = dict.fromkeys(keys, None)
print(defaults)
```

## Доступ к элементам словаря

Доступ к значениям в словаре осуществляется через ключи:

### Обращение по ключу

```python
person = {"name": "John", "age": 30, "city": "New York"}

# Получение значения по ключу
name = person["name"]
print(name)

# Безопасный способ с помощью метода get()
phone = person.get("phone")  # Вернет None, если ключа нет
print(phone)

# Метод get() с значением по умолчанию
phone = person.get("phone", "Not specified")
print(phone)
```

### Проверка наличия ключа

```python
person = {"name": "John", "age": 30, "city": "New York"}

# Проверка наличия ключа
print("name" in person)

print("phone" not in person)
```

### Доступ к вложенным словарям

```python
nested = {
    "person": {"name": "Mary", "age": 25},
    "contacts": {"email": "mary@example.com", "phone": "+1234567890"}
}

# Доступ к значениям во вложенных словарях
name = nested["person"]["name"]
print(name)

# Безопасный доступ с помощью get()
email = nested.get("contacts", {}).get("email")
print(email)

# Если путь не существует, вернется None или значение по умолчанию
website = nested.get("contacts", {}).get("website", "Not specified")
print(website)
```

## Изменение словарей

Словари в Python являются изменяемыми, поэтому их можно легко модифицировать:

### Добавление и изменение элементов

```python
# Создаем словарь
person = {"name": "John", "age": 30}

# Добавление нового ключа и значения
person["city"] = "New York"
print(person)

# Изменение значения существующего ключа
person["age"] = 31
print(person)

# Использование метода update() для массового обновления
person.update({"age": 32, "job": "developer", "language": "Python"})
print(person)
```

### Удаление элементов

```python
person = {"name": "John", "age": 30, "city": "New York", "job": "developer"}

# Удаление элемента по ключу с помощью del
del person["job"]
print(person)

# Удаление и возврат элемента с помощью pop()
age = person.pop("age")
print(age)
print(person)

# pop() с значением по умолчанию (не вызывает ошибку, если ключа нет)
job = person.pop("job", "Not specified")
print(job)

# Удаление и возврат произвольного элемента с помощью popitem()
# В Python 3.7+ возвращает последний добавленный элемент
item = person.popitem()
print(item)
print(person)

# Удаление всех элементов
person.clear()
print(person)
```

## Методы словарей

Python предоставляет множество полезных методов для работы со словарями:

### Получение ключей, значений и пар ключ-значение

```python
person = {"name": "John", "age": 30, "city": "New York"}

# Получение всех ключей
keys = person.keys()
print(keys)

# Получение всех значений
values = person.values()
print(values)

# Получение всех пар ключ-значение
items = person.items()
print(items)

# Объекты dict_keys, dict_values и dict_items являются представлениями словаря
# Они динамически отражают изменения в словаре
person["job"] = "developer"
print(keys)

# Преобразование представлений в списки
keys_list = list(keys)
print(keys_list)
```

### Копирование словарей

```python
import copy

# Словарь с вложенной структурой
original = {"name": "John", "settings": {"theme": "dark"}}

# Поверхностная и глубокая копии
shallow_copy = original.copy()
deep_copy = copy.deepcopy(original)

# Изменение простого ключа работает одинаково в обоих случаях
shallow_copy["name"] = "Peter"
deep_copy["name"] = "Alice"
print(f"original['name']: {original['name']}")  # Остается John

# Различия проявляются при изменении вложенных структур
shallow_copy["settings"]["theme"] = "light"
print(f"После shallow copy - original['settings']['theme']: {original['settings']['theme']}")  # Меняется!

# Создадим новый оригинал для демонстрации deep copy
original = {"name": "John", "settings": {"theme": "dark"}}
deep_copy = copy.deepcopy(original)
deep_copy["settings"]["theme"] = "light"
print(f"После deep copy - original['settings']['theme']: {original['settings']['theme']}")  # Не меняется
```

### Объединение словарей

```python
# Объединение словарей с update
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}

# Создаем копию dict1 и обновляем её данными из dict2
merged = dict1.copy()
merged.update(dict2)
print(merged)

# В Python 3.9+ можно использовать оператор |
# merged = dict1 | dict2
```

### Установка значения по умолчанию

```python
# Подсчет частоты слов
counts = {}
text = "one two one two three"
words = text.split()

# Способ 1: проверка наличия ключа
for word in words:
    if word in counts:
        counts[word] += 1
    else:
        counts[word] = 1
print(counts)

# Способ 2: использование get()
counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1
print(counts)
```

## Итерация по словарям

Существует несколько способов перебора элементов словаря:

```python
person = {"name": "John", "age": 30, "city": "New York"}

# Итерация по ключам (способ по умолчанию)
print("Итерация по ключам:")
for key in person:
    print(key, ":", person[key])

# Итерация по значениям
print("Итерация по значениям:")
for value in person.values():
    print(value)

# Итерация по парам ключ-значение
print("Итерация по парам ключ-значение:")
for key, value in person.items():
    print(key, ":", value)
```
