# Классы и объекты

В предыдущем уроке мы разобрали базу: класс это шаблон, объект — заполненный экземпляр, методы получают `self` первым параметром. В этой статье копнём глубже три темы: как **на самом деле** работает `self`, что объекты в Python **изменяемы**, и почему атрибуты можно добавлять **на лету** (и почему этим лучше не злоупотреблять).

## Как работает self

Когда вы пишете `person.greet()`, Python внутри превращает это в `Person.greet(person)`. То есть объект слева от точки автоматически становится первым аргументом метода — тем самым `self`.

Через `self` метод видит собственные данные:

```python-executable
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

person = Person("Анна", 25)

# Эти два вызова делают одно и то же:
print(person.greet())
# Вывод: Привет, меня зовут Анна, мне 25 лет.
print(Person.greet(person))
# Вывод: Привет, меня зовут Анна, мне 25 лет.
```

`self` это не keyword и не магия. Это просто имя первого параметра метода по соглашению. Технически можно назвать как угодно (`def greet(this):` тоже сработает), но Python-сообщество ожидает `self`, и линтеры будут ругаться на другие имена.

Через `self` методы могут вызывать другие методы того же объекта:

```python-executable
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
# Вывод: Анна: взрослый
```

## Объекты в Python изменяемы

После создания объекта его состояние можно менять: вызывать методы, которые модифицируют атрибуты, или присваивать атрибутам новые значения напрямую.

```python-executable
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
        return sum(self.grades) / len(self.grades)

student = Student("Мария")
print(f"Средний балл: {student.average_grade()}")
# Вывод: Средний балл: Нет оценок

print(student.add_grade(5))
# Вывод: Добавлена оценка: 5
print(student.add_grade(4))
# Вывод: Добавлена оценка: 4
print(student.add_grade(5))
# Вывод: Добавлена оценка: 5

print(f"Средний балл: {student.average_grade()}")
# Вывод: Средний балл: 4.666666666666667
```

Метод `add_grade` модифицирует `self.grades` — список, хранящийся в объекте. Изменения происходят **на месте**: следующий вызов `student.average_grade()` видит обновлённое состояние. Это не «верни новый список», а «измени существующий».

## Динамические атрибуты

В Python объекту можно добавить **любой** атрибут в любой момент, даже если он не объявлен в `__init__`:

```python-executable
class Student:
    def __init__(self, name):
        self.name = name

student = Student("Мария")
student.age = 19            # добавили новый атрибут на лету
student.favorite_color = "синий"

print(student.age)
# Вывод: 19
print(student.favorite_color)
# Вывод: синий
```

Технически это работает, но в реальном коде так почти не пишут. Несколько причин:

-   **Состояние объекта становится непредсказуемым.** Глядя на класс `Student`, нельзя понять, какие атрибуты в действительности есть у объекта.
-   **IDE и линтеры не помогут** с автодополнением: они знают только то, что объявлено в `__init__`.
-   **При опечатке создастся новый атрибут** вместо понятной ошибки. Если вы напишете `student.aeg = 19` вместо `student.age = 19`, Python молча создаст новое поле `aeg`, и баг сложно найти.

Правило простое: все атрибуты объекта объявляйте в `__init__`, даже со значением `None`, если они появятся позже:

```python
class Student:
    def __init__(self, name):
        self.name = name
        self.age = None      # будет заполнено позже
        self.grades = []
```

Так класс честно описывает, какие поля есть у объекта, и опечатки сразу превращаются в `AttributeError`.

## Что дальше?

В следующей статье разберём атрибуты подробнее: разницу между атрибутами экземпляра и класса, и классическую ловушку с mutable-defaults в `__init__`.

---

**Что произойдёт при выполнении `Person.greet(person)`, если `greet` определён с `self` как первый параметр?**

