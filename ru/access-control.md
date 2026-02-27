---
title: Контроль доступа
nav_order: 4
parent: Русский
---
# Контроль доступа
Заметки по контролю доступа.

Этот учебный модуль ценен тем, что позволяет на практике освоить разные подходы к управлению доступом. На каждом этапе участнику предлагается запустить исполняемый файл, находящийся в каталоге /challenge/run. После запуска условия работы с файлом, содержащим флаги, изменяются особым образом. Задача заключается в том, чтобы найти способ получить его содержимое, используя предоставленные возможности и ограничения системы.

Level 1:
```
$ /challenge/run 
===== Welcome to Access Control! =====
In this series of challenges, you will be working with various access control systems.
Break the system to get the flag.


In this challenge you will work with different UNIX permissions on the flag.

The flag file will be owned by you and have 400 permissions.


Before:
-r-------- 1 root root 58 Oct  7 10:07 /flag
After:
-r-------- 1 hacker root 58 Oct  7 10:07 /flag

$ cat /flag
<flag_value>
```
Теория: Права собственности на файл были изменены на права пользователя, и у пользователя достаточно разрешений, чтобы просто прочитать флаг с помощью утилиты cat.

Level 2:
```
$ /challenge/run 
===== Welcome to Access Control! =====
In this series of challenges, you will be working with various access control systems.
Break the system to get the flag.


In this challenge you will work with different UNIX permissions on the flag.

The flag file will be owned by root, group as you, and have 040 permissions.


Before:
-r-------- 1 root root 58 Oct  7 10:11 /flag
After:
----r----- 1 root hacker 58 Oct  7 10:11 /flag

$ cat /flag
<flag_value>

Theory: The file's ownership group was changed to our group and following same principles we can simply read the flag by using the cat utility.
```
Level 3:
```
$ /challenge/run 
===== Welcome to Access Control! =====
In this series of challenges, you will be working with various access control systems.
Break the system to get the flag.


In this challenge you will work with different UNIX permissions on the flag.

The flag file will be owned by you and have 000 permissions.


Before:
-r-------- 1 root root 58 Oct  7 10:12 /flag
After:
---------- 1 hacker root 58 Oct  7 10:12 /flag

$ chmod u+r /flag             # // symbolic value by adding (+) 'r' (read) permissions to 'u' (user), while noting that 'g' stands for (group), and 'o' stands for(other) 
$ chmod 400 /flag             # // numeric value representation with '4' representing read permissions and '00' for ---/--- (no permissions) for group/other

$ cat /flag
<flag_value>
```
Теория: Пользователь, которому принадлежит файл, назначен нашему пользователю, но у него нет установленных прав доступа. Мы можем решить эту проблему,

изменив права доступа к файлу двумя способами, используя числовые значения команды chmod или символические значения.
Level 4:

```
$ /challenge/run 
===== Welcome to Access Control! =====
In this series of challenges, you will be working with various access control systems.
Break the system to get the flag.


In this challenge you will work understand how the SETUID bit for UNIX permissions works.

What if /bin/cat had the SETUID bit set?


Before:
-rwxr-xr-x 1 root root 43416 Sep  5  2019 /bin/cat
After:
-rwsr-xr-x 1 root root 43416 Sep  5  2019 /bin/cat

$ cat /flag
<flag_value>
```
Теория: Утилите «cat» было предоставлено разрешение setuid, которое фактически дает пользователю возможность вызывать «cat»
с идентификатором пользователя-владельца, которым в данном случае является root, предоставляя пользователю полные права на использование утилиты
по своему усмотрению.