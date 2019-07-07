---
layout: post
title: "觀察編譯glibc的產出"
date: 2019-06-08 18:48:03 +0800
comments: true
categories:  [ C, Linux, glibc ]
---
這次介紹編譯glibc 並安裝後的一些發現

## 目錄

* [測試環境](#glibc_env)
* [安裝步驟](#glibc_install)
* [觀察與結論](#glibc_conl)
    * [/lib](#glibc_conl_lib)
    * [/bin](#glibc_conl_bin)
    * [/sbin](#glibc_conl_sbin)
    * [/etc](#glibc_conl_etc)
    * [/share](#glibc_conl_share)
    * [/var](#glibc_conl_var)
    * [/libexec](#glibc_conl_libexec)
    * [/include](#glibc_conl_inc)
* [參考資料](#glibc_conl_ref)
* [附錄](#glibc_conl_ex)

<a name="glibc_env"></a>
## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.2 LTS
Release:	18.04
Codename:	bionic
```


<a name="glibc_install"></a>
## 安裝步驟

* 下載套件

```
git clone http://sourceware.org/git/glibc.git
cd glibc
git checkout --track origin/release/2.27
```

* 在新的目錄編譯 glibc，你可以自行指定安裝路徑，<font color=red>**一定要指定安裝路徑，以免發生嚴重悲劇。** </font>

本次設定主要是針對除錯最佳化，以及避免覆蓋系統原本的 `glibc`

```
cd ../
mkdir out
mkdir rootfs
cd out

# 設定
CFLAGS=-Og CPPFLAGS=-Og CXXFLAGS=-Og ../glibc/configure  --disable-werror --prefix=/tmp/rootfs/

# 編譯
make

# 安裝
make install
```

<a name="glibc_conl"></a>
## 觀察與結論

本來想說 `libc` 用來提供 C 標準函式庫的 binary，那麼了不起就是 `libc.so`和 `libc.a` 以及對應的 header files。安裝完畢後先看一下目錄，事情果然沒有像本組裝工想的那麼簡單。列出第一層目錄如下，除了預期中的`lib`和`include`以外，竟然還有不少預期以外的目錄。

```
$ tree -L 1 -d /tmp/rootfs
.
├── bin
├── etc
├── include
├── lib
├── libexec
├── sbin
├── share
└── var
```

那麼我們來看一下這些目錄下面有什麼東西吧。

<a name="glibc_conl_lib"></a>
### /lib

先來看目錄結構，多了和多國語言相關的函式庫目錄還有 trace shared object PLT (Procedure linkage table) 工具會用到的audio目錄

```
$ tree lib -d
lib
├── audit
└── gconv
```

接下來看`/lib`的檔案

列出幾個我有興趣的檔案

* `*.o`
    * 在`/lib`裏面會發現幾個object file，它們檔名都有`crt`，crt全名是 `C runtime`，顯然和執行的時候有關。我有空會再找時間了解。先列出來介紹幾個如下

* `Scrt1.o`: 這邊我們可以看到`T _start`以及`U main`，望文生義按圖說故事我們可以猜測執行程式的起始點其實是`_start`，做了一些事情後才會去呼叫你寫的`main()`，我做了一個實驗，想知道一個應用程式會連結哪些系統上的object檔案請參考[這邊](#glibc_conl_ex)

```
$ nm Scrt1.o 
0000000000000000 D __data_start
0000000000000000 W data_start
                 U _GLOBAL_OFFSET_TABLE_
0000000000000000 R _IO_stdin_used
                 U __libc_csu_fini
                 U __libc_csu_init
                 U __libc_start_main
                 U main
0000000000000000 T _start
```

* `crtn.o`: 用`nm`去看會發現沒有`symbol`，不過反組譯後會發現有兩個`section`，看起來和main啟動前和使用者程式結束後會有關係。有空會再探討。

```
$ objdump -d crtn.o 

crtn.o:     file format elf64-x86-64


Disassembly of section .init:

0000000000000000 <.init>:
   0:	48 83 c4 08          	add    $0x8,%rsp
   4:	c3                   	retq   

Disassembly of section .fini:

0000000000000000 <.fini>:
   0:	48 83 c4 08          	add    $0x8,%rsp
   4:	c3                   	retq   

```

* `ld-linux-x86_64.so.2`
    * 就是`ld.so`，這個檔案有趣的點是他是一個shared object，但是同時又是可以執行。如果我的懶病沒有發作以後會常常看到這個東西。
* `libc.*`:
    * 直接看symbol就知道，`T`, `U`的定義請翻前面文章，我懶得找。

```
$ nm libc.a | grep "^printf.o:" -A 10
printf.o:
0000000000000000 T _IO_printf
0000000000000000 T printf
0000000000000000 T __printf
                 U stdout
                 U __vfprintf_internal

snprintf.o:
0000000000000000 W snprintf
0000000000000000 T __snprintf
                 U __vsnprintf_internal    
```

* `libm.*`: 一樣看symbol節錄

```
$ nm libm.so.6 |grep " sin"
000000000002eb24 i sin
0000000000034532 W sincos
00000000000419cc i sincosf
0000000000054f26 W sincosf128
00000000000419cc i sincosf32
0000000000034532 W sincosf32x
0000000000034532 W sincosf64
00000000000175c8 W sincosf64x
00000000000175c8 W sincosl
000000000004131e i sinf
0000000000054148 W sinf128
000000000004131e i sinf32
000000000002eb24 i sinf32x
000000000002eb24 i sinf64
0000000000016ead W sinf64x
000000000000ed11 W sinh
000000000001205d W sinhf
0000000000060407 W sinhf128
000000000001205d W sinhf32
000000000000ed11 W sinhf32x
000000000000ed11 W sinhf64
000000000000d9a4 W sinhf64x
000000000000d9a4 W sinhl
0000000000016ead W sinl
00000000000144b5 t sin_pi
0000000000027776 t sin_pi
```

* `libdl`
    * 動態載入函式庫相關函數如`dlvsym`, `dlsym`, 

#### 完整檔案如下

```
$ ls lib
audit                         libc.a                 libmemusage.so              libnss_dns.so               librt-2.28.9000.so
crt1.o                        libc_nonshared.a       libm.so                     libnss_dns.so.2             librt.a
crti.o                        libcrypt-2.28.9000.so  libm.so.6                   libnss_files-2.28.9000.so   librt.so
crtn.o                        libcrypt.a             libmvec-2.28.9000.so        libnss_files.so             librt.so.1
gconv                         libcrypt.so            libmvec.a                   libnss_files.so.2           libSegFault.so
gcrt1.o                       libcrypt.so.1          libmvec_nonshared.a         libnss_hesiod-2.28.9000.so  libthread_db-1.0.so
ld-2.28.9000.so               libc.so                libmvec.so                  libnss_hesiod.so            libthread_db.so
ld-linux-x86-64.so.2          libc.so.6              libmvec.so.1                libnss_hesiod.so.2          libthread_db.so.1
libanl-2.28.9000.so           libdl-2.28.9000.so     libnsl-2.28.9000.so         libpcprofile.so             libutil-2.28.9000.so
libanl.a                      libdl.a                libnsl.so.1                 libpthread-2.28.9000.so     libutil.a
libanl.so                     libdl.so               libnss_compat-2.28.9000.so  libpthread.a                libutil.so
libanl.so.1                   libdl.so.2             libnss_compat.so            libpthread.so               libutil.so.1
libBrokenLocale-2.28.9000.so  libg.a                 libnss_compat.so.2          libpthread.so.0             Mcrt1.o
libBrokenLocale.a             libm-2.28.9000.a       libnss_db-2.28.9000.so      libresolv-2.28.9000.so      Scrt1.o
libBrokenLocale.so            libm-2.28.9000.so      libnss_db.so                libresolv.a
libBrokenLocale.so.1          libm.a                 libnss_db.so.2              libresolv.so
libc-2.28.9000.so             libmcheck.a            libnss_dns-2.28.9000.so     libresolv.so.2
```


<a name="glibc_conl_bin"></a>
### /bin

是除了 `ldd`以外，我全部沒印象。有些甚至不在Ubuntu的預設安裝中。使用者需要另外安裝，如`xtrace`等。

列出幾個我有興趣的工具

* [pldd](http://man7.org/linux/man-pages/man1/pldd.1.html): 列出process使用的shared library。奇怪的是我自己用卻只有列出process的執行檔名稱而已。
* [sotruss](http://manpages.org/sotruss): 經由PLT (Procedure Linkage Table) trace shared library calls
* [sprof](http://manpages.org/sprof): share object 的profile 工具

```
bin
├── catchsegv
├── gencat
├── getconf
├── getent
├── iconv
├── ldd
├── locale
├── localedef
├── makedb
├── mtrace
├── pcprofiledump
├── pldd
├── sotruss
├── sprof
├── tzselect
└── xtrace
```


<a name="glibc_conl_sbin"></a>
### /sbin

* 除了`ldconfig`外其他的不認識

```
$ tree sbin/
sbin/
├── iconvconfig
├── ldconfig
├── nscd
├── sln
├── zdump
etc
├── ld.so.conf
└── rpc
└── zic
```

<a name="glibc_conl_etc"></a>
### /etc

很有趣，竟然有`rpc` (remote procedure call)的檔案，紀錄rpc通訊協定的資訊。

```
etc
├── ld.so.conf
└── rpc
```

<a name="glibc_conl_share"></a>
### /share

* 存放時區以及多國語言相關檔案

<a name="glibc_conl_var"></a>
### /var

跳過

<a name="glibc_conl_libexec"></a>
### /libexec

跳過

<a name="glibc_conl_inc"></a>
### /include

跳過

<a name="glibc_conl_ref"></a>
## 參考資料

* [Linux x86 Program Start Up](http://dbp-consulting.com/tutorials/debugging/linuxProgramStartup.html)
* [ELF: From The Programmer's Perspective: The .init and .fini Sections (1995)](http://l4u-00.jinr.ru/usoft/WWW/www_debian.org/Documentation/elf/elf.html)
* [Stackoverflow: What is the use of _start() in C?](https://stackoverflow.com/questions/29694564/what-is-the-use-of-start-in-c)
* [Stackoverflow: Executing init and fini](https://stackoverflow.com/questions/32700494/executing-init-and-fini)


<a name="glibc_conl_ex"></a>
## 附錄

* 編譯hello.c 囉唆資訊節錄

```
/usr/lib/gcc/x86_64-linux-gnu/7/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/7/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/7/lto-wrapper -plugin-opt=-fresolution=/tmp/ccnUW8Qj.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s -plugin-opt=-pass-through=-lc -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_s --sysroot=/ --build-id --eh-frame-hdr -m elf_x86_64 --hash-style=gnu --as-needed -dynamic-linker /lib64/ld-linux-x86-64.so.2 -pie -z now -z relro -o hello /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/Scrt1.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o -L/usr/lib/gcc/x86_64-linux-gnu/7 -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/7/../../.. /tmp/ccTxSMPW.o -lgcc --push-state --as-needed -lgcc_s --pop-state -lc -lgcc --push-state --as-needed -lgcc_s --pop-state /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o
```


* 會連結系統提供的object檔案列出如下
    * crtn.o
    * Scrt1.o
    * crti.o
    * crtendS.o
    * crtbeginS.o
