# Наследование в Python

> Наследование — это механизм, который позволяет создать новый класс на основе существующего. Дочерний класс получает атрибуты и методы родительского класса, но может расширять и модифицировать эту функциональность.

Проще говоря, наследование представляет отношения типа "является" (is-a): собака **является** животным, легковой автомобиль **является** транспортным средством.

### Зачем нужно наследование?

-   🔄 **Повторное использование кода** — избавляет от дублирования
-   🌲 **Логические иерархии** — отражает естественные отношения между объектами
-   🧩 **Расширяемость** — легко добавлять новую функциональность
-   🔄 **Полиморфизм** — использовать объекты разных классов единообразно

## Простое наследование в Python

Создать класс-наследник в Python очень просто — нужно указать родительский класс в скобках:

```python
# Базовый класс
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "Звук животного"

# Дочерний класс
class Dog(Animal):  # Dog наследуется от Animal
    def speak(self):  # Переопределяем метод
        return f"{self.name} говорит: Гав!"

# Создаем экземпляры
animal = Animal("Существо")
dog = Dog("Рекс")

# Вызываем методы
print(animal.speak())
print(dog.speak())  # Вызывает переопределенный метод
print(dog.name)  # Атрибут унаследован от родительского класса
```

## Вызов методов родительского класса с super()

Часто требуется расширить функциональность родительского метода, а не полностью заменить. Для этого используется функция `super()`:

```python
class Vehicle:
    def __init__(self, brand):
        self.brand = brand

    def info(self):
        return f"Транспорт марки {self.brand}"

class Car(Vehicle):
    def __init__(self, brand, model):
        super().__init__(brand)  # Вызываем конструктор родителя
        self.model = model

    def info(self):
        # Расширяем родительский метод
        return f"{super().info()}, модель {self.model}"

# Создаем автомобиль
car = Car("Toyota", "Corolla")
print(car.info())
```

## Иерархии наследования

Наследование можно выстраивать в многоуровневые иерархии:

```python
class Animal:
    def eat(self):
        return "Животное ест"

class Mammal(Animal):
    def breathe(self):
        return "Дышит легкими"

class Dog(Mammal):
    def bark(self):
        return "Гав!"

# Создаем собаку
dog = Dog()
print(dog.eat())    # Метод из Animal
print(dog.breathe())  # Метод из Mammal
print(dog.bark())   # Метод из Dog
```

## Проверка наследования: isinstance() и issubclass()

Python предоставляет полезные функции для проверки отношений наследования:

```python
# Базовый класс - родитель в иерархии
class Vehicle:
    pass  # Пустой класс для демонстрации наследования

# Дочерние классы - наследуются от Vehicle
class Car(Vehicle):
    pass  # Car является разновидностью Vehicle

class Bicycle(Vehicle):
    pass  # Bicycle тоже является разновидностью Vehicle

# Создаем объекты
car = Car()  # Экземпляр класса Car
bicycle = Bicycle()  # Экземпляр класса Bicycle

# Проверяем тип объектов
# car является экземпляром класса Car?
print(isinstance(car, Car))
# car является экземпляром класса Vehicle (через наследование)?
print(isinstance(car, Vehicle))
# car является экземпляром класса Bicycle?
print(isinstance(car, Bicycle))

# Проверяем отношения между классами
print(issubclass(Car, Vehicle))  # Car является подклассом Vehicle?
print(issubclass(Vehicle, Car))  # Vehicle является подклассом Car?
```

## Абстрактные классы

Абстрактные классы служат как шаблоны для создания других классов. Они содержат абстрактные методы, которые должны быть реализованы в подклассах:

```python
from abc import ABC, abstractmethod

class Shape(ABC):  # Абстрактный класс
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

# Попытка создать экземпляр абстрактного класса
try:
    shape = Shape()
except TypeError as e:
    print(f"Ошибка: {e}")

# Создание экземпляра конкретного класса
rect = Rectangle(5, 3)
print(f"Площадь: {rect.area()}")
print(f"Периметр: {rect.perimeter()}")
```

## Множественное наследование

Python поддерживает множественное наследование, когда класс наследуется от нескольких родителей:

```python
class Flying:
    def fly(self):
        return "Я могу летать!"

class Swimming:
    def swim(self):
        return "Я могу плавать!"

# Множественное наследование
class Duck(Flying, Swimming):
    def sound(self):
        return "Кря!"

# Создаем утку
duck = Duck()
print(duck.fly())
print(duck.swim())
print(duck.sound())
```

### Порядок разрешения методов (MRO)

При множественном наследовании важно знать, в каком порядке Python ищет методы в классах. Этот порядок называется Method Resolution Order (MRO):

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

# Проверяем MRO класса D
print(D.__mro__)

# Создаем объект и вызываем метод
d = D()
print(d.method())  # Будет вызван метод из B, так как B стоит перед C
```

## Проверка понимания

**Что произойдет при вызове метода из дочернего класса, который не переопределяет этот метод родительского класса?**
