# SQLAlchemy ORM: работа с БД через Python-объекты

В Core мы строили SQL из Python-выражений: `select(tasks).where(tasks.c.id == 1)`. Это уже сильно лучше сырых SQL-строк, но в коде всё равно остаются «таблица + колонка», а не привычные объекты.

**ORM** (Object-Relational Mapping) идёт на шаг дальше: таблица описывается как Python-класс, строка таблицы это экземпляр этого класса, а изменение атрибута объекта автоматически отражается в БД. Получается работа с БД на языке обычных Python-объектов.

## Модель: класс как таблица

В SQLAlchemy 2.0+ модели описываются через `DeclarativeBase` с аннотациями типов. Это современный стиль, который заменяет старый `declarative_base()`:

```python-executable
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

engine = create_engine('sqlite:///orm_tasks.db')
Base.metadata.create_all(engine)

print("Модель Task и таблица tasks готовы")
# Вывод: Модель Task и таблица tasks готовы
```

Как это читать:

-   `class Task(Base)` — модель, наследник базового класса
-   `__tablename__ = 'tasks'` — имя таблицы в БД
-   `id: Mapped[int] = mapped_column(primary_key=True)` — колонка `id`, тип int, первичный ключ
-   `title: Mapped[str]` — колонка `title`, тип str, NOT NULL по умолчанию
-   `completed: Mapped[bool] = mapped_column(default=False)` — колонка `completed`, тип bool, по умолчанию `False`

Типы Python (`int`, `str`, `bool`) автоматически отображаются в SQL-типы (`INTEGER`, `VARCHAR`, `BOOLEAN`). Никаких отдельных `Column(Integer, ...)` как в Core.

## Session: единица работы

Для запросов в ORM используется `Session`. Это «единица работы»: она держит загруженные объекты в памяти, отслеживает изменения и одной командой сохраняет всё в БД.

```python-executable
from sqlalchemy.orm import Session

with Session(engine) as session:
    # ... здесь работаем с объектами
    session.commit()

print("Сессия закрыта")
# Вывод: Сессия закрыта
```

`with Session(...)` сам закроет сессию по выходу. `commit()` сохраняет накопленные изменения в БД.

## CRUD через объекты

Дальше предполагаем, что `engine` и класс `Task` уже определены.

### CREATE: создание

```python-executable
from sqlalchemy.orm import Session

with Session(engine) as session:
    task1 = Task(title="Изучить ORM")
    task2 = Task(title="Написать код")
    session.add_all([task1, task2])
    session.commit()
    print(task1)
    print(task2)
# Вывод:
# Task(id=1, title='Изучить ORM', completed=False)
# Task(id=2, title='Написать код', completed=False)
```

Заметьте: `task1.id` после `commit()` уже заполнен. БД назначила его автоматически.

### READ: чтение

В современном SQLAlchemy 2.0 запросы пишутся через `select()` + `session.execute()`. Старый `session.query(...)` ещё работает, но считается устаревшим.

```python-executable
from sqlalchemy import select
from sqlalchemy.orm import Session

with Session(engine) as session:
    # Все строки
    tasks = session.execute(select(Task)).scalars().all()
    for task in tasks:
        print(task)
# Вывод:
# Task(id=1, title='Изучить ORM', completed=False)
# Task(id=2, title='Написать код', completed=False)
```

`.scalars()` нужен потому что `select(Task)` возвращает строки-кортежи (даже если в каждом кортеже один элемент). `.scalars()` распаковывает их в объекты `Task`.

Получить одну запись по первичному ключу проще через `session.get`:

```python-executable
from sqlalchemy.orm import Session

with Session(engine) as session:
    task = session.get(Task, 1)
    print(task)
# Вывод: Task(id=1, title='Изучить ORM', completed=False)
```

С фильтром:

```python-executable
from sqlalchemy import select
from sqlalchemy.orm import Session

with Session(engine) as session:
    stmt = select(Task).where(Task.completed == False)
    pending = session.execute(stmt).scalars().all()
    for task in pending:
        print(task)
# Вывод:
# Task(id=1, title='Изучить ORM', completed=False)
# Task(id=2, title='Написать код', completed=False)
```

### UPDATE: изменение

Самая удобная часть ORM: меняем атрибут объекта, и Session сам понимает что нужно обновить:

```python-executable
from sqlalchemy.orm import Session

with Session(engine) as session:
    task = session.get(Task, 1)
    task.completed = True
    session.commit()
    print(task)
# Вывод: Task(id=1, title='Изучить ORM', completed=True)
```

Никаких явных `UPDATE ... SET ... WHERE ...`. Session отслеживает изменённые атрибуты и при `commit()` отправляет нужный SQL.

### DELETE: удаление

```python-executable
from sqlalchemy.orm import Session

with Session(engine) as session:
    task = session.get(Task, 2)
    session.delete(task)
    session.commit()
    print("Задача удалена")
# Вывод: Задача удалена
```

## Связи между таблицами

В реальных схемах таблицы связаны: у пользователя есть задачи, у поста комментарии. ORM описывает связи через `relationship`, и обращение к связанным записям выглядит как обращение к обычному атрибуту:

```python-executable
from sqlalchemy import ForeignKey
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship, Session
from sqlalchemy import create_engine
from typing import List

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    tasks: Mapped[List["UserTask"]] = relationship(back_populates="user")

class UserTask(Base):
    __tablename__ = 'user_tasks'
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    user: Mapped["User"] = relationship(back_populates="tasks")

engine = create_engine('sqlite:///orm_users.db')
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
# Вывод:
# Анна
#   Изучить Python
#   Написать код
```

`user.tasks` за кулисами выполняет SQL-запрос `SELECT ... FROM user_tasks WHERE user_id = ?`, но в коде это выглядит как обычный доступ к атрибуту. Это и есть главный комфорт ORM: реляционная связь читается как «у пользователя есть задачи».

## Сравнение трёх подходов

| Аспект                | sqlite3                | SQLAlchemy Core        | SQLAlchemy ORM       |
| --------------------- | ---------------------- | ---------------------- | -------------------- |
| Запрос                | SQL-строка             | Python-выражение       | Python-объект        |
| Защита от инъекций    | через `?` вручную      | автоматически          | автоматически        |
| Переносимость между БД | нет                    | есть                   | есть                 |
| Связи                 | JOIN вручную           | JOIN-выражения         | `user.tasks`         |
| UPDATE                | `UPDATE ... SET ...`   | `update().values(...)` | `obj.field = ...`    |
| Контроль над SQL      | максимальный           | высокий                | средний              |

Хорошее правило: ORM для типичной бизнес-логики, Core для сложных запросов где нужен контроль, raw SQL только когда первые два не справляются.

## Что дальше?

ORM это инструмент, который оптимизирует **типичные** случаи работы с БД. Если в проекте 95% запросов это «получи объект, поменяй поле, сохрани», ORM экономит кучу времени. Когда упираетесь в сложный запрос или performance-критичный путь, спускайтесь в Core или пишите SQL напрямую. Эти три уровня дополняют друг друга.

---

**Главное преимущество ORM перед Core?**

