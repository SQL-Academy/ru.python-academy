---
meta:
    title: "SQLAlchemy ORM: работа с БД через Python-объекты"
    description: "Модели как классы, строки как объекты: DeclarativeBase и Mapped, Session, CRUD через атрибуты и связи relationship."
---

# SQLAlchemy ORM: работа с БД через Python-объекты

В Core мы строили SQL из Python-выражений вида `select(...).where(...)`. Это уже сильно лучше сырых SQL-строк, но в коде всё равно остаются «таблица + колонка», а не привычные объекты.

**ORM** (Object-Relational Mapping) идёт на шаг дальше: таблица описывается как Python-класс, строка таблицы — экземпляр этого класса, а изменение атрибута объекта автоматически отражается в БД. Получается работа с БД на языке обычных Python-объектов.

**Python-объект:** `Task(id=1, title="Изучить SQLite")`

**строка в tasks:** `1 | Изучить SQLite`

Атрибуты объекта соответствуют столбцам строки, а Session синхронизирует обе стороны

## Модель: класс как таблица

В SQLAlchemy 2.0+ модели описываются через `DeclarativeBase` с аннотациями типов:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column

class Base(DeclarativeBase):
    pass

class Task(Base):
    __tablename__ = 'tasks'

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    completed: Mapped[bool] = mapped_column(default=False)

    def __repr__(self):
        return f"Task(id={self.id}, title={self.title!r}, completed={self.completed})"

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

print("Модель Task и таблица tasks готовы")
<output>
Модель Task и таблица tasks готовы
</output>
```

Как это читать:

- `class Task(Base)` — модель; `Base` — общий родитель всех моделей, через него SQLAlchemy собирает список таблиц, как `MetaData` в Core;
- `__tablename__` — имя таблицы в БД;
- строки вида `title: Mapped[str]` — колонки: `Mapped[...]` помечает «это колонка таблицы», а тип в скобках становится типом колонки (`int` → INTEGER, `str` → VARCHAR, `bool` → BOOLEAN);
- `mapped_column(...)` дописывают, только когда у колонки есть настройки — первичный ключ или значение по умолчанию;
- `__repr__` — знакомый спецметод: как объект показывает себя при печати.

Поэтому `Base.metadata.create_all(engine)` выглядит знакомо: это та же `create_all`, что и в прошлой главе.

## Session: единица работы

Для запросов в ORM используется `Session` — «единица работы»: она держит загруженные объекты в памяти, отслеживает изменения и одной командой сохраняет всё в БД.

Открывают её через `with Session(engine) as session:` — так сессия закроется сама, а `commit()` сохраняет накопленные изменения.

## CRUD через объекты

Третий раз те же четыре операции — но теперь вы не пишете ни SQL, ни выражений: меняете Python-объекты, а Session сам превращает это в нужные запросы.

CREATE, READ и DELETE узнаются с ходу; главное новое в ORM — отслеживание изменений и связи между таблицами.

База в примерах создаётся в памяти: строка подключения `sqlite:///:memory:` даёт чистую базу при каждом запуске.

### CREATE: создание

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

# Подготовка: модель и таблица из начала статьи
class Base(DeclarativeBase):
    pass

class Task(Base):
    __tablename__ = 'tasks'

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    completed: Mapped[bool] = mapped_column(default=False)

    def __repr__(self):
        return f"Task(id={self.id}, title={self.title!r}, completed={self.completed})"

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

with Session(engine) as session:
    task1 = Task(title="Изучить ORM")
    task2 = Task(title="Написать код")
    session.add_all([task1, task2])
    session.commit()
    print(task1)
    print(task2)
<output>
Task(id=1, title='Изучить ORM', completed=False)
Task(id=2, title='Написать код', completed=False)
</output>
```

Заметьте: `task1.id` после `commit()` уже заполнен. БД назначила его автоматически.

### READ: чтение

Запросы пишутся знакомым по Core `select()`, а выполняет их `session.execute()`:

```python
from sqlalchemy import create_engine, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

# Подготовка: модель, таблица и три задачи (одна уже выполнена)
class Base(DeclarativeBase):
    pass

class Task(Base):
    __tablename__ = 'tasks'

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    completed: Mapped[bool] = mapped_column(default=False)

    def __repr__(self):
        return f"Task(id={self.id}, title={self.title!r}, completed={self.completed})"

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

with Session(engine) as session:
    session.add_all([
        Task(title="Изучить ORM"),
        Task(title="Написать код"),
        Task(title="Сдать проект", completed=True),
    ])
    session.commit()

with Session(engine) as session:
    # Все строки
    stmt = select(Task)
    tasks = session.execute(stmt).scalars().all()
    for task in tasks:
        print(task)
<output>
Task(id=1, title='Изучить ORM', completed=False)
Task(id=2, title='Написать код', completed=False)
Task(id=3, title='Сдать проект', completed=True)
</output>

    # Одна запись по первичному ключу
    task = session.get(Task, 1)
    print(task)
<output>
Task(id=1, title='Изучить ORM', completed=False)
</output>

    # С фильтром: только невыполненные
    pending = session.execute(select(Task).where(Task.completed == False)).scalars().all()
    for task in pending:
        print(task)
