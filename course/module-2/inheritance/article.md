# Наследование в Python

В прошлых уроках мы написали класс `Person` — имя, возраст, метод `greet()`. Теперь нужен класс `Student`. У студента есть имя и возраст (то же, что у человека), есть `greet()` (студент тоже умеет здороваться), но дополнительно есть школа и оценки, а здороваться он умеет «по-своему» — с упоминанием школы.

Можно скопировать весь код `Person` в `Student` и дописать новое. Но если потом мы что-то поправим в `Person`, в копии это не обновится. Дублирование кода — это всегда мина с задержкой.

Наследование позволяет сказать: «`Student` — это `Person`, плюс ещё кое-что». Не копировать, а **продолжить** существующий класс.

## Создание дочернего класса

Чтобы один класс наследовался от другого, имя родителя пишется в скобках после имени дочернего класса:

```python-executable
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

class Student(Person):   # Student наследуется от Person
    pass                  # пока ничего нового не добавляем

# Создаём студента
student = Student("Анна", 20)

# greet() и атрибуты унаследованы от Person
print(student.name)
# Вывод: Анна
print(student.greet())
# Вывод: Привет, меня зовут Анна, мне 20 лет.
```

Мы ни строчки не написали внутри `Student`, но он уже работает — потому что получил `__init__` и `greet()` от `Person`. Это и есть базовое наследование.

## Добавление новых атрибутов и super()

Теперь добавим студенту школу и оценки. Нужно расширить `__init__`: принять и старые параметры (`name`, `age`), и новые (`school`, `grades`). Чтобы не дублировать установку `self.name` и `self.age`, вызовем родительский `__init__` через `super()`:

```python-executable
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

class Student(Person):
    def __init__(self, name, age, school):
        super().__init__(name, age)   # пусть Person сам установит name и age
        self.school = school
        self.grades = []

    def add_grade(self, grade):
        self.grades.append(grade)

student = Student("Анна", 20, "МГУ")
student.add_grade(5)
student.add_grade(4)

print(student.greet())            # унаследованный метод
# Вывод: Привет, меня зовут Анна, мне 20 лет.
print(student.school, student.grades)
# Вывод: МГУ [5, 4]
```

`super()` это ссылка на «родителя текущего класса». `super().__init__(name, age)` означает «вызови `__init__` от `Person`, передай ему `name` и `age`». Так мы переиспользуем логику родителя вместо того, чтобы копировать её.

## Переопределение методов

Дочерний класс может **переопределить** метод родителя — задать своё поведение под тем же именем. Если в `Student` определён свой `greet()`, Python будет вызывать его, а не родительский:

```python-executable
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}."

class Student(Person):
    def __init__(self, name, age, school):
        super().__init__(name, age)
        self.school = school

    def greet(self):
        # Используем родительский greet() и дописываем своё
        return f"{super().greet()} Я учусь в {self.school}."

person = Person("Иван", 30)
student = Student("Анна", 20, "МГУ")

print(person.greet())
# Вывод: Привет, меня зовут Иван.
print(student.greet())
# Вывод: Привет, меня зовут Анна. Я учусь в МГУ.
```

Внутри переопределённого метода можно вызвать `super().greet()`, чтобы не повторять логику родителя, а только расширить её.

## Многоуровневые иерархии

Наследоваться можно цепочкой: `Student` от `Person`, а `GraduateStudent` от `Student`. Получится цепочка из трёх классов, и потомок будет иметь доступ ко всему, что есть выше:

```python-executable
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Привет, я {self.name}."

class Student(Person):
    def __init__(self, name, school):
        super().__init__(name)
        self.school = school

class GraduateStudent(Student):
    def __init__(self, name, school, advisor):
        super().__init__(name, school)
        self.advisor = advisor

grad = GraduateStudent("Анна", "МГУ", "Петров И.И.")

# Метод из Person, атрибуты со всех уровней
print(grad.greet())
# Вывод: Привет, я Анна.
print(grad.school, "/", grad.advisor)
# Вывод: МГУ / Петров И.И.
```

Когда Python ищет метод или атрибут, он идёт по цепочке: сначала в текущем классе, потом в родителе, потом в родителе родителя — пока не найдёт.

## Проверка типов: isinstance и issubclass

`isinstance(obj, Class)` проверяет, является ли объект экземпляром класса (или любого его наследника). `issubclass(A, B)` проверяет, является ли класс `A` наследником `B`:

```python-executable
class Person:
    pass

class Student(Person):
    pass

student = Student()

# Student - это разновидность Person
print(isinstance(student, Student))
# Вывод: True
print(isinstance(student, Person))
# Вывод: True

# Отношения между классами
print(issubclass(Student, Person))
# Вывод: True
print(issubclass(Person, Student))
# Вывод: False
```

Главное здесь: студент **является** человеком (`isinstance(student, Person)` это `True`), но человек не обязательно студент. Наследование задаёт отношение «is-a» в одну сторону.

## А что насчёт множественного наследования?

Python разрешает классу иметь несколько родителей: `class Duck(Flying, Swimming):`. Это работает, но порядок поиска методов в таких иерархиях быстро становится неочевидным (есть отдельный алгоритм — Method Resolution Order, MRO). На практике для большинства задач хватает одного родителя, и так код легче читать. С множественным наследованием можно разобраться позже, в продвинутом курсе.

## Что дальше?

В следующем уроке возьмём второй принцип ООП — инкапсуляцию: как прятать внутренности класса за интерфейсом и почему это делает код устойчивее к изменениям.

---

**Что произойдёт при вызове метода из дочернего класса, который не переопределяет этот метод родительского класса?**

