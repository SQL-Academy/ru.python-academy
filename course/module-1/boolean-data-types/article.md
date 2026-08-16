---
meta:
    title: "Логические данные в Python"
    description: "Подробное описание логического типа данных bool в Python: создание логических значений, операции, преобразование типов и практические примеры использования."
---

# Логические данные в Python

Когда программа решает, что делать дальше, выводить ли сообщение, пропустить ли пользователя, повторить ли запрос, она работает с логическими значениями. Тип `bool` в Python хранит ровно два значения: `True` и `False`, и из них собираются все условия и проверки.

## Что такое логические данные?

В Python для значений «истина» и «ложь» существует специальный тип данных `bool`, который может принимать только два значения:

- `True` (истина)
- `False` (ложь)

Спросим у Python тип, и он подтвердит, что перед ним `bool`:

```python
is_raining = True
print(type(is_raining))
<output>
<class 'bool'>
</output>
```

> **Важно**: `True` и `False` всегда пишутся с большой буквы. Если написать `true` или `false`, Python не поймёт и выдаст ошибку.

## Логические операторы

Часто нужно объединять несколько условий. Например: «поеду на пляж, если будет солнечно **И** тепло» или «куплю этот телефон, если он красивый **ИЛИ** недорогой». В Python для этого есть три оператора: `and`, `or`, `not`.

### Оператор and (логическое И)

Возвращает `True` только если **оба** значения истинны:

```python
sunny = True
warm = True

# Поеду на пляж, если солнечно И тепло
going_to_beach = sunny and warm
print(f"Солнечно: {sunny}, Тепло: {warm}")
<output>
Солнечно: True, Тепло: True
</output>
print(f"Идем на пляж? {going_to_beach}")
<output>
Идем на пляж? True
</output>

# Что если погода изменится?
warm = False  # Стало холодно
going_to_beach = sunny and warm
print(f"Солнечно: {sunny}, Тепло: {warm}")
<output>
Солнечно: True, Тепло: False
</output>
print(f"Идем на пляж? {going_to_beach}")
<output>
Идем на пляж? False
</output>
```

### Оператор or (логическое ИЛИ)

Возвращает `True`, если **хотя бы одно** значение истинно:

```python
phone_is_beautiful = True
phone_is_cheap = False

# Куплю телефон, если он красивый ИЛИ недорогой
will_buy_phone = phone_is_beautiful or phone_is_cheap
print(f"Телефон красивый: {phone_is_beautiful}, Телефон дешевый: {phone_is_cheap}")
<output>
Телефон красивый: True, Телефон дешевый: False
</output>
print(f"Купим телефон? {will_buy_phone}")
<output>
Купим телефон? True
</output>
```

### Оператор not (логическое НЕ)

Инвертирует значение: `True` становится `False`, и наоборот:

```python
have_homework = True
print(f"У меня есть домашнее задание: {have_homework}")
<output>
У меня есть домашнее задание: True
</output>
print(f"У меня НЕТ домашнего задания: {not have_homework}")
<output>
У меня НЕТ домашнего задания: False
</output>
```

## Сравнение значений

Логические значения часто появляются в результате сравнения:

```python
# Сравнение чисел
my_age = 25
friend_age = 30

print(f"Мой возраст: {my_age}, возраст друга: {friend_age}")
<output>
Мой возраст: 25, возраст друга: 30
</output>
print(f"Наш возраст одинаковый? {my_age == friend_age}")
<output>
Наш возраст одинаковый? False
</output>
print(f"Наш возраст разный? {my_age != friend_age}")
<output>
Наш возраст разный? True
</output>
print(f"Я младше? {my_age < friend_age}")
<output>
Я младше? True
</output>

# Сравнение строк (по алфавиту)
print(f"'apple' < 'banana': {'apple' < 'banana'}")
<output>
'apple' < 'banana': True
</output>
```

### Оператор is: сравнение по идентичности

Помимо `==` в Python есть оператор `is`. Они кажутся похожими, но проверяют разные вещи:

- `==` сравнивает **значения** (что у нас внутри)
- `is` сравнивает **идентичность** (это один и тот же объект в памяти или нет)

```python
my_scores = [90, 85, 95]
friend_scores = [90, 85, 95]  # Такой же список, но другой объект
same_list = my_scores         # Тот же самый объект

print(f"Содержимое одинаковое? {my_scores == friend_scores}")
<output>
Содержимое одинаковое? True
</output>
print(f"Это один и тот же объект? {my_scores is friend_scores}")
<output>
Это один и тот же объект? False
</output>
print(f"same_list это тот же объект что и my_scores? {my_scores is same_list}")
<output>
same_list это тот же объект что и my_scores? True
</output>
```

**Правило:** в подавляющем большинстве случаев нужен `==`. Оператор `is` уместен только для проверок на специальные синглтоны: `is None`, `is True`, `is False`. Применять `is` к числам, строкам, спискам почти всегда ошибка.

## Приоритет операторов: что вычисляется первым?

Операторы выполняются в следующем порядке (от высшего к низшему):

1. `not` (самый высокий приоритет)
2. `and`
3. `or` (самый низкий приоритет)

```python
# Пример с приоритетами
has_ticket = True
has_passport = False
has_visa = True

# Можно ли поехать за границу?
# Нужен билет И (паспорт ИЛИ виза)
can_travel = has_ticket and (has_passport or has_visa)

print(f"has_ticket and (has_passport or has_visa) = {can_travel}")
<output>
has_ticket and (has_passport or has_visa) = True
</output>

# Разберем вычисление пошагово:
step1 = has_passport or has_visa  # Сначала вычисляется выражение в скобках
print(f"Шаг 1: has_passport or has_visa = {step1}")
<output>
Шаг 1: has_passport or has_visa = True
</output>

step2 = has_ticket and step1  # Затем применяется оператор and
print(f"Шаг 2: has_ticket and (результат шага 1) = {step2}")
<output>
Шаг 2: has_ticket and (результат шага 1) = True
</output>
```

> **Совет**: если сомневаетесь в порядке выполнения, используйте скобки. Они делают код более понятным и точно контролируют порядок вычислений.

## Преобразование в логический тип: что считается истиной?

### Функция bool

Python может преобразовать любое значение в логический тип:

```python
print(bool(100))
<output>
True
</output>
print(bool(0))
<output>
False
</output>
print(bool("Hello"))
<output>
True
</output>
print(bool(""))
<output>
False
</output>
```

### Что считается истинным и ложным?

В Python большинство значений считаются истинными (`True`).

Ложными (`False`) считаются только:

- `False` (логическое «нет»)
- `None` (отсутствие значения)
- Нули: `0`, `0.0`, `0j`
- Пустые контейнеры: `""`, `()`, `[]`, `{}`

```python
money = 0
if money:
    print("У меня есть деньги")
else:
    print("Мой кошелек пуст")
<output>
Мой кошелек пуст
</output>

name = "Алекс"
if name:
    print(f"Привет, {name}")
else:
    print("Привет, незнакомец")
<output>
Привет, Алекс
</output>
```

## Проверка понимания

Давайте проверим, как вы усвоили материал:

**Что вернет следующее выражение?**

```python
result = (False or True) and not (False and True or True)
```

1. True — Разберём по шагам: (False or True) = True; внутри второй скобки (False and True) = False, потом False or True = True; not True = False; True and False = False. Правильный ответ — False.

2. **Правильный ответ:** False — Разберём по шагам: (False or True) = True; внутри второй скобки (False and True) = False, потом False or True = True; not True = False; True and False = False.

3. None — None здесь взяться неоткуда: в выражении участвуют только True и False, поэтому результатом будет одно из них. Стоит запомнить на будущее: and и or возвращают не «истину вообще», а одно из тех значений, к которым применены, — окажись среди них None, он мог бы и вернуться.

4. Ошибка синтаксиса — Это корректное логическое выражение в Python.