<output>
Task(id=1, title='Изучить ORM', completed=False)
Task(id=2, title='Написать код', completed=False)
</output>
```

`.scalars()` нужен, потому что `select(Task)` возвращает строки-кортежи: он распаковывает их в объекты `Task`.

Одну запись по первичному ключу быстрее всего достаёт `session.get`.

Фильтр пишется как в Core, только вместо `tasks_table.c.completed` — атрибут класса `Task.completed`: третья задача выполнена, и в выборку она не попала.

### UPDATE и DELETE

Самая удобная часть ORM: меняем атрибут объекта, и Session сам понимает, что нужно обновить. Удаление — через `session.delete`:

```python
from sqlalchemy import create_engine, select
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, Session

# Подготовка: модель, таблица и две задачи
class Base(DeclarativeBase):
    pass

class Task(Base):
    __tablename__ = 'tasks'

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    completed: Mapped[bool] = mapped_column(default=False)

    def __repr__(self):
        return f"Task(id={self.id}, title={self.title!r}, completed={self.completed})"

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

with Session(engine) as session:
    session.add_all([Task(title="Изучить ORM"), Task(title="Написать код")])
    session.commit()

with Session(engine) as session:
    task = session.get(Task, 1)
    task.completed = True
    session.commit()

    task2 = session.get(Task, 2)
    session.delete(task2)
    session.commit()

    tasks = session.execute(select(Task)).scalars().all()
    for task in tasks:
        print(task)
<output>
Task(id=1, title='Изучить ORM', completed=True)
</output>
```

Никаких явных `UPDATE ... SET` и `DELETE ... WHERE`: Session отслеживает изменённые атрибуты и удалённые объекты и при `commit()` отправляет нужный SQL. Осталась одна задача, и она уже выполнена.

## Связи между таблицами

В реальных схемах таблицы связаны: у пользователя есть задачи, у поста комментарии.

Связь держится на **внешнем ключе**: в таблице `user_tasks` есть столбец `user_id`, куда кладётся `id` пользователя-владельца, — так строка задачи знает, чья она. Запись `ForeignKey("users.id")` говорит базе, что значение в этом столбце обязано существовать в `users.id`, иначе задача окажется ничьей.

`relationship` — надстройка ORM над этим столбцом: вместо ручного поиска всех строк с нужным `user_id` вы пишете `user.tasks`.

`back_populates` связывает две стороны, чтобы `user.tasks` и `task.user` описывали одну и ту же связь, а не две независимые. В коде обращение к связанным записям выглядит как обращение к обычному атрибуту:

```python
from sqlalchemy import create_engine, ForeignKey
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    tasks: Mapped[list["UserTask"]] = relationship(back_populates="user")

class UserTask(Base):
    __tablename__ = 'user_tasks'
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    user: Mapped["User"] = relationship(back_populates="tasks")

engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)

with Session(engine) as session:
    anna = User(name="Анна", tasks=[
        UserTask(title="Изучить Python"),
        UserTask(title="Написать код"),
    ])
    session.add(anna)
    session.commit()

    user = session.get(User, anna.id)
    print(user.name)
    for task in user.tasks:
        print(f"  {task.title}")
<output>
Анна
  Изучить Python
  Написать код
</output>
```

Имя `"UserTask"` в аннотации стоит в кавычках, потому что этот класс объявлен ниже по файлу: в момент чтения `User` Python его ещё не знает.

`user.tasks` за кулисами выполняет SQL-запрос: выбирает из `user_tasks` строки с нужным `user_id`. Но в коде это выглядит как обычный доступ к атрибуту. Это и есть главный комфорт ORM: реляционная связь читается как «у пользователя есть задачи».

## Сравнение трёх подходов

| Аспект                 | sqlite3              | SQLAlchemy Core        | SQLAlchemy ORM    |
| ---------------------- | -------------------- | ---------------------- | ----------------- |
| Запрос                 | SQL-строка           | Python-выражение       | Python-объект     |
| Защита от инъекций     | через `?` вручную    | автоматически          | автоматически     |
| Переносимость между БД | нет                  | есть                   | есть              |
| Связи                  | JOIN вручную         | JOIN-выражения         | `user.tasks`      |
| UPDATE                 | `UPDATE ... SET ...` | `update().values(...)` | `obj.field = ...` |
| Контроль над SQL       | максимальный         | высокий                | средний           |

Хорошее правило:

- ORM — для типичной бизнес-логики;
- Core — для сложных запросов, где нужен контроль;
- сырой SQL — только когда первые два не справляются.

## Проверка понимания

**Главное преимущество ORM перед Core?**

1. Запросы выполняются быстрее — Наоборот, ORM добавляет небольшие накладные расходы по сравнению с Core (отслеживание объектов). Преимущество в удобстве, а не в скорости.

2. **Правильный ответ:** Работа с обычными Python-объектами вместо SQL-выражений, автоматическое отслеживание изменений — Изменили атрибут объекта — Session сам сделает нужный UPDATE при commit(). Связи между таблицами читаются как атрибуты (user.tasks), а не как явные JOIN.

3. Возможность работать без понимания SQL — Понимание SQL всё равно нужно: ORM выполняет SQL под капотом, и без понимания, что он делает, легко получить лишние медленные запросы. Преимущество ORM — удобство, а не «магия избавления от SQL».

4. ORM проверяет схему БД во время выполнения — Это скорее особенность типизированных моделей (Mapped\[str]), но проверка происходит не во время выполнения, а в IDE или линтере. Главное преимущество ORM в работе с объектами вместо SQL.

ORM — инструмент, который оптимизирует **типичные** случаи работы с БД. Если в проекте 95% запросов — «получи объект, поменяй поле, сохрани», ORM экономит кучу времени. Когда упираетесь в сложный запрос или узкое место по скорости, спускайтесь в Core или пишите SQL напрямую. Эти три уровня дополняют друг друга.
