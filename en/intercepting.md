---
title: Intercepting Communication
nav_order: 2
parent: English
---
# Intercepting Communication
Notes and practice on intercepting communication.

# Communication Interception
Notes on Communication Interception.

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
In this task, you need to connect to host 10.0.0.2 on port 31337. The server immediately sends the contents of /flag.

Execution example:

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
In this task, the server waits for a message from the client. If the client sends the string "Hello, World!", the server will return the contents of /flag.

Example execution:

Code
```
root@ip-10-0-0-1:/# nc 10.0.0.2 31337
Hello, World!
```
Thus:

In Connect, simply connect to the port to receive the flag.

In Send, you must first send the string "Hello, World!", and only then will the server return the flag.