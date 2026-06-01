# SQLite в Python: основы работы с реляционными базами данных

Большинство production-приложений хранят данные не в файлах, а в **реляционных базах данных**: PostgreSQL, MySQL, SQLite. Это значит — рано или поздно вы будете писать SQL-запросы из Python и читать результаты обратно в код.

В этой статье разберём базовый паттерн на примере **SQLite**. Это полноценная реляционная БД, встроенная в Python из коробки: ничего устанавливать не нужно, БД хранится в одном файле, и тот же подход работает с любой другой СУБД.

Эта статья даст **общее представление** о работе с SQL из Python. Для глубокого изучения самого SQL рекомендуем [бесплатный курс SQL Academy](https://sql-academy.org/ru/guide) с практическими заданиями.

## Подключение и cursor

Для работы с SQLite в стандартной библиотеке есть модуль `sqlite3`. Базовый паттерн: создаём соединение, получаем `cursor` (через него выполняем запросы), закрываем соединение.

```python-executable
import sqlite3

# Подключаемся к БД (файл создастся автоматически)
connection = sqlite3.connect('tasks.db')
cursor = connection.cursor()

# ... здесь будут запросы

connection.close()
print("Готово")
# Вывод: Готово
```

Удобнее использовать соединение как контекстный менеджер (`with`) — он сам закроет соединение и сохранит изменения:

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    # ... запросы

print("Готово")
# Вывод: Готово
```

## Создание таблицы

В реляционной БД данные лежат в **таблицах**. Каждая таблица описывается схемой: какие столбцы, какого типа, какие ограничения. Создаём через SQL-команду `CREATE TABLE`:

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')

print("Таблица tasks готова")
# Вывод: Таблица tasks готова
```

Что значат части SQL:

-   `CREATE TABLE IF NOT EXISTS tasks` — создать таблицу `tasks`, если ещё нет
-   `id INTEGER PRIMARY KEY AUTOINCREMENT` — целочисленный первичный ключ с автоинкрементом
-   `title TEXT NOT NULL` — текстовое поле, обязательное
-   `completed BOOLEAN DEFAULT FALSE` — логическое поле, по умолчанию `False`

## CRUD: четыре базовые операции

CRUD это акроним от **C**reate / **R**ead / **U**pdate / **D**elete. Это четыре операции, которые покрывают почти всю работу с данными. Дальше предполагаем, что таблица `tasks` уже создана.

### CREATE: добавление данных

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute(
        "INSERT INTO tasks (title) VALUES (?)",
        ("Изучить SQLite",)
    )
    cursor.execute(
        "INSERT INTO tasks (title) VALUES (?)",
        ("Сделать покупки",)
    )

print("Задачи добавлены")
# Вывод: Задачи добавлены
```

Здесь принципиально важно: значения **не вставляются прямо в строку SQL** через f-string или конкатенацию. Вместо этого используется параметр `?`, а значение передаётся вторым аргументом в `execute`. Это защищает от **SQL-инъекций**:

```python
# ОПАСНО: пользовательский ввод склеивается с SQL
user_input = "'; DROP TABLE tasks; --"
cursor.execute(f"INSERT INTO tasks (title) VALUES ('{user_input}')")
# выполнится: INSERT ... VALUES (''); DROP TABLE tasks; --')

# БЕЗОПАСНО: значение передаётся отдельно
cursor.execute("INSERT INTO tasks (title) VALUES (?)", (user_input,))
# выполнится: INSERT ... VALUES ('\'; DROP TABLE tasks; --')
```

Правило: **никогда не склеивайте пользовательский ввод в SQL-строку**, всегда используйте параметры через `?`.

### READ: чтение данных

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute("SELECT id, title, completed FROM tasks")
    rows = cursor.fetchall()

for row in rows:
    print(row)
# Вывод:
# (1, 'Изучить SQLite', 0)
# (2, 'Сделать покупки', 0)
```

`cursor.fetchall()` возвращает все строки результата как список кортежей. Доступ к полям по индексу: `row[0]` это `id`, `row[1]` это `title` и т.д.

Если нужно достать только одну строку (например по id), используется `fetchone()`:

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute("SELECT title FROM tasks WHERE id = ?", (1,))
    row = cursor.fetchone()

print(row)
# Вывод: ('Изучить SQLite',)
```

### UPDATE: обновление данных

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute(
        "UPDATE tasks SET completed = ? WHERE id = ?",
        (True, 1)
    )

print("Задача 1 помечена выполненной")
# Вывод: Задача 1 помечена выполненной
```

`WHERE id = ?` обязательно: без условия `UPDATE` обновит **все** строки в таблице.

### DELETE: удаление данных

```python-executable
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute("DELETE FROM tasks WHERE id = ?", (2,))

print("Задача 2 удалена")
# Вывод: Задача 2 удалена
```

Та же история: без `WHERE` `DELETE` удалит **всю** таблицу строк.

## Что осталось за кадром

В реальном production-коде есть несколько важных тем, которые мы здесь не разбираем подробно, но о которых стоит знать:

-   **Транзакции** (`BEGIN`/`COMMIT`/`ROLLBACK`): группа изменений выполняется атомарно — либо все, либо ни одной. `with sqlite3.connect(...)` коммитит автоматически при выходе из блока.
-   **Связи между таблицами** (`FOREIGN KEY`, `JOIN`): большинство реальных схем содержат несколько связанных таблиц (пользователи и их задачи, заказы и товары).
-   **Индексы**: ускоряют поиск по часто используемым колонкам.

Эти темы покрывает [курс SQL Academy](https://sql-academy.org/ru/guide).

## Что дальше?

В следующей статье возьмём **SQLAlchemy Core**: это библиотека, которая позволяет строить SQL-запросы из Python-выражений вместо строк. SQL-инъекции там защищены автоматически, а один и тот же код работает с PostgreSQL, MySQL и SQLite.

---

**Почему нужно передавать значения в `execute()` через параметр `?`, а не вставлять напрямую в SQL-строку?**

