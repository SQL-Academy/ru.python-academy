---
meta:
    title: "Использование библиотек в Python"
    description: "Введение в библиотеки Python: что такое библиотеки, как их импортировать и использовать, стандартная библиотека и сторонние модули."
---

# Использование библиотек в Python

Вам понадобилось значение π или квадратный корень из числа. Выводить формулу и проверять её самому — долго и легко ошибиться, а кто-то уже написал и отладил этот код за вас. Готовые наборы такого кода называются библиотеками: подключаете нужную одной строкой и пользуетесь.

## Что такое библиотеки?

> Библиотека (или модуль) в Python — это файл с кодом, содержащий функции, классы и переменные, которые вы можете использовать в своих программах.

## Импорт библиотек

Чтобы использовать библиотеку, нужно сначала импортировать её в свою программу. Python предлагает несколько способов импорта:

### Импорт всей библиотеки

Подключаем модуль целиком. Всё его содержимое остаётся за именем модуля, и обращаемся мы к нему через точку:

```python
import math

radius = 5
circle_area = math.pi * radius ** 2
print(f"Площадь круга радиусом {radius} равна {circle_area:.2f}")
<output>
Площадь круга радиусом 5 равна 78.54
</output>
```

### Импорт конкретных элементов

Забираем из модуля только нужное. Эти имена попадают прямо в вашу программу, и писать `math.` перед ними больше не надо:

```python
from math import sqrt, floor

x = 16
result = sqrt(x)
print(f"Квадратный корень из {x} равен {result}")
<output>
Квадратный корень из 16 равен 4.0
</output>

y = floor(3.7)
print(f"Округление 3.7 вниз: {y}")
<output>
Округление 3.7 вниз: 3
</output>
```

### Импорт с переименованием

Тот же импорт целиком, но под коротким именем. Пригодится, когда полное имя длинное или уже занято вашей переменной:

```python
import math as m

angle = 45
sin_value = m.sin(m.radians(angle))
print(f"Синус {angle} градусов равен {sin_value:.4f}")
<output>
Синус 45 градусов равен 0.7071
</output>
```

## Виды библиотек в Python

В Python существует три основных вида библиотек:

1. **Встроенные модули** — модули, которые уже включены в стандартную библиотеку Python и доступны сразу после установки Python.

2. **Сторонние библиотеки** — модули, созданные другими разработчиками, которые нужно установить дополнительно.

3. **Собственные модули** — модули, которые вы создаете сами для организации вашего кода.

### Примеры встроенных модулей

```python
import random

random_number = random.randint(1, 10)
print(f"Случайное число: {random_number}")
<output>
Случайное число: 7
</output>

import datetime

current_date = datetime.datetime.now()
print(f"Текущая дата и время: {current_date}")
<output>
Текущая дата и время: 2023-07-15 14:30:45.123456
</output>
```

## Поиск функций в документации

При работе с библиотеками важно знать, как найти информацию о доступных функциях. Python предоставляет несколько способов:

### Использование функции help()

`help()` печатает всё, что модуль рассказывает о себе: назначение, список функций, описание каждой. У `math` справка длинная, поэтому ниже показано только её начало, а многоточие в конце — знак, что она на этом не кончается.

```python
import math
help(math)
<output>
Help on module math:

NAME
    math

MODULE REFERENCE
    https://docs.python.org/3.12/library/math.html

    The following documentation is automatically generated from the Python
    source files.  It may be incomplete, incorrect or include features that
    are considered implementation detail and may vary between Python
    implementations.  When in doubt, consult the module reference at the
    location listed above.

DESCRIPTION
    This module provides access to the mathematical functions
    defined by the C standard.

FUNCTIONS
    acos(x, /)
        Return the arc cosine (measured in radians) of x.

        The result is between 0 and pi.
...
</output>
```

### Использование функции dir()

```python
import random
attributes = dir(random)

# Выведем только первые 10 элементов для краткости
print(attributes[:10])
<output>
['BPF', 'LOG4', 'NV_MAGICCONST', 'RECIP_BPF', 'Random', 'SG_MAGICCONST', 'SystemRandom', 'TWOPI', '_ONE', '_Sequence']
</output>
```

## Проверка понимания

**Какие из следующих способов импорта библиотек являются правильными в Python?**

1. **Правильный ответ:** import math — Это правильный способ импорта всей библиотеки. После этого функции используются через имя модуля: math.sqrt(16).

2. **Правильный ответ:** from math import sqrt — Это правильный способ импорта конкретной функции из модуля. После этого функцию можно использовать напрямую: sqrt(16).

3. import sqrt from math — Это неправильный синтаксис. Правильный способ: "from math import sqrt".

4. import math.sqrt — Этот синтаксис не работает в Python. Нельзя импортировать конкретную функцию через точку.

В следующем уроке посмотрим [встроенные библиотеки](https://python-academy.org/ru/guide/built-in-libraries) — те, что доступны сразу после установки Python.
