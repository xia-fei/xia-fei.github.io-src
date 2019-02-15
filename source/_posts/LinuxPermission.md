---
title: Linux目录读写执行权限分析
date: 2019-01-30 10:45:47
tags: linux
---


对目录设置权限 是指控制目录里面的文件  
```bash
[xkz@10_29_1_54 ~]$ chmod 000 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
d--------- 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ ls -l test
ls: cannot open directory test: Permission denied
```

单执行单写单读
```bash
[xkz@10_29_1_54 ~]$ chmod 111 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
d--x--x--x 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ ls -l test
ls: cannot open directory test: Permission denied

[xkz@10_29_1_54 ~]$ chmod 222 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
d-w--w--w- 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ ls -l test
ls: cannot open directory test: Permission denied

[xkz@10_29_1_54 ~]$ chmod 444 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
dr--r--r-- 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ ls -l test
ls: cannot access test/f1: Permission denied
total 0
-????????? ? ? ? ?            ? f1
```

里面文件有权限，文件夹只有执行权限 可读
```bash
[xkz@10_29_1_54 ~]$ chmod 777 test/f1 
[xkz@10_29_1_54 ~]$ chmod 111 test
[xkz@10_29_1_54 ~]$ ls -l test
ls: cannot open directory test: Permission denied

[xkz@10_29_1_54 ~]$ cat test/f1
hello
```


需要 读和执行权限`5` 同时才算 我们理解的有权限
```bash
读+写
[xkz@10_29_1_54 ~]$ chmod 666 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
drw-rw-rw- 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ cat test/f1
cat: test/f1: Permission deniedc
读+执行
[xkz@10_29_1_54 ~]$ chmod 555 test
[xkz@10_29_1_54 ~]$ ls -l
total 4
dr-xr-xr-x 2 xkz xkz 4096 Jan 30 10:47 test
[xkz@10_29_1_54 ~]$ ls -l test
total 4
-rw-rw-r-- 1 xkz xkz 6 Jan 30 10:47 f1
[xkz@10_29_1_54 ~]$ cat test/f1 
hello
```
