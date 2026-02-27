---
title: Access Control
nav_order: 4
parent: English
---
# Access Control
Notes and practice on access control.

This training module is valuable because it allows you to practice various approaches to access control. At each stage, the participant is asked to run an executable file located in the /challenge/run directory. After running, the conditions for working with the file containing the flags are modified in a specific way. The task is to find a way to retrieve its contents using the provided capabilities and system restrictions.

Level 1:
```
$ /challenge/run
===== Welcome to Access Control! =====
In this series of challenges, you will work with various access control systems.
Break the system to get the flag.

In this challenge, you will work with different UNIX permissions on the flag.

The flag file will be owned by you and have 400 permissions.

Before:
-r-------- 1 root root 58 Oct 7 10:07 /flag
After:
-r-------- 1 hacker root 58 Oct 7 10:07 /flag

$ cat /flag
<flag_value>
```
Theory: The file's ownership has been changed to the user's, and the user has sufficient permissions to simply read the flag using the cat utility.

Level 2:
```
$ /challenge/run
===== Welcome to Access Control! =====
In this series of challenges, you will be working with various access control systems.
Break the system to get the flag.

In this challenge, you will work with different UNIX permissions on the flag.

The flag file will be owned by root, a group like you, and have 040 permissions.

Before:
-r-------- 1 root root 58 Oct 7 10:11 /flag
After:
----r----- 1 root hacker 58 Oct 7 10:11 /flag

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
-r-------- 1 root root 58 Oct 7 10:12 /flag
After:
---------- 1 hacker root 58 Oct 7 10:12 /flag

$ chmod u+r /flag # // symbolic value by adding (+) 'r' (read) permissions to 'u' (user), while noting that 'g' stands for (group), and 'o' stands for (other)
$ chmod 400 /flag # // numeric value representation with '4' representing read permissions and '00' for ---/--- (no permissions) for group/other

$ cat /flag
<flag_value>
```
Theory: The user who owns the file is assigned to our user, but they have no permissions set. We can solve this problem by

changing file permissions in two ways: using numeric values ​​with the chmod command or symbolic values.
Level 4:

```
$ /challenge/run
===== Welcome to Access Control! =====
In this series of challenges, you will work with various access control systems.
Break the system to get the flag.

In this challenge, you will understand how the SETUID bit for UNIX permissions works.

What if /bin/cat had the SETUID bit set?

Before:
-rwxr-xr-x 1 root root 43416 Sep 5 2019 /bin/cat
After:
-rwsr-xr-x 1 root root 43416 Sep 5 2019 /bin/cat

$ cat /flag
<flag_value>
```
Theory: The "cat" utility was granted setuid permission, which effectively gives the user the ability to invoke "cat"
with the user ID of the owner, which in this case is root, granting the user full rights to use the utility
as they see fit.