# 2. вторая

- py36

  https://github.com/pyenv/pyenv

  ```
  git clone https://github.com/pyenv/pyenv.git ~/.pyenv
  echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
  echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
  echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' \
      >> ~/.bash_profile
  exec $SHELL
  pyenv install 3.6.4
  ```
- ipython notebook
  ```
  pip3 install ipython
  ipython
  ```
- tuple, list, set, frozenset, dict (in, оперции -&|+, for, enumerate)
- баян [::]
- str, bytes, bytearray
- .join – забыл про это рассказать :( напомните на след. паре. Коротко:
  ```
  " ".join(["a", "b", "c"])  # "a b c"
  ```
- '{}'.format(x), '%r' % x, f'{x}', r'', b'', u''
  Форматирование .format
  ```
  "Имя \"{name:s}\". Возраст: {age:d}".format(name="Ваня", age=8)
  "Блюдо \"{:s}\". Сумма: {:04.02f}".format("Борщ", 80)
  ```

  Форматирование "двумя перстами" – вариант "староверов":
  ```
  "%r" % [1, 2, 3]
  "Блюдо \"%s\". Сумма: %04.02f" % ("Борщ", 80)
  "Блюдо \"%(name)s\". Сумма: %(cost)04.02f" % {
    "name": "Борщ",
    "cost": 80,
  }
  ```

  f-строки (новый, правильный вариант):
  ```
  name = "Ваня"
  age = 8
  f"Имя \"{name}\". Возраст: {age}"
  f"Имя \"{name:s}\". Возраст: {age:03d}"
  ```

  Немного извращений с f-строками:
  ```
  In [2]: import random

  In [3]: dimension = 4

  In [4]: f'{random.randrange(1, 10**dimension):0{dimension}}'
  Out[4]: '1331'
  ```
- open/close, with open
- from collections import defaultdict, Counter
- Задачка. Посчитать общую статистику имён в ftp://shannon.usu.edu.ru/python/hw2/home.html
  ```
  from collections import Counter
  from urllib.request import urlopen

  content = urlopen('ftp://shannon.usu.edu.ru/python/hw2/home.html')

  with content as fh:
      html_b = fh.read()

  html = str(html_b, encoding="utf-8")

  names = Counter()
  while(html.find('<a') != -1):
      html = html[html.find('<a'):]
      html = html[html.find('>') + 1:]

      full_name = html[:html.find('<')]
      last, first = full_name.split()
      names[first] += 1

  print(names)
  ```

  Другой вариант:
  ```
  from collections import Counter
  from urllib.request import urlopen
  from bs4 import BeautifulSoup

  with urlopen('ftp://shannon.usu.edu.ru/python/hw2/home.html') as fp:
      print(
        Counter([
            x.text.split()[1]
            for x in BeautifulSoup(
                str(fp.read(), encoding='cp1251')
            ).find_all('a')
        ])
      )
  ```

- str.find, str.index
- Домой:
  Сохранить страницу локально. Реализовать недостающий функционал в скрипте:
  ftp://shannon.usu.edu.ru/python/hw2/homestat_stripped.py. Прилагаются тесты:
  ftp://shannon.usu.edu.ru/python/hw2/test_homestat.py.
  Требования:
  Статистика по именам за каждый год (2 балла)
  Соответствие PEP8 (1 балл)
  Отдельная статистика для мальчиков, отдельная для девочек (1 балл, опционально)
