# Наследование

## super()

+ задача: взять имплементацию метода предка предка.

```Python
class A:
    def hello(self):
        print("Hello A")


class B(A):
    def hello(self):
        super().hello()
        print("Hello B")


class C(B):
    def hello(self):
        # super().super().hello()
        super(B, self).hello()
        super(self.__class__.__mro__[0], self).hello()
        print("Hello C")
```

## исключения

try/except/else/finally

```Python
import sys

try:
  print("try")
  raise KeyError
except (KeyError, Exception) as e:
  print("except (KeyError, Exception)")
  sys.exit(1)
except Exception as e:
  print("except Exception")
except KeyError as e:
  print("except KeyError")
finally:
  print("finally")
```

```Python
d = dict()

try:
  d.key = d['key']
except (KeyError, AttributeError) as e:
  raise SpecificException from e
```

## context managers

```Python
class File():
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode

    def __enter__(self):
        self.open_file = open(self.filename, self.mode)
        return self.open_file

    def __exit__(self, *args):
        self.open_file.close()


files = []
for _ in range(10000):
    with File('foo.txt', 'w') as infile:
        infile.write('foo')
        files.append(infile)
```

## пишем КМ в виде класса и генератора

```Python
from contextlib import contextmanager

@contextmanager
def tag(name):
    print("<%s>" % name)
    yield
    print("</%s>" % name)
```

# Вопросы по пи-таску, что я записал

- что с pep8 для Татьяны - http://rukeba.com/by-the-way/pep8-korotko-i-po-russki/ - 80%
  этого - это 70% баллов.
- bmp для девочек
  1. графический интерфейс для просмотра уровня шума при изменении битов стегано
     с возможностью выгрузки результата
  2. поиск стеганографии в бмп - под кодировки cp1251, utf-8, utf-9. Возможность
     задать сразу директорию с файлами для анализа. Возможность использовать словарь
  3. http-proxy, вшивающий стегано в проходящие бмп-файлы.
- smtp для Рината - добавить планировщик отправки сообщений: через N минут, в такое-то время.
  При перезагрузке должен продолжать работать.
- нечёткий поиск для Паши - анонимайзер, позволяющий цензурировать некоторые слова методом
  нечёткого поиска.
- Задача с шаблонизатором для Андрея. Сделать на qt IDE для шаблонизатора -
  чтобы можно было редактировать шаблон, стили, видеть это всё и выводить в файл
  результат.
