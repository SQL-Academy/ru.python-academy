# SQLAlchemy Core: SQL из Python-выражений

В прошлой статье мы выполняли SQL-запросы через `sqlite3`. Это работает, но есть две проблемы.

**Первая: SQL живёт в строке**, и любая опечатка или неаккуратная вставка пользовательских данных это потенциальная SQL-инъекция. Параметры `?` спасают, но про них нужно помнить каждый раз.

**Вторая: каждая СУБД имеет свой диалект SQL.** Если приложение пишется под SQLite, а потом переезжает на PostgreSQL — почти наверняка часть запросов придётся переписывать.

**SQLAlchemy Core** решает обе проблемы: SQL строится из Python-выражений, безопасность встроена по умолчанию, и один и тот же код работает с PostgreSQL, MySQL, SQLite. Простые `SELECT/WHERE` действительно похожи во всех СУБД, но как только заходим в специфические функции (даты, строки, агрегаты) или схему, синтаксис расходится, и Core переводит ваш Python в правильный диалект:

## Установка

```bash
pip install sqlalchemy
```

Для SQLite дополнительных драйверов не нужно. Для PostgreSQL ставится отдельно `psycopg2-binary`, для MySQL — `pymysql`.

## Engine: подключение

`Engine` это объект, отвечающий за связь с БД. Создаётся один раз на приложение:

```python-executable
from sqlalchemy import create_engine

engine = create_engine('sqlite:///tasks.db', echo=True)
print("Engine готов")
# Вывод: Engine готов
```

Параметр `echo=True` включает вывод выполняемых SQL-запросов в консоль. Удобно в обучении и при отладке, в production его выключают.

Строка подключения для других СУБД:

-   `postgresql://user:pass@host:5432/dbname`
-   `mysql+pymysql://user:pass@host/dbname`
-   `sqlite:///file.db`

## Описание таблицы

В Core структура таблицы описывается объектом `Table` — Python-эквивалент SQL-команды `CREATE TABLE`:

```python-executable
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, Boolean

engine = create_engine('sqlite:///tasks.db')
metadata = MetaData()

tasks_table = Table(
    'tasks',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String, nullable=False),
    Column('completed', Boolean, default=False),
)

# Создаём таблицу в БД (если её ещё нет)
metadata.create_all(engine)

print("Таблица tasks готова")
# Вывод: Таблица tasks готова
```

`MetaData` это коллекция всех `Table`-объектов приложения. `metadata.create_all(engine)` создаёт сразу все таблицы из коллекции, которых ещё нет в БД.

## CRUD: четыре операции

Для каждой операции в Core есть готовый помощник: `insert()`, `select()`, `update()`, `delete()`. Дальше предполагаем, что `engine` и `tasks_table` уже определены, как выше.

### INSERT

```python-executable
from sqlalchemy import insert

with engine.connect() as connection:
    result = connection.execute(
        insert(tasks_table),
        [
            {'title': 'Изучить SQLAlchemy Core', 'completed': False},
            {'title': 'Написать приложение', 'completed': False},
            {'title': 'Протестировать код', 'completed': False},
        ],
    )
    connection.commit()

print(f"Добавлено строк: {result.rowcount}")
# Вывод: Добавлено строк: 3
```

Значения передаются списком словарей — это batch-вставка одним запросом. SQLAlchemy сам подставит параметры безопасно.

### SELECT

```python-executable
from sqlalchemy import select

with engine.connect() as connection:
    result = connection.execute(select(tasks_table))
    for row in result:
        print(row.id, row.title, row.completed)
# Вывод:
# 1 Изучить SQLAlchemy Core 0
# 2 Написать приложение 0
# 3 Протестировать код 0
```

Доступ к колонкам по имени (`row.title`), а не по индексу как в `sqlite3`. Условие фильтрации добавляется через `.where()`:

```python-executable
from sqlalchemy import select

with engine.connect() as connection:
    result = connection.execute(
        select(tasks_table).where(tasks_table.c.id == 1)
    )
    row = result.first()
    print(row.title)
# Вывод: Изучить SQLAlchemy Core
```

`tasks_table.c.id` это «колонка `id` таблицы `tasks`». Сравнения (`==`, `>`, `<`, `.in_()`, `.like()`) превращаются в SQL автоматически.

### UPDATE

```python-executable
from sqlalchemy import update

with engine.connect() as connection:
    result = connection.execute(
        update(tasks_table)
        .where(tasks_table.c.id == 1)
        .values(completed=True)
    )
    connection.commit()

print(f"Обновлено строк: {result.rowcount}")
# Вывод: Обновлено строк: 1
```

### DELETE

```python-executable
from sqlalchemy import delete

with engine.connect() as connection:
    result = connection.execute(
        delete(tasks_table).where(tasks_table.c.id == 3)
    )
    connection.commit()

print(f"Удалено строк: {result.rowcount}")
# Вывод: Удалено строк: 1
```

## Что дальше?

В следующей статье возьмём **SQLAlchemy ORM** — слой выше Core, где таблицы становятся Python-классами, строки — объектами, и вам почти не нужно думать в терминах SQL. Хорошо подходит для типичной бизнес-логики; Core остаётся в арсенале для случаев, когда нужен точный контроль над запросом.

---

**Главное преимущество SQLAlchemy Core перед сырыми SQL-строками в `sqlite3`?**

