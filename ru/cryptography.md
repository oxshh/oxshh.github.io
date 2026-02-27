---
title: Криптография
nav_order: 3
parent: Русский
---
# Криптография
Заметки по криптографии.

XOR

Source code
```
/challenge/run
#!/opt/pwn.college/python

import random
import sys

key = random.randrange(1, 256)
plain_secret = random.randrange(0, 256)
cipher_secret = plain_secret ^ key

print(f"The key: {key}")
print(f"Encrypted secret: {cipher_secret}")
if int(input("Decrypted secret? ")) == plain_secret:
    print("CORRECT! Your flag:")
    print(open("/flag").read())
else:
    print("INCORRECT!")
    sys.exit(1)

hacker@cryptography~xor:/$ /challenge/run 
The key: 17
Encrypted secret: 47
Decrypted secret? 

In [1]: key = 17
   ...: encrypted_secret = 47
   ...: decrypted_secret = key ^ encrypted_secret
   ...: print(f"Decrypted secret: {decrypted_secret}")
Decrypted secret: 62

Now we can provide this as the answer.

hacker@cryptography~xor:/$ /challenge/run 
The key: 17
Encrypted secret: 47
Decrypted secret? 62
CORRECT! Your flag:
```
Автоматизируем этот процесс, чтобы он работал с любыми входными данными.

Exploit

~/script.py

```
#!/usr/bin/env python3

import subprocess
import re

# Запустите исполняемый файл с заданием и зафиксируйте вывод.
proc = subprocess.Popen(["/challenge/run"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)

# Прочитайте ключ и зашифрованный код из выходных данных.
output = proc.stdout.readline()
key = int(re.search(r'\d+', output).group())

output = proc.stdout.readline()
encrypted_secret = int(re.search(r'\d+', output).group())

# Восстановите открытый секрет с помощью операции XOR.
decrypted_secret = key ^ encrypted_secret

# Отправьте восстановленный расшифрованный секретный ключ в качестве входных данных.
proc.stdin.write(f"{decrypted_secret}\n")
proc.stdin.flush()

# Выведите оставшийся результат (CORRECT! или INCORRECT! и, возможно, флаг).
for line in proc.stdout:
    print(line, end="")
```