---
meta:
    title: "SQLite: первая база данных в Python"
    description: "Подключение к SQLite из Python, создание таблицы, четыре операции CRUD и защита от SQL-инъекций через параметры."
---

# SQLite: первая база данных в Python

Сообщения в вашем телефоне, история браузера, настройки половины приложений на ноутбуке — огромная их часть лежит в **SQLite**: полноценной реляционной БД без сервера, встроенной прямо в Python. Ставить ничего не нужно, вся база — один файл, а тот же подход потом работает с любой другой СУБД.

На SQLite и разберём базовый приём работы с реляционной БД: как из Python отправить SQL-запрос и прочитать строки обратно в код.

## Подключение и cursor

Для работы с SQLite в стандартной библиотеке есть модуль `sqlite3`. Базовый паттерн — три шага:

- открыть **соединение** — открытую базу, как файл после `open()`;
- получить **курсор** — он отправляет в базу запросы и держит результат последнего;
- в конце закрыть соединение.

```python
import sqlite3

# Подключаемся к БД (файл создастся автоматически)
connection = sqlite3.connect('tasks.db')
cursor = connection.cursor()

# ... здесь будут запросы

connection.close()
print("Готово")
<output>
Готово
</output>
```

`Python-код` → SQL-запрос → `tasks.db` → строки-кортежи → `Python-код`

## Создание таблицы

В реляционной БД данные лежат в **таблицах**. Каждая таблица описывается схемой: какие столбцы, какого типа, какие ограничения. Создаём через SQL-команду `CREATE TABLE`.

Дальше во всех примерах соединение обёрнуто в `with`: SQLite записывает изменения не мгновенно, их нужно подтверждать (по-английски commit), и `with` делает это сам при выходе из блока, а при ошибке откатывает. Соединение он, правда, не закрывает, но в коротких скриптах, как ниже, оно закроется вместе с программой; в долгоживущем коде зовите `close()`, как в первом примере.

```python
import sqlite3

with sqlite3.connect('tasks.db') as connection:
    cursor = connection.cursor()
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY,
            title TEXT NOT NULL,
            completed BOOLEAN DEFAULT FALSE
        )
    ''')

print("Таблица tasks готова")
<output>
Таблица tasks готова
</output>
```

Что значат части SQL:

- `CREATE TABLE IF NOT EXISTS tasks` — создать таблицу `tasks`, если её ещё нет
- `id INTEGER PRIMARY KEY` — целочисленный первичный ключ; SQLite нумерует новые строки сам
- `title TEXT NOT NULL` — текстовое поле, обязательное
- `completed BOOLEAN DEFAULT FALSE` — логическое поле, по умолчанию `False`

## CRUD: четыре базовые операции

CRUD — акроним от **C**reate / **R**ead / **U**pdate / **D**elete: четыре операции, которые покрывают почти всю работу с данными.

Тренироваться будем на базе в памяти: `:memory:` вместо имени файла даёт чистую базу при каждом запуске. Первые строки внутри `with` — подготовка: таблица и две задачи.

### CREATE: добавление данных

```python
import sqlite3

with sqlite3.connect(':memory:') as connection:
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT)")

    cursor.execute(
        "INSERT INTO tasks (title) VALUES (?)",
        ("Изучить SQLite",)
    )
    cursor.execute(
        "INSERT INTO tasks (title) VALUES (?)",
        ("Сделать покупки",)
    )

print("Задачи добавлены")
<output>
Задачи добавлены
</output>
```

Значения передаются кортежем вторым аргументом `execute`, а в самом SQL вместо них стоит параметр `?`. Запятая в `("Изучить SQLite",)` обязательна: именно она делает скобки кортежем из одного элемента. Почему значения не вклеивают прямо в строку запроса — покажем сразу после того, как научимся читать данные.

### READ: чтение данных

```python
import sqlite3

with sqlite3.connect(':memory:') as connection:
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT)")
    cursor.execute("INSERT INTO tasks (title) VALUES ('Изучить SQLite'), ('Сделать покупки')")

    cursor.execute("SELECT id, title FROM tasks")
    rows = cursor.fetchall()

for row in rows:
    print(row)
<output>
(1, 'Изучить SQLite')
(2, 'Сделать покупки')
</output>
```

`cursor.fetchall()` возвращает все строки результата как список кортежей: доступ к полям по индексу, `row[0]` — это `id`, `row[1]` — `title`.

Когда нужна только одна строка, вместо `fetchall()` используют `fetchone()`: он возвращает первую строку результата. Увидим его в деле чуть ниже.

### SQL-инъекции: почему `?`

