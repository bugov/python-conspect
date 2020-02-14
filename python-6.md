# Бинарные файлы

## struct

Зачем надо парсить на таком уровне?

```Python
import socket, sys
from struct import *


def main():
    try:
        sock = socket.socket(
            socket.AF_INET,
            socket.SOCK_RAW,
            socket.IPPROTO_TCP,
        )
    except socket.error as e:
        print(f'Error Code : {str(e[0])} Message {e[1]}')
        sys.exit(1)

    while True:
        sniff(sock)


def sniff(sock):
    packets = sock.recvfrom(65565)
    packet = packets[0]

    # ip header
    ip_header = packet[0:20]

    # ! - ntoh
    # B	unsigned char
    # H	unsigned short
    # s	char[]
    iph = unpack('!BBHHHBBH4s4s' , ip_header)
    version_ihl = iph[0]
    version = version_ihl >> 4
    ihl = version_ihl & 0xF
    iph_length = ihl * 4
    ttl = iph[5]
    protocol = iph[6]
    s_addr = socket.inet_ntoa(iph[8]);
    d_addr = socket.inet_ntoa(iph[9]);

    print(
f"""
Version:\t{version}
IP Header Length:\t{ihl}
TTL:\t{ttl}
Protocol:\t{protocol}
Source:\t{s_addr}
Destination:\t{d_addr}
"""
)

    # tcp header
    tcp_header = packet[iph_length:iph_length+20]
    # L unsigned long
    tcph = unpack('!HHLLBBHHH' , tcp_header)
    s_port = tcph[0]
    d_port = tcph[1]
    sequence = tcph[2]
    acknowledgement = tcph[3]
    doff_reserved = tcph[4]
    tcph_length = doff_reserved >> 4

    print(
f"""
Source Port:\t{s_port}
Destination Port:\t{d_port}
"""
)

    h_size = iph_length + tcph_length * 4
    data_size = len(packet) - h_size
    data = packet[h_size:]

    print(f'Data : {data}')
    print("=" * 79)

if __name__ == '__main__':
    main()
```

## open('b'), seek, tell, read

```Python
#! /usr/bin/env python3
import sys


def file_size(file_name):
    with open(file_name) as fh:
        content = fh.read()

    print(len(content))


if __name__ == '__main__':
    file_size(sys.argv[1])
```

`mode='rb'` – read bytes

seek - передвинуть "указатель" файла на N байт (не букв! Привет, utf!).

tell – угадываем.

read – уже видели, но всё же `help(read)`.

## прочитать файл с конца

```Python
#! /usr/bin/env python3
import os
import sys


def reverse_file(file_name):
    size = os.path.getsize(file_name)

    with open(file_name) as fh:
        while size > 0:
            size -= 1
            fh.seek(size)
            print(f'{fh.read(1)}', end='')


if __name__ == '__main__':
    reverse_file(sys.argv[1])
```

## tar parser

```Python
#! /usr/bin/env python3
import sys
from struct import unpack

# Offset   Length   Contents
#   0    100 bytes  File name ('\0' terminated, 99 maxmum length)
# 100      8 bytes  File mode (in octal ascii)
# 108      8 bytes  User ID (in octal ascii)
# 116      8 bytes  Group ID (in octal ascii)
# 124     12 bytes  File size (s) (in octal ascii)
# 136     12 bytes  Modify time (in octal ascii)
# 148      8 bytes  Header checksum (in octal ascii)
# 156      1 bytes  Link flag
# 157    100 bytes  Linkname ('\0' terminated, 99 maxmum length)
# 257      8 bytes  Magic ("ustar  \0")
# 265     32 bytes  User name ('\0' terminated, 31 maxmum length)
# 297     32 bytes  Group name ('\0' terminated, 31 maxmum length)
# 329      8 bytes  Major device ID (in octal ascii)
# 337      8 bytes  Minor device ID (in octal ascii)
# 345    167 bytes  Padding
# 512   (s+p)bytes  File contents (s+p) := (((s) + 511) & ~511), round up to 512 bytes
def tar_stat(file_name):
    with open(file_name, mode='rb') as fh:
        content = fh.read(106)
        tar = unpack('@100sHHH' , content)
        print(f"Name: {tar[0]}")
        print(f"Mode: {tar[1]}")
        print(f"UID:  {tar[2]}")
        print(f"GID:  {tar[2]}")


if __name__ == '__main__':
    tar_stat(sys.argv[1])
```
