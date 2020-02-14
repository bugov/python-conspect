# Регулярная пара

Регулярные выражения уже рассматривались в курсе ОС. По сути - это средство
примитивнго гибкого поиска и начальной валидации данных.

## re

– модуль python для работы с regex.
- задачки на регулярные выражения, студенты у доски
- matched groups
- re.(DOTALL, IGNORECASE, ASCII, MULTILINE)
- split, search, match, findall

```python
>>> import re
>>> re.split(r'\W+', 'Words, words, words.')
['Words', 'words', 'words', '']


# Этакий аналог find/index
>>> match = re.search(r'\w+', '123 dfsdf \n qwe')
>>> match.group()
'123'


# Поиск полного совпадения - аналог ^$
# + пример именованного захвата (named capture)
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Malcolm Reynolds")
>>> m.group('first_name')
'Malcolm'
>>> m.group('last_name')
'Reynolds'


# У всех этих функций есть параметр flags
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Вася Пупкин")
>>> m.group('first_name')
'Вася'
>>> m = re.match(r"(?P<first_name>\w+) (?P<last_name>\w+)", "Вася Пупкин", flags=re.ASCII)
>>> m.group('first_name')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'NoneType' object has no attribute 'group'


>>> re.sub(r'-+', lambda x: '+' * len(x.group(0)), 'asd-vsdv----qwe')
'asd+vsdv++++qwe'
```

```python
import re
from collections import Counter
from urllib.request import urlopen

with urlopen('ftp://shannon.usu.edu.ru/python/hw2/home.html') as fp:
    html = str(fp.read(), encoding='cp1251')

match = re.findall('<a href=.*?>(\w+)\s+(\w+)</a>', html)
counts = Counter([x for _, x in match])
```

- Задачка с предыдущей пары, но не str.find, а re.match
- ДЗ на вход argv слово, wiki/слово - какой путь до слова "философия",
  не ходим по страницам дважды, не дольше 500 секунд.

# Разбор ДЗ



# функции

- *args,

```python
def printf(fmt, *args):
  print(fmt % args)
```
- **kwargs

- docstrings

```python
def hello(name):
  """Hello function
  name - human's name
  returns hello string
  """
  return f"Hello {name}!"
```

- annotations
```Python
def hello(name: "human's name"):
  return f"Hello {name}"
```

- typing

```Python
from typing import Optional, List

def smth(a: int = 0, b: List[int]) -> (int, int):
    pass
```

Захват переменных lambda’ой, проблемы и решение

Проблемный код:
```Python
fs = []
for i in range(10):
  fs.append(lambda x: x**i)
for f in fs:
  print(f(2))
```


```Python
fs = []
for i in range(10):
  fs.append(lambda x, i=i: x**i)
for f in fs:
  print(f(2))
```
