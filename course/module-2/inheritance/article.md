---
meta:
    title: "Наследование в Python"
    description: "Как один класс продолжает другой: дочерние классы, super(), переопределение методов и проверка типа через isinstance."
---

# Наследование в Python

В прошлых уроках мы написали класс `Person` — имя, возраст, метод `greet()`. Теперь нужен класс `Student`. У студента есть имя и возраст (то же, что у человека), есть `greet()` (студент тоже умеет здороваться), но дополнительно есть школа, а здороваться он умеет «по-своему» — с упоминанием школы.

Можно скопировать весь код `Person` в `Student` и дописать новое. Но если потом мы что-то поправим в `Person`, в копии это не обновится. Дублирование кода — это всегда мина с задержкой.

Наследование позволяет сказать: «`Student` — это `Person`, плюс ещё кое-что». Не копировать, а **продолжить** существующий класс.

## Создание дочернего класса

Чтобы один класс наследовался от другого, имя родителя пишется в скобках после имени дочернего класса:

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Привет, меня зовут {self.name}, мне {self.age} лет."

class Student(Person):   # Student наследуется от Person
    pass                  # пока ничего нового не добавляем

student = Student("Анна", 20)

# greet() и атрибуты унаследованы от Person
print(student.name)
<output>
Анна
</output>
print(student.greet())
<output>
Привет, меня зовут Анна, мне 20 лет.
</output>
```

Мы ни строчки не написали внутри `Student`, но он уже работает — потому что получил `__init__` и `greet()` от `Person`. Это и есть базовое наследование.

`class Student(Person)` — наследует: `name`, `age`, `greet()` — из Person.

school объявлен в Student, а name, age и greet() достались от Person

## Добавление новых атрибутов и super()

Теперь добавим студенту школу. Нужно расширить `__init__`: принять и старые параметры (`name`, `age`), и новый (`school`). Чтобы не дублировать установку `self.name` и `self.age`, вызовем родительский `__init__` через `super()`:

```python
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

student = Student("Анна", 20, "МГУ")

print(student.greet())            # унаследованный метод
<output>
Привет, меня зовут Анна, мне 20 лет.
</output>
print(student.school)
<output>
МГУ
</output>
```

`super()` — это обращение к родителю текущего класса. Вызов в примере читается так: «попроси `Person` выполнить его `__init__`, передав `name` и `age`». Так мы переиспользуем логику родителя вместо того, чтобы копировать её.

## Переопределение методов

Дочерний класс может **переопределить** метод родителя — задать своё поведение под тем же именем. Если в `Student` определён свой `greet()`, Python будет вызывать его, а не родительский:

```python
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
<output>
Привет, меня зовут Иван.
</output>
print(student.greet())
<output>
Привет, меня зовут Анна. Я учусь в МГУ.
</output>
```

Внутри переопределённого метода можно вызвать `super().greet()`, чтобы не повторять логику родителя, а только расширить её.

## Многоуровневые иерархии

Наследоваться можно цепочкой: `Student` от `Person`, а `GraduateStudent` от `Student`. Потомок получает доступ ко всему, что объявлено выше:

```python
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
<output>
Привет, я Анна.
</output>
print(grad.school, "/", grad.advisor)
<output>
МГУ / Петров И.И.
</output>
```

Когда Python ищет метод или атрибут, он идёт по цепочке: сначала в текущем классе, потом в родителе, потом в родителе родителя — пока не найдёт.

## Проверка типа: isinstance

`isinstance(obj, Class)` проверяет, является ли объект экземпляром класса или любого его наследника:

```python
class Person:
    pass

class Student(Person):
    pass

person = Person()
student = Student()

print(isinstance(student, Student))
<output>
True
</output>
print(isinstance(student, Person))
<output>
True
</output>
print(isinstance(person, Student))
<output>
False
</output>
```

Главное здесь: студент **является** человеком, поэтому `isinstance(student, Person)` даёт `True`. Обратное неверно: человек — не обязательно студент.

## Проверка понимания

**Что произойдёт при вызове метода из дочернего класса, который не переопределяет этот метод родительского класса?**

1. **Правильный ответ:** Будет вызван метод родительского класса — Если метод не переопределён в дочернем классе, Python ищет его выше по цепочке родителей и выполняет найденную версию.

2. Будет вызвано исключение AttributeError — Если метод существует у родителя, Python найдёт и выполнит его, даже если в дочернем классе метод явно не определён.

3. Ничего не произойдёт, метод просто не будет выполнен — Python пройдёт по иерархии классов и, если найдёт метод у родителя, выполнит его.

4. Метод вернёт None по умолчанию — Будет выполнена родительская версия метода, а не возвращено None.

В следующем уроке возьмём второй принцип ООП — инкапсуляцию: как прятать внутренности класса, открывая наружу только нужное, и почему это делает код устойчивее к изменениям.
