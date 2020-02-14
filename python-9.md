# Работа с сетью (конкурентная)

Вспоминаем стек TCP/IP, понятие сокета.

## Пишем сервак
### Однопоточный echo-сервер

```Python
import socket


def handler(conn):
    while True:
        data = conn.recv(2048)

        if data == 'quit\r\n':
            conn.shutdown(1)
            conn.close()
            break

        elif data == 'stop\r\n':
            conn.shutdown(1)
            conn.close()
            exit()

        if not data:
            break

        conn.send(data)
        print(data)


def listen():
    connection = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    #connection.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    connection.bind(('0.0.0.0', 5555))
    connection.listen(10)

    while True:
        current_connection, address = connection.accept()
        handler(current_connection)


if __name__ == "__main__":
    try:
        listen()
    except KeyboardInterrupt:
        pass
```

### Многопроцессный

```Python
import os
import socket


def handler(conn):
    while True:
        data = conn.recv(2048)

        if data == 'quit\r\n':
            conn.shutdown(1)
            conn.close()
            break

        elif data == 'stop\r\n':
            conn.shutdown(1)
            conn.close()
            exit()

        if not data:
            break

        conn.send(data)
        print(data)


def listen():
    connection = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    #connection.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    connection.bind(('0.0.0.0', 5555))
    connection.listen(10)

    while True:
        current_connection, address = connection.accept()
        pid = os.fork()  # https://ru.wikipedia.org/wiki/Fork

        if pid == 0:
            handler(current_connection)


if __name__ == "__main__":
    try:
        listen()
    except KeyboardInterrupt:
        pass
```

### Многопоточный

```Python
import socket
from threading import Thread  # https://ru.wikipedia.org/wiki/POSIX_Threads


class HandlerThread(Thread):
    def __init__(self, conn):
        super().__init__()
        self.conn = conn

    def run(self):
        while True:
            data = self.conn.recv(2048)

            if data == 'quit\r\n':
                self.conn.shutdown(1)
                self.conn.close()
                break

            elif data == 'stop\r\n':
                self.conn.shutdown(1)
                self.conn.close()
                exit()

            if not data:
                break

            self.conn.send(data)
            print(data)

        self.conn.shutdown(1)
        self.conn.close()


def listen():
    connection = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    #connection.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    connection.bind(('0.0.0.0', 5555))
    connection.listen(10)

    while True:
        current_connection, address = connection.accept()
        handler = HandlerThread(current_connection)
        handler.start()


if __name__ == "__main__":
    try:
        listen()
    except KeyboardInterrupt:
        pass
```

# GIL

https://ru.wikipedia.org/wiki/Global_Interpreter_Lock

```Python
import threading

num = 0


def doubler(num_lock):
    global num;
    print(threading.currentThread().getName() + '\n')
    for _ in range(1000000):
        num += 1


def main():
    threads = []
    for i in range(5):
        threads.append(threading.Thread(target=doubler, args=(num_lock,)))

    for t in threads:
        t.start()

    for t in threads:
        t.join()

    print(num)


if __name__ == '__main__':
    main()
```

# локи (Lock, RLock), гонки

```Python
import threading

num = 0
num_lock = threading.Lock()


def doubler(num_lock):
    global num;
    print(threading.currentThread().getName() + '\n')
    for _ in range(1000000):
        with num_lock:
            num += 1


def main():
    threads = []
    for i in range(5):
        threads.append(threading.Thread(target=doubler, args=(num_lock,)))

    for t in threads:
        t.start()

    for t in threads:
        t.join()

    print(num)


if __name__ == '__main__':
    main()
```

# IPC: mmap, shared mem, etc

```
Файл	Все ОС.
Сигнал	Большинство ОС; в некоторых ОС, например, в Windows, сигналы доступны только в библиотеках, реализующих стандартную библиотеку языка Си, и не могут использоваться для IPC.
Сокет	Большинство ОС.
Канал	Все ОС, совместимые со стандартом POSIX.
Именованный канал	Все ОС, совместимые со стандартом POSIX.
Неименованный канал	Все ОС, совместимые со стандартом POSIX.
Семафор	Все ОС, совместимые со стандартом POSIX.
Разделяемая память	Все ОС, совместимые со стандартом POSIX.
Обмен сообщениями
(без разделения)	Используется в парадигме MPI, Java RMI, CORBA и других.
Проецируемый в память файл (mmap)	Все ОС, совместимые со стандартом POSIX. При использовании временного файла возможно возникновение гонки. ОС Windows также предоставляет этот механизм, но посредством API, отличающегося от API, описанного в стандарте POSIX.
Очередь сообщений (Message queue)	Большинство ОС.
Почтовый ящик	Некоторые ОС.
```
