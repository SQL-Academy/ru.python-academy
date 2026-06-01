# Полиморфизм

Допустим, у нас есть собака, кошка и утка, и нужно у каждой вызвать `speak()`. Без полиморфизма получится примерно так:

```python
for animal in animals:
    if isinstance(animal, Dog):
        print(animal.bark())
    elif isinstance(animal, Cat):
        print(animal.meow())
    elif isinstance(animal, Duck):
        print(animal.quack())
```

И каждый раз, когда появляется новый тип животного, нужно дописать ещё одну `elif`-ветку. Код, который **знает** обо всех возможных типах, не способен расширяться без переписывания.

Полиморфизм — это идея «вызывай одно и то же, а класс пусть сам решает, как реагировать». Тогда тот же код превращается в:

```python
for animal in animals:
    print(animal.speak())
```

`Dog.speak()` возвращает «Гав!», `Cat.speak()` — «Мяу!», `Duck.speak()` — «Кря!». Один вызов, разное поведение в зависимости от объекта.

## Полиморфизм через наследование

Это самый частый случай: общий родительский класс задаёт «контракт» (какие методы должны быть у потомков), а каждый потомок реализует их по-своему.

```python-executable
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return f"{self.name} молчит."

class Dog(Animal):
    def speak(self):
        return f"{self.name} говорит: Гав!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} говорит: Мяу!"

class Duck(Animal):
    def speak(self):
        return f"{self.name} говорит: Кря!"

animals = [Dog("Бобик"), Cat("Мурка"), Duck("Кряк")]
for animal in animals:
    print(animal.speak())
# Вывод:
# Бобик говорит: Гав!
# Мурка говорит: Мяу!
# Кряк говорит: Кря!
```

Внутри цикла мы не задаём вопрос «а кто ты такой?» — каждое животное само знает, как ему говорить. Если добавится `class Cow(Animal):` со своим `speak()`, цикл не изменится ни на строчку.

## Абстрактные классы

Часто хочется быть уверенным, что **все потомки точно реализовали нужный метод**. Например, базовый `Shape` должен заставлять каждый конкретный класс (Rectangle, Circle, …) реализовать `area()`. Если кто-то забудет, лучше пусть это обнаружится сразу, а не во время работы программы.

Для этого в Python есть **абстрактные базовые классы** из модуля `abc`:

```python-executable
from abc import ABC, abstractmethod
import math

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

    def describe(self):
        # Использует area() и perimeter(), не зная, как они реализованы
        return f"Площадь: {self.area():.2f}, периметр: {self.perimeter():.2f}"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return math.pi * self.radius ** 2

    def perimeter(self):
        return 2 * math.pi * self.radius

shapes = [Rectangle(5, 3), Circle(4)]
for i, shape in enumerate(shapes, 1):
    print(f"Фигура {i}: {shape.describe()}")
# Вывод:
# Фигура 1: Площадь: 15.00, периметр: 16.00
# Фигура 2: Площадь: 50.27, периметр: 25.13
```

Что нам дал `ABC`:

-   Класс `Shape` нельзя инстанцировать (`Shape()` упадёт). Это и логично: «фигура» сама по себе бессмысленна, нужна конкретная.
-   Если бы `Circle` забыл определить `perimeter()`, Python отказался бы создавать объект `Circle()` с ошибкой ещё на этапе создания, а не позже при вызове несуществующего метода.

При этом метод `describe()` определён прямо в `Shape` и работает для любого потомка — он опирается на `area()` и `perimeter()`, не зная их реализации. Это и есть полиморфизм через абстрактный базовый класс.

## Утиная типизация

Python идёт ещё дальше: для полиморфизма **не обязательно** общее наследование. Если объект ведёт себя как нужно (имеет правильные методы), он сойдёт. Эту идею формулируют так: «если оно ходит как утка и крякает как утка, то это утка».

```python-executable
class Duck:
    def swim(self):
        return "Утка плывёт."

    def sound(self):
        return "Кря-кря!"

class Person:
    def swim(self):
        return "Человек плывёт."

    def sound(self):
        return "Привет!"

def describe(entity):
    print(entity.swim())
    print(entity.sound())

describe(Duck())
# Вывод:
# Утка плывёт.
# Кря-кря!
describe(Person())
# Вывод:
# Человек плывёт.
# Привет!
```

Функция `describe` не проверяет, утка перед ней или человек. Ей важно только одно: чтобы у объекта были методы `swim()` и `sound()`. Никакого общего родителя у `Duck` и `Person` нет, но полиморфизм работает.

Это очень питоновский подход: вместо «опиши тип» — «опиши поведение». На практике он часто экономит лишние слои абстракций.

## Что дальше?

Полиморфизм закрывает четыре принципа ООП. Главный практический эффект: код, который вызывает методы через общий интерфейс, не нужно переписывать, когда появляются новые классы. Расширяемость встроена в саму архитектуру.

---

**Что из перечисленного ниже является примером полиморфизма в Python?**

