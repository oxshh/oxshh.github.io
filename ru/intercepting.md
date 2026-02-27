---
title: Перехват коммуникаций
nav_order: 2
parent: Русский
---
# Перехват коммуникаций
Заметки по перехвату коммуникаций.

Connect

Source code
/challenge/run

Code
```
#!/usr/bin/exec-suid --real -- /usr/bin/python -I

import os
import socket

import psutil
from dojjail import Host, Network

flag = open("/flag").read()
parent_process = psutil.Process(os.getppid())

class ServerHost(Host):
    def entrypoint(self):
        server_socket = socket.socket()
        server_socket.bind(("0.0.0.0", 31337))
        server_socket.listen()
        while True:
            try:
                connection, _ = server_socket.accept()
                connection.sendall(flag.encode())
                connection.close()
            except ConnectionError:
                continue

user_host = Host("ip-10-0-0-1", privileged_uid=parent_process.uids().effective)
server_host = ServerHost("ip-10-0-0-2")
network = Network(hosts={user_host: "10.0.0.1", server_host: "10.0.0.2"}, subnet="10.0.0.0/24")
network.run()

user_host.interactive(environ=parent_process.environ())
```
В этом задании нужно подключиться к хосту 10.0.0.2 на порт 31337. Сервер сразу отправляет содержимое /flag.

Пример выполнения:

Code
```
hacker@intercepting-communication~connect:/$ /challenge/run 
root@ip-10-0-0-1:/#

root@ip-10-0-0-1:/# nc 10.0.0.2 31337
Send
Source code
/challenge/run
```
Code
```
#!/usr/bin/exec-suid --real -- /usr/bin/python -I

import os
import socket

import psutil
from dojjail import Host, Network

flag = open("/flag").read()
parent_process = psutil.Process(os.getppid())

class ServerHost(Host):
    def entrypoint(self):
        server_socket = socket.socket()
        server_socket.bind(("0.0.0.0", 31337))
        server_socket.listen()
        while True:
            try:
                connection, _ = server_socket.accept()
                while True:
                    client_message = connection.recv(1024).decode()
                    if not client_message:
                        break
                    if client_message == "Hello, World!\n":
                        connection.sendall(flag.encode())
                        break
                connection.close()
            except ConnectionError:
                continue

user_host = Host("ip-10-0-0-1", privileged_uid=parent_process.uids().effective)
server_host = ServerHost("ip-10-0-0-2")
network = Network(hosts={user_host: "10.0.0.1", server_host: "10.0.0.2"}, subnet="10.0.0.0/24")
network.run()

user_host.interactive(environ=parent_process.environ())
```
В этом задании сервер ждёт сообщение от клиента. Если клиент отправит строку "Hello, World!\n", сервер вернёт содержимое /flag.

Пример выполнения:

Code
```
root@ip-10-0-0-1:/# nc 10.0.0.2 31337
Hello, World!
```
Таким образом:

В Connect достаточно просто подключиться к порту, чтобы получить флаг.

В Send нужно сначала отправить строку "Hello, World!", и только после этого сервер вернёт флаг.