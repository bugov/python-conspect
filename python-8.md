# Работа с сетью (синхронная)

## Общие знания про сети (tcp/ip стек) - половина пары

Рисуем сеть:

0. Витая пара, Физический: соединяем 2 компа шнуром, общаемся - передаём сами данные.
1. MAC, Канальный: уже не 2 компа, а десяток - надо ещё и адрес, кому сообщение.
2. IP, Сетевой: уже не десяток, а тысяча: если все начнут друг с другом общаться
   – получится каша, никто никого не услышит – надо рассадить "говорунов" по
   разным "комнатам".
3. Port, Транспортный: ну хорошо, доставили мы до хоста сообщение... но там ведь
   может быть поднятно несколько сетевых сервисов - надо указать, к какому
   конкретно.
4. Разное, Прикладной: взаимодействие на уровне приложения, например, запрос HTTP
   страницы веб-сервера.

И всё пакуется одно в другое на каждом уровне по типу цифрового пакета:

```
+--------------------+
| Заголовок | Данные |
+--------------------+
```

# модуль socket - скачиваем страницу e1.ru

```Python
import socket
from contextlib import contextmanager


@contextmanager
def socket_context(*args, **kwargs):
    sock = socket.socket(*args, **kwargs)
    try:
        yield sock
    finally:
        sock.close()


def read_website(domain, port, timeout=10):
    result = b''
    with socket_context(socket.AF_INET, socket.SOCK_STREAM) as sock:
        sock.settimeout(timeout)
        sock.connect((domain, port))
        sock.sendall(
            b"GET / HTTP/1.1\r\nHost: %b\r\n\r\n"
            % bytes(domain, encoding='utf-8')
        )

        chunk = sock.recv(1024)
        result += chunk
        while chunk:
            chunk = sock.recv(1024)
            result += chunk

    return result


def main():
    read_website('e1.ru', 80)


if __name__ == '__main__':
    main()
```

## send/sendall

```Python
def sendall(sock, data, flags=0):
    ret = sock.send(data, flags)
    if ret > 0:
        return sendall(sock, data[ret:], flags)
    else:
        return None
```

- recv - блокирующий, может вернуть меньше, пример
- загрузить файл - где передавать размер / терм. символ?
- shutdown

- Запускаем скрипт на преподской машине tcp 13000 - 30 минут
  - help
  - собираем флаги python*

## Пишем сервак
### Однопоточный echo-сервер

```Python
import socket

def listen():
    connection = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    #connection.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    connection.bind(('0.0.0.0', 5555))
    connection.listen(10)

    while True:
        current_connection, address = connection.accept()

        while True:
            data = current_connection.recv(2048)

            if data == 'quit\r\n':
                current_connection.shutdown(1)
                current_connection.close()
                break

            elif data == 'stop\r\n':
                current_connection.shutdown(1)
                current_connection.close()
                exit()

            elif data:
                current_connection.send(data)
                print(data)


if __name__ == "__main__":
    try:
        listen()
    except KeyboardInterrupt:
        pass
```
