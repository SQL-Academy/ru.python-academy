---
meta:
    title: "Классы и объекты"
    description: "Собираем первый класс по шагам: __init__, self и методы. Затем глубже: как работает self, изменяемость объектов и динамические атрибуты."
---

# Классы и объекты

В прошлой главе мы смотрели на класс издалека: форма, шаблон, объекты. Теперь соберём такой класс сами, строчка за строчкой.

## Собираем класс по шагам

Самый короткий класс в Python выглядит так:

```python
class Person:
    pass

person = Person()
print(type(person))
<output>
<class '__main__.Person'>
</output>
```

`class Person:` объявляет класс, а `Person()` создаёт по нему объект. Правда, пока пустой: никаких данных внутри нет.

Данные закладывают в `__init__` — специальном методе, который Python сам вызывает при каждом `Person(...)`. Аргументы вызова попадают в его параметры:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

person = Person("Анна", 25)
print(person.name)
<output>
Анна
</output>
print(person.age)
<output>
25
</output>
```

Строка `self.name = name` читается так: «возьми параметр `name` и сохрани его в объекте под именем `name`». Слева — атрибут объекта, справа — параметр метода.

Это и есть заполнение шаблона со схемы из прошлой главы: вызов `Person("Анна", 25)` заполнил пропуски.

Данные есть, теперь добавим действие. Метод объявляют как обычную функцию, только внутри класса, и первым параметром он всегда принимает `self`:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

person = Person("Анна", 25)
print(person.greet())
<output>
Привет, меня зовут Анна, мне 25 лет.
</output>
```

Заметили странность? В `def greet(self)` параметр объявлен, а в `person.greet()` мы ничего не передаём. Откуда же `self` берётся?

## Как работает self

Метод `greet` записан в классе один раз, а объектов может быть сколько угодно. Значит, при вызове методу нужно как-то узнать, **чей** `name` печатать. Эту работу делает точка: вызов `person.greet()` Python выполняет как `Person.greet(person)` — объект слева от точки сам становится первым аргументом. Он и приходит в параметр `self`.

`person.greet()` — Python выполняет как → `Person.greet(person)`; объект слева от точки становится первым аргументом и приходит в `self`.

Проверим, что это буквально так:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

person = Person("Анна", 25)

# Эти два вызова делают одно и то же:
print(person.greet())
<output>
Привет, меня зовут Анна, мне 25 лет.
</output>
print(Person.greet(person))
<output>
Привет, меня зовут Анна, мне 25 лет.
</output>
```

Раз `self` — обычный первый параметр, имя ему можно дать любое: `def greet(this):` тоже сработает. Но всё Python-сообщество пишет `self`, и линтеры ругаются на другие имена.

Через `self` методы могут вызывать другие методы того же объекта:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def is_adult(self):
        return self.age >= 18

    def describe(self):
        status = "взрослый" if self.is_adult() else "несовершеннолетний"
        return f"{self.name}: {status}"

person = Person("Анна", 25)
print(person.describe())
<output>
Анна: взрослый
</output>
```

## Объекты в Python изменяемы

До сих пор методы только читали данные: `is_adult` смотрел на `self.age` и ничего не менял. Но состояние объекта можно и менять: методом, как `add_grade` ниже, или присваиванием атрибуту напрямую.

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.grades = []

    def add_grade(self, grade):
        self.grades.append(grade)
        return f"Добавлена оценка: {grade}"

    def average_grade(self):
        if not self.grades:
            return "Нет оценок"
        return round(sum(self.grades) / len(self.grades), 1)

student = Student("Мария")
print(f"Средний балл: {student.average_grade()}")
<output>
Средний балл: Нет оценок
</output>

print(student.add_grade(5))
<output>
Добавлена оценка: 5
</output>
print(student.add_grade(4))
<output>
Добавлена оценка: 4
</output>
print(student.add_grade(5))
<output>
Добавлена оценка: 5
</output>

print(f"Средний балл: {student.average_grade()}")
<output>
Средний балл: 4.7
</output>
```

Метод `add_grade` меняет `self.grades` — список, хранящийся в объекте. Изменения происходят **на месте**: следующий вызов `student.average_grade()` видит обновлённое состояние. Это не «верни новый список», а «измени существующий».

## Динамические атрибуты

Менять существующие атрибуты — обычное дело. Python разрешает больше: объекту можно добавить **любой** новый атрибут в любой момент, даже не объявленный в `__init__`. Проверим на упрощённом `Student`, у которого есть только имя:

```python
class Student:
    def __init__(self, name):
        self.name = name

student = Student("Мария")
student.age = 19            # добавили новый атрибут на лету
student.favorite_color = "синий"

print(student.age)
<output>
19
</output>
print(student.favorite_color)
<output>
синий
</output>
```

Технически это работает, но в реальном коде так почти не пишут, и вот почему:

- **Состояние объекта становится непредсказуемым.** Глядя на класс `Student`, нельзя понять, какие атрибуты в действительности есть у объекта.
- **IDE и линтеры не помогут** с автодополнением: они знают только то, что объявлено в `__init__`.
- **При опечатке создастся новый атрибут** вместо понятной ошибки. Если вы напишете `student.aeg = 19` вместо `student.age = 19`, Python молча создаст новое поле `aeg`, и баг сложно найти.

Поэтому все атрибуты объекта объявляйте в `__init__` — даже со значением `None`, если они появятся позже:

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.age = None      # будет заполнено позже
        self.grades = []
```

Так класс честно описывает, какие поля есть у объекта, и опечатки сразу превращаются в `AttributeError`.

## Проверка понимания

**Что произойдёт при выполнении `Person.greet(person)`, если `greet` определён с `self` как первый параметр?**

1. Ошибка: нельзя вызывать метод через имя класса. — Вызывать метод через имя класса можно. Просто нужно передать объект первым аргументом вручную.

2. **Правильный ответ:** То же самое, что и person.greet() — person станет self. — Запись person.greet() Python выполняет как Person.greet(person): объект слева от точки сам передаётся первым аргументом и приходит в self.

3. Метод вызовется, но self будет None. — self получит ровно то, что передано первым аргументом — в данном случае объект person.

4. Создастся новый объект Person. — Объект создаётся только через Person() (вызов конструктора), а не через вызов метода.

В следующей статье разберём атрибуты подробнее: чем атрибуты экземпляра отличаются от атрибутов класса и как не наступить на классическую ловушку с изменяемыми значениями по умолчанию в `__init__`.