Теперь можно показать, зачем параметр `?` нужен на самом деле. Представьте поле поиска, куда пользователь вводит название задачи, — а злоумышленник вводит кусок SQL:

```python
# ОПАСНО: пользовательский ввод склеивается с SQL
search = "' OR '1'='1"
cursor.execute(f"SELECT * FROM tasks WHERE title = '{search}'")
# SQL превращается в: SELECT * FROM tasks WHERE title = '' OR '1'='1'
# условие '1'='1' истинно всегда → вернутся ВСЕ задачи, а не только нужная

# БЕЗОПАСНО: значение передаётся отдельно
cursor.execute("SELECT * FROM tasks WHERE title = ?", (search,))
# ищется задача с буквальным названием "' OR '1'='1" — лишнего не вернётся
```

Звёздочка в `SELECT *` — «все колонки разом». Злоумышленник подставил кусок SQL в обычное поле поиска и получил все строки таблицы; тем же приёмом обходят проверку пароля или удаляют данные. Правило: **никогда не склеивайте пользовательский ввод в SQL-строку**, всегда используйте параметры через `?`.

### UPDATE: обновление данных

```python
import sqlite3

with sqlite3.connect(':memory:') as connection:
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT, completed BOOLEAN DEFAULT FALSE)")
    cursor.execute("INSERT INTO tasks (title) VALUES ('Изучить SQLite'), ('Сделать покупки')")

    cursor.execute(
        "UPDATE tasks SET completed = ? WHERE id = ?",
        (True, 1)
    )

    cursor.execute("SELECT id, title, completed FROM tasks WHERE id = ?", (1,))
    print(cursor.fetchone())
<output>
(1, 'Изучить SQLite', 1)
</output>
```

Первая задача теперь выполнена: логические значения SQLite хранит как `0` и `1`, и `completed` сменился с нуля на единицу. А `WHERE id = ?` обязательно: без условия `UPDATE` обновит **все** строки таблицы.

### DELETE: удаление данных

```python
import sqlite3

with sqlite3.connect(':memory:') as connection:
    cursor = connection.cursor()
    cursor.execute("CREATE TABLE tasks (id INTEGER PRIMARY KEY, title TEXT)")
    cursor.execute("INSERT INTO tasks (title) VALUES ('Изучить SQLite'), ('Сделать покупки')")

    cursor.execute("DELETE FROM tasks WHERE id = ?", (2,))

    cursor.execute("SELECT id, title FROM tasks")
    print(cursor.fetchall())
<output>
[(1, 'Изучить SQLite')]
</output>
```

Вторая задача исчезла — осталась одна строка. Та же история, что с `UPDATE`: без `WHERE` команда `DELETE` удалит **все** строки таблицы.

## Что осталось за кадром

В реальном production-коде есть несколько важных тем, которые мы здесь не разбираем подробно, но о которых стоит знать:

- **Транзакции** (`BEGIN`/`COMMIT`/`ROLLBACK`): группа изменений выполняется атомарно — либо все, либо ни одной. `with sqlite3.connect(...)` коммитит автоматически при выходе из блока.
- **JOIN и выборки по нескольким таблицам сразу**: большинство реальных схем содержат несколько связанных таблиц (пользователи и их задачи, заказы и товары), и данные из них достают одним запросом.
- **Индексы**: ускоряют поиск по часто используемым колонкам.

Эти темы покрывает [курс SQL Academy](https://sql-academy.org/ru/guide).

## Проверка понимания

**Почему нужно передавать значения в `execute()` через параметр `?`, а не вставлять напрямую в SQL-строку?**

1. **Правильный ответ:** Для защиты от SQL-инъекций — Текст запроса и значения едут в базу по разным каналам: запрос разбирается первым, а плейсхолдер — это дырка под значение. Что бы в неё ни положили, как SQL это уже не прочитается, поэтому враждебный ввод остаётся просто строкой в поле, а не командой.

2. Для ускорения работы запроса — Влияние на скорость есть (БД может закешировать план запроса), но это побочный эффект. Главная причина это безопасность.

3. Чтобы SQLite мог автоматически преобразовать типы — Преобразование типов выполняется в обоих случаях. Главная причина параметризованных запросов это защита от SQL-инъекций.

4. Это требование стандарта SQL — Плейсхолдер «?» — это особенность драйвера sqlite3, а не SQL-стандарта. В других СУБД плейсхолдеры другие (%s, :name). Главная причина их использования — безопасность.

В следующей статье возьмём **SQLAlchemy Core**: это библиотека, которая позволяет строить SQL-запросы из Python-выражений вместо строк. SQL-инъекции там защищены автоматически, а один и тот же код работает с PostgreSQL, MySQL и SQLite.
