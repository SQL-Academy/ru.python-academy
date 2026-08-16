---
meta:
    title: "SQLAlchemy Core: SQL из Python-выражений"
    description: "Как строить SQL из Python-выражений: Engine, Table, помощники insert/select/update/delete и один код для разных СУБД."
---

# SQLAlchemy Core: SQL из Python-выражений

В прошлой главе SQL-запросы жили в строках, и пока запрос постоянный, со строкой нет проблем. Но чаще запрос зависит от данных: скажем, найти задачу по названию, которое ввёл пользователь. Первым в голову приходит f-строка:

```python
query = f"SELECT * FROM tasks WHERE title = '{name}'"
```

Работает ровно до того дня, когда в `name` прилетает апостроф или чужой кусок SQL. За удобством прячутся две проблемы сырых строк.

**Первая: SQL живёт в строке**, и любая неаккуратная вставка пользовательских данных — потенциальная SQL-инъекция. Параметры `?` спасают, но про них нужно помнить каждый раз.

**Вторая: каждая СУБД имеет свой диалект SQL.** Если приложение пишется под SQLite, а потом переезжает на PostgreSQL — почти наверняка часть запросов придётся переписывать.

**SQLAlchemy Core** решает обе проблемы: SQL строится из Python-выражений, безопасность встроена по умолчанию, и один и тот же код работает с PostgreSQL, MySQL, SQLite. Простые `SELECT/WHERE` действительно похожи во всех СУБД, но как только заходим в специфические функции (даты, строки, агрегаты), синтаксис расходится, и Core переводит ваш Python в правильный диалект:

Одно выражение — три диалекта: Postgres и MySQL совпали, SQLite получил свой вариант, и плейсхолдер у каждого свой

Новых слов в Core по сути четыре:

- `Engine` — подключение к базе;
- `Table` — описание таблицы;
- `insert`/`select`/`update`/`delete` — помощники вместо SQL-строк;
- `.c` — доступ к колонкам.

Всё остальное — знакомый по прошлой главе SQL.

## Установка

```bash
pip install sqlalchemy
```

Для SQLite дополнительных драйверов не нужно. Для PostgreSQL ставится отдельно `psycopg2-binary`, для MySQL — `pymysql`.

## Engine: подключение

`Engine` — это объект, отвечающий за связь с БД. Создаётся один раз на приложение, а адрес базы задаётся строкой подключения:

```python
from sqlalchemy import create_engine

engine = create_engine('sqlite:///core_tasks.db')
print("Engine готов")
<output>
Engine готов
</output>
```

У `create_engine` есть параметр `echo=True`: с ним каждый выполняемый SQL печатается в консоль. Удобно при отладке, в production выключают.

Строка подключения для других СУБД:

- `postgresql://user:pass@host:5432/dbname`
- `mysql+pymysql://user:pass@host/dbname`
- `sqlite:///file.db`

## Описание таблицы

В Core структура таблицы описывается объектом `Table` — Python-эквивалент SQL-команды `CREATE TABLE`:

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, Boolean

engine = create_engine('sqlite:///core_tasks.db')
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
<output>
Таблица tasks готова
</output>
```

`nullable=False` — тот же `NOT NULL` из прошлой главы: поле обязательное. `MetaData` — коллекция всех `Table`-объектов приложения. `metadata.create_all(engine)` создаёт сразу все таблицы из коллекции, которых ещё нет в БД.

## Те же четыре операции, но выражениями

Те же CREATE, READ, UPDATE, DELETE, что и в прошлой главе, — только вместо SQL-строк Python-выражения. Для каждой операции в Core есть помощник: `insert()`, `select()`, `update()`, `delete()`.

Новое в знакомой четвёрке: пакетная вставка списком, доступ к колонкам через `.c` и Python-операторы в `.where()`. База в примерах создаётся в памяти: строка подключения `sqlite:///:memory:` даёт чистую базу при каждом запуске.

### INSERT и SELECT

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, Boolean
from sqlalchemy import insert, select

