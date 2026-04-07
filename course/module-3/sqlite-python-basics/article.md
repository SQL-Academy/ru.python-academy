# SQLite в Python: основы работы с реляционными базами данных

Реляционные базы данных доминируют в современной разработке (5 из 8 позиций в топе популярных СУБД). Понимание их принципов работы — основа для эффективного использования любых инструментов работы с данными.

Мы изучим основы на примере **SQLite** — простой, но мощной реляционной БД, встроенной в Python.

## Зачем изучать SQL перед ORM?

Многие сразу переходят к изучению ORM (SQLAlchemy), но понимание основ SQL даст вам:

-   Понимание принципов работы реляционных БД
-   Умение отлаживать и оптимизировать запросы
-   Уверенность при работе с любыми СУБД

## О SQL в этом курсе

Мы **кратко рассмотрим основы** работы с SQLite, чтобы понять принципы. Для глубокого изучения SQL рекомендуем [бесплатный интерактивный курс SQL Academy](https://sql-academy.org/ru/guide) с практическими заданиями.

## Первые шаги с SQLite

SQLite идеален для изучения:

-   **Встроен в Python** — ничего устанавливать не нужно
-   **Файловая БД** — всё хранится в одном файле
-   **Полноценный SQL** — стандартные команды работают

Создадим простое приложение для управления задачами:

```python
import sqlite3

# Подключаемся к БД (файл создастся автоматически)
connection = sqlite3.connect('tasks.db')
cursor = connection.cursor()

print("✅ База данных создана!")
connection.close()
```

## Создание таблицы

```python
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()

    # Создаем таблицу для задач
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')

    print("✅ Таблица создана!")
```

**Что означает SQL:**

-   `CREATE TABLE` — создать таблицу
-   `IF NOT EXISTS` — только если её еще нет
-   `INTEGER PRIMARY KEY` — уникальный номер (ID)
-   `TEXT NOT NULL` — текстовое поле, обязательное для заполнения
-   `BOOLEAN DEFAULT FALSE` — логическое значение, по умолчанию `False`

## CRUD операции

### CREATE — Добавление данных

```python
import sqlite3

# Сначала создаем таблицу (если её нет)
with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')

def add_task(title):
    with sqlite3.connect('tasks.db') as connection:
        cursor = connection.cursor()
        cursor.execute(
            "INSERT INTO tasks (title) VALUES (?)",
            (title,)
        )
        print(f"✅ Задача '{title}' добавлена!")

# Добавляем задачи
add_task("Изучить SQLite")
add_task("Сделать покупки")
```

**Важно:** Символ `?` защищает от SQL-инъекций — всегда используйте его для вставки данных!

### READ — Чтение данных

```python
import sqlite3

# Создаем таблицу и добавляем тестовые данные
with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')
    # Добавляем данные для демонстрации
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (1, 'Изучить SQLite', FALSE)")
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (2, 'Сделать покупки', FALSE)")

def get_all_tasks():
    with sqlite3.connect('tasks.db') as connection:
        cursor = connection.cursor()
        cursor.execute("SELECT * FROM tasks")
        return cursor.fetchall()

def show_tasks():
    tasks = get_all_tasks()
    print("📋 Список задач:")
    for task in tasks:
        status = "✅" if task[2] else "⏳"
        print(f"{status} {task[1]}")

show_tasks()
```

### UPDATE — Обновление данных

```python
import sqlite3

# Создаем таблицу с данными
with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (1, 'Изучить SQLite', FALSE)")
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (2, 'Сделать покупки', FALSE)")

def complete_task(task_id):
    with sqlite3.connect('tasks.db') as connection:
        cursor = connection.cursor()
        cursor.execute(
            "UPDATE tasks SET completed = TRUE WHERE id = ?",
            (task_id,)
        )
        print(f"✅ Задача {task_id} выполнена!")

complete_task(1)
```

### DELETE — Удаление данных

```python
import sqlite3

# Создаем таблицу с данными
with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (1, 'Изучить SQLite', FALSE)")
    cursor.execute("INSERT OR IGNORE INTO tasks (id, title, completed) VALUES (2, 'Сделать покупки', FALSE)")

def delete_task(task_id):
    with sqlite3.connect('tasks.db') as connection:
        cursor = connection.cursor()
        cursor.execute("DELETE FROM tasks WHERE id = ?", (task_id,))
        print(f"🗑️ Задача {task_id} удалена!")

delete_task(2)
```

## Что дальше?

В следующей статье мы изучим **SQLAlchemy Core** — более мощный и удобный способ работы
с SQL в Python, который сохраняет контроль над запросами и добавляет множество полезных возможностей.
