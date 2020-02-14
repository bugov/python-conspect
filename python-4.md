# Немного покодим

Небольшой парсинг логов - надо ли?

# Классная пара

## Пример создания собственного класса

```Python
class InvalidMatrixException(ValueError):
    pass


class Matrix:
    def __init__(self, array):
        self._array = array
        self._horizontal = len(array)
        self._vertical = len(array[0])

        for line in array:
            if len(line) != self._vertical:
                raise InvalidMatrixException

    def __repr__(self):
        return f"<Matrix {self._horizontal}x{self._vertical}>"

    def __str__(self):
        result = []
        for self._array:
            ...

    def __add__(self, other):

    ...
```

## Переопределение операций

В рамках примера выше можно показать список всяких `__eq__`, `__mul__`.

## Простейшее использование юниттестов

```Python
import unittest

class TestACase(unittest.TestCase):
    def setUp(self):
        self.widget = Widget('The widget')

    def tearDown(self):
        self.widget.dispose()
```

## private?

```Python

In [4]: class A:
   ...:     def __b(self):
   ...:         print("Hello")
   ...:         

In [5]: a = A()

In [6]: a.__b()
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-6-9ce524e2e55b> in <module>()
----> 1 a.__b()

AttributeError: 'A' object has no attribute '__b'

In [7]: a._A__b()
Hello

```

Задача на паре:
Написать свой аналог complex или Matrix с поддержкой сложения, вычитания, умножения. Сделать на это несколько тестов. Для тех, кто успел раньше: с поддержкой деления, возведения в степень.

# Итераторы / Генераторы

Если нужно получать значения, которые получаем поэтапно.

```python
In [25]: a = [x * x for x in range(10)]

In [26]: a
Out[26]: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

In [27]: a = (x * x for x in range(10))

In [28]: a
Out[28]: <generator object <genexpr> at 0x1093e7258>

In [29]: for x in a: print(x)
0
1
4
9
16
25
36
49
64
81

In [30]: for x in a: print(x)

In [31]: a = (x * x for x in range(10))

In [32]: next(a)
Out[32]: 0

In [33]: next(a)
Out[33]: 1

In [34]: next(a)
Out[34]: 4
```

Свой итерабельный объект:

```Python
In [49]: class A:
    ...:     def __init__(self, start=0):
    ...:         self.start = start
    ...:     
    ...:     def __iter__(self):
    ...:         return self
    ...:
    ...:     def __next__(self):
    ...:         self.start += 1
    ...:         
    ...:         if self.start > 100:
    ...:             raise StopIteration()
    ...:
    ...:         return self.start
    ...:     

In [50]: a = A()

In [51]: print([x for x in a])
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100]
```

Что внутри `for in`:

```python
In [52]: a = [1, 2, 3, 4]

In [53]: i = iter(a)

In [54]: next(i)
Out[54]: 1

In [55]: next(i)
Out[55]: 2

In [56]: next(i)
Out[56]: 3

In [57]: next(i)
Out[57]: 4

In [58]: next(i)
---------------------------------------------------------------------------
StopIteration                             Traceback (most recent call last)
<ipython-input-58-a883b34d6d8a> in <module>()
----> 1 next(i)

StopIteration:
```