# Подготовка: движок и таблица из примеров выше
engine = create_engine('sqlite:///:memory:')
metadata = MetaData()

tasks_table = Table(
    'tasks', metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String, nullable=False),
    Column('completed', Boolean, default=False),
)

metadata.create_all(engine)

with engine.connect() as connection:
    result = connection.execute(
        insert(tasks_table),
        [
            {'title': 'Изучить SQLAlchemy Core'},
            {'title': 'Написать приложение'},
        ],
    )
    connection.commit()
    print(f"Добавлено строк: {result.rowcount}")
<output>
Добавлено строк: 2
</output>

    result = connection.execute(select(tasks_table))
    for row in result:
        print(row.id, row.title, row.completed)
<output>
1 Изучить SQLAlchemy Core False
2 Написать приложение False
</output>
```

Значения в `insert` передаются списком словарей: это batch-вставка одним запросом, и параметры SQLAlchemy подставит безопасно сам.

Колонку `completed` мы не указываем, потому что сработало `default=False` из описания таблицы.

Строки читаются доступом по имени (`row.title`), а не по индексу, как в `sqlite3`. И `completed` вернулся как `False`, а не `0`: SQLAlchemy знает тип колонки (`Boolean`) и сам приводит значение к Python-типу.

### UPDATE и DELETE

Условие фильтрации добавляется методом `.where()`, строки обновляет помощник `update()`, удаляет — `delete()`:

```python
from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String, Boolean
from sqlalchemy import insert, select, update, delete

# Подготовка: движок, таблица и две задачи
engine = create_engine('sqlite:///:memory:')
metadata = MetaData()

tasks_table = Table(
    'tasks', metadata,
    Column('id', Integer, primary_key=True),
    Column('title', String, nullable=False),
    Column('completed', Boolean, default=False),
)

metadata.create_all(engine)

with engine.connect() as connection:
    connection.execute(insert(tasks_table), [
        {'title': 'Изучить SQLAlchemy Core'},
        {'title': 'Написать приложение'},
    ])
    connection.commit()

    connection.execute(
        update(tasks_table)
        .where(tasks_table.c.id == 1)
        .values(completed=True)
    )
    connection.execute(
        delete(tasks_table).where(tasks_table.c.id == 2)
    )
    connection.commit()

    result = connection.execute(select(tasks_table))
    for row in result:
        print(row.id, row.title, row.completed)
<output>
1 Изучить SQLAlchemy Core True
</output>
```

`tasks_table.c.id` — «колонка `id` таблицы `tasks`», а `==` превращается в SQL-сравнение автоматически; так же работают `>`, `<`, `.in_()` и `.like()`.

Финальный `select` подтверждает: первая задача осталась одна и уже с `completed=True`. Обе операции сработали.

## Проверка понимания

**Главное преимущество SQLAlchemy Core перед сырыми SQL-строками в `sqlite3`?**

1. Запросы выполняются быстрее — Производительность примерно одинаковая, иногда Core даже чуть медленнее за счёт построения SQL из выражения. Главное преимущество не в скорости.

2. **Правильный ответ:** Защита от SQL-инъекций и переносимость между разными СУБД — Значение уходит в драйвер отдельно от текста запроса, подставлять его в строку вручную не нужно, а один и тот же Python-код работает с PostgreSQL, MySQL и SQLite — Core сам подстраивает SQL под выбранную СУБД.

3. Возможность вообще не писать SQL — Знание SQL по-прежнему нужно: Core напрямую соответствует SQL-операциям select, insert, update, delete. Просто синтаксис Python вместо строк.

4. Автоматическая генерация документации к схеме — Core не генерирует документацию. Главные преимущества — безопасность и переносимость между СУБД.

В следующей статье возьмём **SQLAlchemy ORM** — слой выше Core, где таблицы становятся Python-классами, строки — объектами, и вам почти не нужно думать в терминах SQL. Хорошо подходит для типичной бизнес-логики; Core остаётся в арсенале для случаев, когда нужен точный контроль над запросом.
