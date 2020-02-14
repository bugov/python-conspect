# Как из глобальных переменных и функций сделать класс

по мотивам ДЗ

# Повтор + продолжение генераторов

```Python
def get_gen():
    prev = 0
    cur = 1

    while True:
        yield start
        prev, cur = cur, prev + cur
```

Фибоначи:

```Python
class Fib:
    def __init__(self):
        self.prev = 0
        self.cur = 1

    def __iter__(self):
        return self

    def __next__(self):
        self.prev, self.cur = self.cur, self.prev + self.cur
        return self.cur
```

```Python
matrix = list(zip(range(10), range(10, 20), range(20, 30)))
list(zip(*matrix))
```

– транспонирование.

itertools, operator

# Декораторы

## паттерн

– структурный шаблон проектирования, предназначенный для
динамического подключения дополнительного поведения к объекту.

## для функций, для класса

```Python
def log(func):
    def wrap(*args, **kwargs):
        print(f"Starting {func.__name__}")
        func(*args, **kwargs)
        print(f"{func.__name__} done")
    return wrap


@log
def something(num):
    print(f"Gotta {num}")


@log
class A:
    def __init__(self):
        print("Hello")
```

+ кеширующий
+ логирующий @log(prefix), потом функцю log переписать в класс

```Python
def log(type):
    def decorator(func):
        def wrap(*args, **kwargs):
            print(f"[{type}] Starting {func.__name__}")
            func(*args, **kwargs)
            print(f"[{type}] {func.__name__} done")
        return wrap
    return decorator


@log("WARN")
def something(num):
    print(f"Gotta {num}")
```

```Python
class Log:
    def __init__(self, type):
        self.type = type

    def __call__(self, func):
        def wrap(*args, **kwargs):
            print(f"[{self.type}] Starting {func.__name__}")
            func(*args, **kwargs)
            print(f"[{self.type}] {func.__name__} done")
        return wrap


@Log("WARN")
def something(num):
    print(f"Gotta {num}")
```

## написать `@singleton` (`__new__`, `__init__`)

Делаем "руками" синглтон:

```Python
class A:
    def __new__(cls):
        if not hasattr(cls, '_instance'):
            cls._instance = super().__new__(cls)

        return cls._instance


a = A()
a.text = "hello"
b = A()
print(b.text)
```

Декоратором:

```Python
def singleton(cls):
    def wrap(*args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = cls(*args, **kwargs)

        return cls._instance
    return wrap


@singleton
class A:
    pass


a = A()
a.text = "hello"
b = A()
print(b.text)
```
