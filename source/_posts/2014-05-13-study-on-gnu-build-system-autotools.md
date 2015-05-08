---
layout: post
title: 'GNU Build System: Autotools 初探'
date: 2014-05-13 10:43
comments: true
categories: [Linux, autotools]
---
身為組裝工，常常執行下面的指令

* ./configure
* make
* 組裝

用了那麼久從來沒有思考過這是什麼樣的東西，慚愧之餘趕快惡補一下。

---
## 目錄
* [Overview](#overview)
	* [Autotool 元件簡介](#at-overview)
    * [Autoconf](#at-aconf)
    * [Automake](#at-amake)
    * [Libtool](#at-ltool)
  * [合體](#at-integrate)
* [範例](#auto_ex)
	* [測試環境](#auto_ex_env)
	* [測試程式](#auto_ex_prgs)
  * [導入Autotools](#auto_ex_start)
    * [configure.ac](#auto_ex_configac)
    * [Makefile.am](#auto_ex_maam)
    * [產生`configure`並編譯](#auto_ex_config)
* [參考資料及資源](#ref)

---
<a name="overview"></a>
## Overview
### 緣由：
雖然C語言號稱高移植性，然而遺憾的是因為System call, library等介面早期並沒有規範。這造成使用C語言開發多平台軟體的時候需要針對不同的平台做許多的檢查。後來GNU針對這部份提出了autotool工具減輕開發的負擔。

如果懶得看元件簡介，請直接看[Overoview結論](#at-integrate)

---
<a name="at-overview"></a>
### Autotool 元件
從Wikipedia可以看到[GNU Build system](http://en.wikipedia.org/wiki/GNU_build_system)的項目中有提到三個元件：
* [Autoconf](#at-aconf)
* [Automake](#at-amake)
* [Libtool](#at-ltool)

---
<a name="at-aconf"></a>
#### [Autoconf - GNU Project - Free Software Foundation (FSF)](http://www.gnu.org/software/autoconf)
從[這邊](http://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.69/html_node/Making-configure-Scripts.html#Making-configure-Scripts)可以看到`configure`負責產生

* Makefile
	* 讓你編譯程式
* C option用的header file (optional)
	* `config.h`
* `configure.status`
	* 重新自動產生上面的資料
* `configure.cache` (optional)
	* cache之前configure偵測的系統結果

套件中有幾個程式

* `autoscan`
	* 產生`configure.ac`
* `autoconf`
	* 從`configure.ac`中產生`configure`
  * 為何是`或`呢？這和autoconf的版本有關
* `autoheader`
	* 從`configure.ac`產生`config.h.in`
* `ifnames`
	* 掃描C原始碼，抽出ifdef的資訊

---
<a name="at-amake"></a>
#### [Automake - GNU Project - Free Software Foundation (FSF)](http://www.gnu.org/software/automake/)

套件中有兩個程式

* automake
	* 從`Makefile.am`中產生`Makefile.in`，讓`configure`執行時可以產生`Makefile`
* aclocal
	* 從`configure.ac`或`configure.in`產生`aclocal.m4`

---
<a name="at-ltool"></a>
#### [GNU Libtool - The GNU Portable Library Tool](http://www.gnu.org/software/libtool/)

* 提供抽象化的介面處理函式庫的平台差異問題，本篇暫不討論。

---
另外GNU 其他針對移植性輔助的套件有

* [GNU gettext](http://en.wikipedia.org/wiki/Gettext)
	* i18n套件，協助專案中的各國語言訊息開發。
* [pkg-config](http://en.wikipedia.org/wiki/Pkg-config)
	* 提供開發時的函式庫資訊，開發者不需要知道函式庫的路徑和旗標，直接問pkg-config就好。
* [GCC](http://en.wikipedia.org/wiki/GNU_Compiler_Collection)
	* 全名是`GNU Compiler Collection`

---
<a name="at-integrate"></a>
### 合體
從Wikipedia的[圖：Autoconf-automake-process](http://en.wikipedia.org/wiki/File:Autoconf-automake-process.svg)解釋的非常清楚。

回到最前面，我們可以觀察到autotool的目的就是

* **產生平台上可以編譯的環境**

為了達到這樣的目的，系統需要做到下面的功能

* 檢查平台環境
* 產生Makefile

因此，開發者最少要告訴autotools **所需平台環境** 和 **產生Makefile** 這兩項資訊。這也是`configure.ac`和`Makefile.am`存在的目的。也就是說，開發者需要自行設定`configure.ac`和`Makefile.am`。

而這些錯綜複雜的關係可以描述如下

* autotool先產生`configure.ac` (透過`autoscan`或是自幹）
* `autoreconf`吃`configure.ac`和`Makefile.am`產生`confgure`, `config.h.in`, 和`Makefile.in`
* autoreconf細節
  * 吃`configure.ac`，呼叫`aclocal`產生`aclocal.m4`
  * aclocal是給autotools中自訂專案檢查編譯巨集用。
	* 吃`configure.ac`和`aclocal.m4`，呼叫`autoheader`產生`config.h.in`
	* 吃`configure.ac`和`aclocal.m4`，呼叫`autoconf`產生`configure`
	* 吃`configure.ac`和`aclocal.m4`和各處的`Makefile.am`，呼叫`automake`產生`Makefile.in`

---
<a name="auto_ex"></a>
## 範例

---
<a name="auto_ex_env"></a>
### 測試環境

* Ubuntu 12.04.4

---
<a name="auto_ex_prgs"></a>
### 測試程式

[範例程式細節在這邊](http://wen00072-blog.logdown.com/posts/203304-dry-test-file-template)，檔案各別重新分配到`src`, `include`, `libs`這三個目錄。不想看code只要知道每個檔案都有參考到某個自訂的header file就好了。

```text 測試程式樹狀架構
├── include
│   ├── liba.h
│   └── libb.h
├── libs
│   ├── liba.c
│   └── libb.c
└── src
    └── test.c
```

---
<a name="auto_ex_start"></a>
### 導入Autotools
簡述一下流程

* 使用`autoscan`產生`configure.ac`範本
* 手動更改`configure.ac`
* 寫`Makefile.am`，根目錄以及需要編譯的地方都要一份
* `autoreconf --install`產生檔案
  * 替代方案：執行下面程式
    * `aclocal`
    * `autoheader`
    * `automake --add-missing`
    * `autoconf`
* `configure`
* `make`
* `make install`安裝或`make dist`打包

---
<a name="auto_ex_configac"></a>
#### configure.ac
執行`autoscan`後會產生`configure.scan`檔案，把這個檔案rename成`configure.ac`

```makefile configure.ac最初範本
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.68])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([libs/libb.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC

# Checks for libraries.

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h])

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_FUNC_MALLOC

AC_CONFIG_FILES([Makefile
                 libs/Makefile
                 src/Makefile])
AC_OUTPUT
```

接下來就是更動`configure.ac`，更動完和原本的差別如下：
```text configure.scan和configure.ac 差別
5,6c5,6
< AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
< AC_CONFIG_SRCDIR([libs/libb.c])
---
> AC_INIT([Test_Autotools], [0], [test])
> AM_INIT_AUTOMAKE([foreign -Wall -Werror])
8a9,14
> # Use static libary
> AC_PROG_RANLIB
> 
```
簡單說明如下

* 更改套件版本資訊
* 刪除`AC_CONFIG_SRCDIR`
* 設定automake
	* `foreign`表示這不是標準的GNU Coding Standard，因此不會檢查額外的檔案如`NEWS`, `README`, `ChangeLog`等。
  * 共用編譯參數
* `AC_PROG_RANLIB`表示要使用static library

---
<a name="auto_ex_maam"></a>
#### Makefile.am
前面提到，每個目錄都要有一個`Makefile.am`。關於`Makefile.am`語法，可以參考 [Alexandre Duret Lutz: Autotools Tutorial](https://www.lrde.epita.fr/~adl/autotools.html)大略了解。更詳細的部份可以看[Automake手冊：8.3 Building a Shared Library](http://www.gnu.org/software/automake/manual/html_node/A-Shared-Library.html#A-Shared-Library)

範例中`Makefile.am`如下：

* ./Makefile.am：　基本上就是依照列出的順序編譯

```makefile ./Makefile.am
SUBDIRS = libs src
```

* libs/Makefile.am

```makefile libs/Makefile.am
AM_CFLAGS = -I../include
lib_LIBRARIES = liba.a libb.a
liba_a_SOURCES = liba.c
libb_a_SOURCES = libb.c

include_HEADERS = ../include/liba.h ../include/libb.h
```
最上面形式就是`要裝到目錄`_`Keyword`，所以表示我們要產生`liba.a`和`libb.a`，安裝時放在`/usr/local/lib`下面、指定library由哪些原始程式檔案組成、同時要把header files放在`/usr/local/include`下。(configure沒指定預設prefix為/usr/local)

* src/Makefile.am

```makefile src/Makefile.am
LDADD = ../libs/liba.a ../libs/libb.a
AM_CFLAGS = -I../include

bin_PROGRAMS = test
test_SOURCES = test.c
```

一樣的形式：`要裝到目錄`_`Keyword`。這邊宣告要安裝到`/usr/loca/bin`、要link `liba.a`和`libb.a`、並且5指定執行檔的原始程式檔案。

---
經過上面的處理，我們多了三個`Makefile.am`和`configure.ac`以及一些暫存檔案。新的樹狀目錄列表如下：
```text autoscan後測試程式樹狀架構
.
├── autom4te.cache
│   ├── output.0
│   ├── requests
│   └── traces.0
├── autoscan.log
├── configure.ac
├── include
│   ├── liba.h
│   └── libb.h
├── libs
│   ├── liba.c
│   ├── libb.c
│   └── Makefile.am
├── Makefile.am
└── src
    ├── Makefile.am
    └── test.c
```

---
<a name="auto_ex_config"></a>
#### 產生`configure`並編譯

如前所述，直接使用`autoreconf --install`就可以了
```text autoreconf --install以及之後的測試程式樹狀架構
$ autoreconf --install  
configure.ac:6: installing `./install-sh'
configure.ac:6: installing `./missing'
libs/Makefile.am: installing `./depcomp'

.
├── aclocal.m4
├── autom4te.cache
│   ├── output.0
│   ├── output.1
│   ├── output.2
│   ├── requests
│   ├── traces.0
│   ├── traces.1
│   └── traces.2
├── autoscan.log
├── config.h.in
├── configure
├── configure.ac
├── depcomp
├── include
│   ├── liba.h
│   └── libb.h
├── install-sh
├── libs
│   ├── liba.c
│   ├── libb.c
│   ├── Makefile.am
│   └── Makefile.in
├── Makefile.am
├── Makefile.in
├── missing
└── src
    ├── Makefile.am
    ├── Makefile.in
    └── test.c
```

./configure結果節錄如下
```
$ ./configure --prefix=/tmp/build
checking for a BSD-compatible install... /usr/bin/install -c
...
checking for GNU libc compatible malloc... yes
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/Makefile
config.status: creating libs/Makefile
config.status: creating config.h
config.status: config.h is unchanged
config.status: executing depfiles commands
```
幾點要注意的：

* `config.status`被產生，並且被執行時生出`config.h`以及各目錄的的`Makefile`
* 這邊指定prefix為`/tmp/build`，因此`make install`可以看到被安裝目錄樹狀結構如下：

```text 安裝目錄樹狀結構
.
├── bin
│   └── test
├── include
│   ├── liba.h
│   └── libb.h
└── lib
    ├── liba.a
    └── libb.a
```


---
<a name="ref"></a>
## 參考資料及資源
強烈推荐 [Alexandre Duret Lutz: Autotools Tutorial](https://www.lrde.epita.fr/~adl/autotools.html)，主要的想法和資料都是從這邊出來。而且他寫的更加詳細、更加易懂。

* FSF官方資料:
  * [Autoconf - GNU Project - Free Software Foundation (FSF)](http://www.gnu.org/software/autoconf)
  * [Automake - GNU Project - Free Software Foundation (FSF)](http://www.gnu.org/software/automake/)
  * [GNU Libtool - The GNU Portable Library Tool](http://www.gnu.org/software/libtool/)
* [Wikipedia: GNU build system](http://en.wikipedia.org/wiki/GNU_build_system)
* [Wikipedia: Autoconf](http://en.wikipedia.org/wiki/Autoconf)
* [陳雍穆: automake](http://netlab.cse.yzu.edu.tw/~armor/columns/automake/automake.htm)
* [$4: GNU Build System (aka Autotools)](http://fourdollars.github.io/autotools/#1)
* [石頭閒語: Hello Autoconf, the GNU Build System](http://blog.roodo.com/rocksaying/archives/12687975.html)
* [GNU autoconf (automake) "Hello World" step-by-step example  ](http://www.xinotes.net/notes/note/1824/)
* [Alexandre Duret Lutz: Autotools Tutorial](https://www.lrde.epita.fr/~adl/autotools.html)
* [Autotools Mythbuster](https://www.flameeyes.eu/autotools-mythbuster/index.html)
* [今天的 Tetralet 又在唧唧喳喳了: 利用 Autotools 來建立 Makefile 檔案](http://tetralet.luna.com.tw/?op=ViewArticle&articleId=200&blogId=1)
