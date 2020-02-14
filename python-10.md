# TPE, PPE, subprocess

Thread pool executor, Process pool executor

```python
>>> subprocess.call(["ls", "-l"])
0
>>> subprocess.call("exit 1", shell=True)
1
>>> subprocess.check_call(["ls", "-l"])
0
>>> subprocess.check_call("exit 1", shell=True)
>>> subprocess.check_output(
     "ls non_existent_file; exit 0",
     stderr=subprocess.STDOUT,
     shell=True)
'ls: non_existent_file: No such file or directory\n'
```

Контекстный менеджер:

```python
with Popen(["ifconfig"], stdout=PIPE) as proc:
    log.write(proc.stdout.read())
```

# TPE, PPE

```python
import concurrent.futures
import urllib.request

URLS = [
    'http://www.foxnews.com/',
    'http://www.cnn.com/',
    'http://europe.wsj.com/',
    'http://www.bbc.co.uk/',
    'http://some-made-up-domain.com/',
]

# Retrieve a single page and report the URL and contents
def load_url(url, timeout=10):
    with urllib.request.urlopen(url, timeout=timeout) as conn:
        return conn.read()

def tpe():
    # We can use a with statement to ensure threads are cleaned up promptly
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        # Start the load operations and mark each future with its URL
        future_to_url = {executor.submit(load_url, url, 60): url for url in URLS}
        for future in concurrent.futures.as_completed(future_to_url):
            url = future_to_url[future]
            try:
                data = future.result()
            except Exception as exc:
                print('%r generated an exception: %s' % (url, exc))
            else:
                print('%r page is %d bytes' % (url, len(data)))

def ppe():
    with concurrent.futures.ProcessPoolExecutor(max_workers=5) as executor:
        for url, data in zip(URLS, executor.map(load_url, URLS)):
            print('%r page is %d bytes' % (url, len(data)))
```

## subprocess

# система import

## как работает import a.b.c (про последовательность)

```Bash
$ mkdir -p ./a/b/c
$ echo "print(__name__)" > ./a/__init__.py
$ echo "print(__name__)" > ./a/b/__init__.py
$ echo "print(__name__)" > ./a/b/c/__init__.py
$ python -c 'import a.b.c'
a
a.b
a.b.c
$ python -c 'from a.b import c'
a
a.b
a.b.c
```

Теперь без `__init__.py`

```Bash
$ mkdir -p ./d/e/f
$ echo "print(__name__)" > ./d/e/f/g.py
$ python -c 'from d.e.f import g'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ImportError: No module named d.e.f
```

## `__init__.py`, `__all__`, `*`

Так мы понимаем, что для модульной системы python нам нужно использовать
`__init__.py`. Если мы хотим сделать модуль `hello.world`, в котором будут лежать
функции, с которыми нам нужно работать, то требуется сначала создать
модуль `hello`. Это можно сделать 2-мя способами:

1. Создаём файл `hello.py`
2. Создаём файл `hello/__init__.py`

С точки зрения python - это будет один и тот же модуль. И стоит использовать
первый вариант во всех случаях, исключая тот вайриант, когда нам надо
сделать ещё и подмодули.

*Замечу, что файл __init__.py - это своего рода "магия" и пример возможности
нескольких одинаково хороших вариантов решения
одной и той же задачи. Учитывая философию Python
(существет только 1 правильный способ сделать это), __init__.py можно рассматривать
как пример плохого дизайна в Python.*

Итак, мы использовали 2ой способ, где `__init__.py` - просто пустой файл.

Вообще, писать код в `__init__.py` - не самый лучший вариант – там принято
лишь объявлять служебные константы, например, версию модуля `__version__ = "0.0.1"`
или же для простоты использования импортировать необходимые функции,
чтобы не искать их по подмодулям.

Пример из `django/forms/__init__.py`:

```
"""
Django validation and HTML form handling.
"""

from django.core.exceptions import ValidationError  # NOQA
from django.forms.boundfield import *  # NOQA
from django.forms.fields import *  # NOQA
from django.forms.forms import *  # NOQA
from django.forms.formsets import *  # NOQA
from django.forms.models import *  # NOQA
from django.forms.widgets import *  # NOQA
```

Также можно заметить довольно странный импорт `*`. Это тоже пример плохого
дизайна. По сути это импорт всех публичных имён модуля. Немного похоже
на перечисление содержимого директории в bash.

Пример:

```python
$ cat my_mod.py
__all__ = (
    'ZERO',
    'sum',
    'sub',
    'mul',
    'div',
)

ZERO = 0


def sum(a, b):
    return a + b


def sub(a, b):
    return a - b


def mul(a, b):
    return a * b


def div(a, b):
    if b == ZERO:
        raise ZeroDivisionError()

    return a / b


def some_fun():
    raise NotImplementedError()
```

В `__all__` мы перечисляем все нужные для импорта имена. Что ж, используем:

```
$ ipy
Python 3.6.4 (default, Mar  9 2018, 23:15:03)
Type 'copyright', 'credits' or 'license' for more information
IPython 6.2.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: from my_mod import *

In [2]: ZERO
Out[2]: 0

In [3]: sum(1, 2)
Out[3]: 3

In [4]: some_fun()
---------------------------------------------------------------------------
NameError                                 Traceback (most recent call last)
<ipython-input-4-c929fb12600e> in <module>()
----> 1 some_fun()

NameError: name 'some_fun' is not defined
```

Так почему же это плохой подход? Всё потому, что это неявное поведение:
мы не указываем, какие модули нам нужны. А явное лучше неявного.

## importlib

```
In [7]: import importlib

In [8]: mod = importlib.import_module('my_mod')

In [9]: mod.sum(1, 2)
Out[9]: 3

In [12]: my_mod = __import__('my_mod')

In [13]: my_mod.sum(1, 2)
Out[13]: 3
```

1. Поиск по имени модуля
2. Привязка к текущему скоупу

Можно посмотреть, какие модули доступны

```
sys.modules
```

## sys.meta_path

Пишем свою реализацию для поддержки .aaa файлов:

```Python
import os
import sys
from importlib.abc import MetaPathFinder, SourceLoader, _register
from importlib.machinery import ModuleSpec


class AaaLoader(SourceLoader):
    def __init__(self, fullname, path, target):
        self.path = path

    def get_data(self, path):
        last_mile = path.split('.')[-1]
        with open(os.path.join(self.path, f'{last_mile}.aaa')) as fh:
            return fh.read()

    def get_filename(self, fullname):
        return fullname

    def module_repr(self, module):
        return f'<module {module.__name__}>'

    def run_file(self, file_name, argv=None, params=None):
        with open(file_name):
            script = f.read()

        file_name_parent_dir = os.path.abspath(os.path.dirname(file_name))
        if file_name_parent_dir not in sys.path:
            sys.path.insert(0, file_name_parent_dir)

        return self.run_script(script, file_name, argv, params)


class AaaFinder(MetaPathFinder):
    @classmethod
    def find_spec(cls, full_name, paths=None, target=None):
        last_mile = full_name.split('.')[-1]
        python_path = paths if paths else sys.path

        for path in python_path:
            full_path = os.path.join(path, last_mile)
            module_full_path = f'{full_path}.aaa'

            if os.path.isfile(module_full_path):
                return ModuleSpec(
                    name=full_name,
                    loader=AaaLoader(full_name, path, target),
                    origin=module_full_path
                )

        return None


finder = AaaFinder()
sys.meta_path.insert(0, finder)

from a.b.c import d
d.hello()
```

## from import
## перезагрузка модуля - почему плохо?

imp.reload(module)
