---
layout: post
title: "從gdb dump process 記憶體資料"
date: 2019-03-20 22:29:54 +0800
comments: true
categories:  [gdb, debug, trace]
---

無聊在trace [vdso](http://man7.org/linux/man-pages/man7/vdso.7.html) 找到的技巧。整理如下

## 方法說明

* `gdb 你要的程式`
* `gdb` 設常用的 `system call` 如 `open`
* 執行程式
* 中斷後
    * `info proc mappings`
    * `dump memory 檔名 開始位址 結束位址`
    * 離開
* 剩下看你拿要dump 的檔案做啥了

## 範例： dump process中的vdso記憶體區塊，觀察vdso symbol

* `gdb 你要的程式`
```
$ gdb /bin/ls
GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git
...
Reading symbols from /bin/ls...(no debugging symbols found)...done.
(gdb) 
```

* `gdb` 設常用的 `system call` 如 `open`

```
(gdb) b open
Breakpoint 1 at 0x3d30
```

* 執行程式

```
(gdb) r
Starting program: /bin/ls 

Breakpoint 1, __libc_open64 (file=file@entry=0x7ffff7df6428 "/etc/ld.so.cache", oflag=oflag@entry=524288)
    at ../sysdeps/unix/sysv/linux/open64.c:39
39	../sysdeps/unix/sysv/linux/open64.c: No such file or directory.
```

* `info proc mapings`

```
(gdb) info proc mappings 
process 30094
Mapped address spaces:

          Start Addr           End Addr       Size     Offset objfile
      0x555555554000     0x555555573000    0x1f000        0x0 /bin/ls
      0x555555772000     0x555555775000     0x3000    0x1e000 /bin/ls
      0x555555775000     0x555555776000     0x1000        0x0 [heap]
      0x7ffff7dd5000     0x7ffff7dfc000    0x27000        0x0 /lib/x86_64-linux-gnu/ld-2.27.so
      0x7ffff7ff7000     0x7ffff7ffa000     0x3000        0x0 [vvar]
      0x7ffff7ffa000     0x7ffff7ffc000     0x2000        0x0 [vdso]
      0x7ffff7ffc000     0x7ffff7ffe000     0x2000    0x27000 /lib/x86_64-linux-gnu/ld-2.27.so
      0x7ffff7ffe000     0x7ffff7fff000     0x1000        0x0 
      0x7ffffffde000     0x7ffffffff000    0x21000        0x0 [stack]
  0xffffffffff600000 0xffffffffff601000     0x1000        0x0 [vsyscall]
```

* `dump memory 檔名 開始位址 結束位址`

```
(gdb) dump memory vdso 0x7ffff7ffa000  0x7ffff7ffc000
(gdb) quit
A debugging session is active.

	Inferior 1 [process 30094] will be killed.

Quit anyway? (y or n) y
```

* 看dump 檔案 symbol

```
$ file vdso 
vdso: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, BuildID[sha1]=cef5ed3c3dc2b39082ae093560c168a8b427ebb6, stripped
$ nm -D vdso 
0000000000000a30 W clock_gettime
0000000000000f30 W getcpu
0000000000000d40 W gettimeofday
0000000000000000 A LINUX_2.6
0000000000000f10 W time
0000000000000a30 T __vdso_clock_gettime
0000000000000f30 T __vdso_getcpu
0000000000000d40 T __vdso_gettimeofday
0000000000000f10 T __vdso_time
```

## 加碼，觀察ASLR

懶的說明ASLR，請自行參考[連結](https://en.wikipedia.org/wiki/Address_space_layout_randomization)。
另外請自行比較下面兩個 `cat` 的記憶體區塊位址。

```
$ cat /proc/self/maps
5601e5311000-5601e5319000 r-xp 00000000 103:02 13631603                  /bin/cat
...
5601e6814000-5601e6835000 rw-p 00000000 00:00 0                          [heap]
...
7ffe6edcf000-7ffe6edf0000 rw-p 00000000 00:00 0                          [stack]
7ffe6edfb000-7ffe6edfe000 r--p 00000000 00:00 0                          [vvar]
7ffe6edfe000-7ffe6ee00000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]


$ cat /proc/self/maps
556ef8185000-556ef818d000 r-xp 00000000 103:02 13631603                  /bin/cat
...
556ef91df000-556ef9200000 rw-p 00000000 00:00 0                          [heap]
...
7ffc83c55000-7ffc83c76000 rw-p 00000000 00:00 0                          [stack]
7ffc83d4a000-7ffc83d4d000 r--p 00000000 00:00 0                          [vvar]
7ffc83d4d000-7ffc83d4f000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]

```
