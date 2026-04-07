# SQLAlchemy Core: современный подход к SQL в Python

Мы изучили основы работы с SQLite через модуль `sqlite3`. Теперь познакомимся с **SQLAlchemy Core** — более мощным и удобным способом работы с базами данных в Python.

## Что такое SQLAlchemy Core?

**SQLAlchemy Core** — это Python-библиотека, которая предоставляет элегантный и мощный способ работы с SQL базами данных. Она действует как "умная обертка" над SQL, сохраняя полный контроль над запросами, но делая код более безопасным и переносимым.

### Основная идея

Вместо написания SQL-строк:

```sql
"SELECT * FROM users WHERE age > 25 AND city = 'Moscow'"
```

Вы пишете Python-код:

```python
select(users_table).where(
    (users_table.c.age > 25) & (users_table.c.city == 'Moscow')
)
```

**SQLAlchemy Core автоматически:**

-   **Защищает от SQL-инъекций** — параметры экранируются автоматически
-   **Генерирует правильный SQL** для разных баз данных (PostgreSQL, MySQL, SQLite)
-   **Проверяет синтаксис** на этапе написания кода
-   **Обеспечивает автокомплит** в IDE

## Почему SQLAlchemy Core?

**SQLAlchemy Core** решает ключевые проблемы работы с SQL в Python:

### Безопасность из коробки

**Проблема с `sqlite3`:**

```python
# ОПАСНО! Уязвимость к SQL-инъекциям
user_input = "'; DROP TABLE users; --"
cursor.execute(f"SELECT * FROM users WHERE name = '{user_input}'")
```

**Решение с SQLAlchemy Core:**

```python
# БЕЗОПАСНО! Автоматическая защита
select(users_table).where(users_table.c.name == user_input)
```

### Переносимость между БД

**Проблема:** Разный SQL-синтаксис в разных базах данных.

**Решение:** Один код работает везде:

```python
# Работает с PostgreSQL, MySQL, SQLite одинаково
select(users_table).where(users_table.c.age > 25).limit(10)
```

### Удобство разработки

-   **Автокомплит** — IDE подсказывает доступные поля и методы
-   **Проверка ошибок** — синтаксические ошибки видны сразу
-   **Читаемость** — код понятен без знания SQL-диалектов

## Установка SQLAlchemy

```bash
pip install sqlalchemy
```

Для работы с разными БД нужны дополнительные драйверы, но SQLite работает сразу.

## Основные концепции

### Engine — подключение к БД

```python
from sqlalchemy import create_engine

# Создаем подключение к SQLite
engine = create_engine('sqlite:///tasks.db', echo=True)

print("✅ Подключение создано!")
```

`echo=True` показывает выполняемые SQL-запросы — удобно для обучения!

### MetaData и Table — описание структуры

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, Boolean

# Создаем объекты для работы
engine = create_engine('sqlite:///tasks.db')
metadata = MetaData()

# Описываем таблицу как Python-объект
tasks_table = Table(
    'tasks',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String, nullable=False),
    Column('completed', Boolean, default=False)
)

# Создаем таблицу в БД
metadata.create_all(engine)

print("✅ Таблица создана!")
```

**Преимущества перед чистым SQL:**

-   **Читаемость** — структура таблицы понятна сразу
-   **Автокомплит** — IDE подсказывает поля и методы
-   **Переносимость** — работает с любой СУБД

## CRUD операции с SQLAlchemy Core

Рассмотрим все основные операции работы с данными в одном примере:

```python
from sqlalchemy import (
    create_engine, MetaData, Table, Column, Integer, String, Boolean,
    insert, select, update, delete
)

# === ПОДГОТОВКА ===
# Создаем подключение к базе данных
engine = create_engine('sqlite:///tasks.db')
metadata = MetaData()

# Описываем структуру таблицы
tasks_table = Table(
    'tasks', metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String, nullable=False),
    Column('completed', Boolean, default=False)
)

# Создаем таблицу в базе данных
metadata.create_all(engine)

with engine.connect() as connection:

    # === CREATE: Добавляем новые задачи ===
    insert_query = insert(tasks_table).values([
        {'title': 'Изучить SQLAlchemy Core', 'completed': False},
        {'title': 'Написать приложение', 'completed': False},
        {'title': 'Протестировать код', 'completed': False}
    ])

    result = connection.execute(insert_query)
    print(f"✅ CREATE: Добавлено {result.rowcount} задач")


    # === READ: Читаем все задачи ===
    select_query = select(tasks_table)
    result = connection.execute(select_query)
    tasks = result.fetchall()

    print("\n📋 READ: Список всех задач:")
    for task in tasks:
        status = "✅" if task.completed else "⏳"
        print(f"  {task.id}. {status} {task.title}")


    # === UPDATE: Отмечаем первую задачу как выполненную ===
    update_query = update(tasks_table).where(
        tasks_table.c.id == 1
    ).values(completed=True)

    result = connection.execute(update_query)
    print(f"\n🔄 UPDATE: Обновлено {result.rowcount} задач")


    # === READ: Проверяем изменения ===
    result = connection.execute(select_query)
    tasks = result.fetchall()

    print("\n📋 READ: Обновленный список:")
    for task in tasks:
        status = "✅" if task.completed else "⏳"
        print(f"  {task.id}. {status} {task.title}")


    # === DELETE: Удаляем задачу с ID=2 ===
    delete_query = delete(tasks_table).where(tasks_table.c.id == 2)
    result = connection.execute(delete_query)
    print(f"\n🗑️ DELETE: Удалено {result.rowcount} задач")


    # === READ: Финальный список ===
    result = connection.execute(select_query)
    tasks = result.fetchall()

    print("\n📋 READ: Финальный список:")
    for task in tasks:
        status = "✅" if task.completed else "⏳"
        print(f"  {task.id}. {status} {task.title}")

    # Сохраняем изменения
    connection.commit()

```

## Сравнение с чистым SQL

| Аспект            | Чистый SQL            | SQLAlchemy Core       |
| ----------------- | --------------------- | --------------------- |
| **Безопасность**  | Нужно помнить про `?` | Автоматическая защита |
| **Переносимость** | Привязка к одной СУБД | Работает везде        |
| **Читаемость**    | SQL-строки            | Python-объекты        |
| **Автокомплит**   | Нет                   | Есть в IDE            |
| **Сложность**     | Простой для начала    | Чуть сложнее          |

## Что мы изучили?

Теперь вы знаете основы SQLAlchemy Core:

**Концепции** — Engine, MetaData, Table  
**CRUD операции** — insert, select, update, delete  
**Преимущества** — безопасность, переносимость, удобство

Это мощная основа для работы с любыми базами данных в Python!

## Что дальше?

В следующей статье мы изучим **SQLAlchemy ORM** — еще более высокоуровневый подход, где таблицы становятся Python-классами, а записи — объектами. Это делает работу с данными максимально удобной.

**Главное преимущество SQLAlchemy Core перед чистым SQL?**
