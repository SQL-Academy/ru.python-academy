# SQLAlchemy ORM: объектно-реляционное отображение в Python

Мы изучили SQLAlchemy Core — мощный способ работы с SQL через Python-код. Теперь познакомимся с **SQLAlchemy ORM** — еще более удобным подходом, где таблицы базы данных становятся Python-классами, а записи — объектами.

## Что такое ORM?

**ORM (Object-Relational Mapping)** — это технология, которая связывает объекты в коде с записями в базе данных. Вместо написания SQL-запросов, вы работаете с обычными Python-объектами.

### Основная идея

**Без ORM (SQLAlchemy Core):**

```python
# Создаем SQL-запрос для получения пользователя
select_query = select(users_table).where(users_table.c.id == 1)
result = connection.execute(select_query)
user_row = result.fetchone()
print(user_row.name)  # Работаем с Row объектом
```

**С ORM:**

```python
# Работаем с Python-объектом напрямую
user = session.get(User, 1)  # User - это Python класс
print(user.name)  # Обращаемся к атрибуту объекта
user.email = "new@email.com"  # Изменяем как обычный объект
session.commit()  # Сохраняем изменения
```

### Преимущества ORM

-   **Pythonic код** — работаете с объектами, а не со строками SQL
-   **Автоматическое отслеживание изменений** — ORM сам знает что нужно сохранить
-   **Простые связи между таблицами** — `user.posts` вместо JOIN запросов
-   **Валидация данных** — проверки на уровне Python-классов

## Почему сначала Core, потом ORM?

**Понимание основ** — Core показал как работают SQL-запросы под капотом  
**Контроль** — иногда нужен точный контроль над SQL  
**Отладка** — зная Core, легче понять что делает ORM  
**Выбор инструмента** — для каждой задачи подходящий уровень абстракции

## Установка и настройка

ORM входит в основной пакет SQLAlchemy:

```bash
pip install sqlalchemy
```

## Создание моделей

В ORM таблицы описываются как Python-классы. Начнем с простого примера:

### Шаг 1: Базовая настройка

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# Базовый класс для всех моделей
Base = declarative_base()

# Подключение к базе данных
engine = create_engine('sqlite:///orm_tasks.db')

print("✅ Базовая настройка готова!")

```

### Шаг 2: Создание модели

```python
from sqlalchemy import Column, Integer, String, Boolean

# Модель = Python-класс = таблица в БД
class Task(Base):
    __tablename__ = 'tasks'  # Имя таблицы

    # Поля таблицы как атрибуты класса
    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    completed = Column(Boolean, default=False)

    # Красивое отображение объекта
    def __repr__(self):
        status = "✅" if self.completed else "⏳"
        return f"<Task: {status} {self.title}>"

print("✅ Модель Task создана!")

```

### Шаг 3: Создание таблицы и сессии

```python
# Создаем таблицу в базе данных
Base.metadata.create_all(engine)

# Создаем сессию для работы с данными
Session = sessionmaker(bind=engine)
session = Session()

print("✅ Таблица создана, сессия готова!")

```

## CRUD операции через ORM

Теперь изучим основные операции с данными. Каждая операция в отдельном примере:

### CREATE: Создание объектов

```python
# Создаем объекты как обычные Python-экземпляры
task1 = Task(title="Изучить ORM")
task2 = Task(title="Написать код")

# Добавляем в сессию и сохраняем
session.add_all([task1, task2])
session.commit()

print("✅ CREATE: Созданы задачи:")
print(f"  {task1}")
print(f"  {task2}")

```

### READ: Чтение данных

```python
# Получить все задачи
all_tasks = session.query(Task).all()
print("📋 READ: Все задачи:")
for task in all_tasks:
    print(f"  {task}")

```

```python
# Получить конкретную задачу по ID
task = session.get(Task, 1)
print(f"🎯 Задача с ID=1: {task}")

```

### UPDATE: Обновление объектов

```python
# Просто меняем атрибут объекта!
task = session.get(Task, 1)
task.completed = True
task.title = "Изучить ORM ✨"

# Сохраняем изменения
session.commit()

print(f"🔄 UPDATE: {task}")

```

### DELETE: Удаление объектов

```python
# Находим и удаляем задачу
task_to_delete = session.get(Task, 2)
session.delete(task_to_delete)
session.commit()

print(f"🗑️ DELETE: Удалена задача")


# Проверяем что осталось
remaining_tasks = session.query(Task).all()
print(f"📋 Осталось задач: {len(remaining_tasks)}")
for task in remaining_tasks:
    print(f"  {task}")

```

### Главные преимущества ORM

-   **Простота** — работаете с объектами как обычно в Python
-   **Автоматизация** — ORM сам отслеживает изменения
-   **Безопасность** — защита от SQL-инъекций из коробки
-   **Читаемость** — код понятен без знания SQL

## Отношения между таблицами

ORM позволяет легко связывать таблицы. Покажем на простом примере:

```python
from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship

# Пользователь (один) → Задачи (много)
class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)

    # Связь с задачами
    tasks = relationship("UserTask")

# Задача принадлежит пользователю
class UserTask(Base):
    __tablename__ = 'user_tasks'

    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'))

# Создаем таблицы и данные
Base.metadata.create_all(engine)
session = Session()

user = User(name="Анна")
task1 = UserTask(title="Изучить Python", user_id=1)
task2 = UserTask(title="Написать код", user_id=1)

session.add_all([user, task1, task2])
session.commit()

# Магия ORM: получаем связанные объекты
print(f"👤 Пользователь: {user.name}")
print(f"📋 Задач: {len(user.tasks)}")
for task in user.tasks:
    print(f"  - {task.title}")

session.close()

```

**Главное преимущество:** `user.tasks` автоматически получает связанные записи без написания JOIN запросов!

## Сравнение подходов

| Аспект                 | Чистый SQL      | SQLAlchemy Core     | SQLAlchemy ORM      |
| ---------------------- | --------------- | ------------------- | ------------------- |
| **Синтаксис**          | SQL строки      | Python-функции      | Python-объекты      |
| **Безопасность**       | Ручная          | Автоматическая      | Автоматическая      |
| **Связи таблиц**       | JOIN запросы    | Сложные выражения   | `user.tasks`        |
| **Изменения**          | UPDATE SQL      | `update().values()` | `obj.field = value` |
| **Кривая обучения**    | Нужно знать SQL | Средняя             | Простая для начала  |
| **Производительность** | Максимальная    | Высокая             | Хорошая             |

## Что мы изучили?

Теперь вы знаете основы SQLAlchemy ORM:

**Концепция ORM** — объекты вместо SQL  
**Создание моделей** — классы как таблицы  
**CRUD через объекты** — интуитивные операции  
**Связи между таблицами** — простая работа с отношениями

**Главное преимущество ORM перед Core?**
