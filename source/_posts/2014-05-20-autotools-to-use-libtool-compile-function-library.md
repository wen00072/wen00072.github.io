---
layout: post
title: '在Autotools使用Libtool編譯函式庫'
date: 2014-05-20 06:46
comments: true
categories: 
---
前面討論了[Autotools](http://wen00072-blog.logdown.com/posts/198783-study-on-gnu-build-system-autotools)和[Libtool](http://wen00072-blog.logdown.com/posts/199470-study-on-the-libtool)，今天就來合體一下。其實只要把原本[Autotools](http://wen00072-blog.logdown.com/posts/198783-study-on-gnu-build-system-autotools)範例做少部份的修改就可以了。

目錄：

* [前情提要](#preview)
* [更動說明](#ex)
	* [configure.ac](#ex-conf)
	* [libs/Makefile.am](#ex-libs)
	* [src/Makefile.am](#ex-src)
  * [執行結果](#ex-result)
* [參考資料](#ref)

---
<a name = "preview"></a>
## 前情提要

先複習一下

* Autotools
	* 目的是協助處理不同平台相依性的問題。
  * Autotools 產生
    * scripts用來偵測環境，最終產生平台相依的Makefile。
    * config.h存放不同設定。
  * 使用者最低限度的需要自己寫/更改
      * 每個需要目錄中的`Makefile.am` 給automake產生Makefile.in。
      * `configure.ac`給autotools中工具使用。
      
* Libtool
  *	協助處理不同平台編譯以及link時相依性的問題。 (我自己想的，可能不精確。)
	* 是一個script
  * 編譯會產生
    * 原始檔衍生出來的兩個object 檔案，一個為標準object檔案，另外為[PIC](http://en.wikipedia.org/wiki/Position-independent_code)版本
      * *.lo檔案描述該原始檔的metadata
    * library
      * *.la檔案描述library的metadata

---
<a name = "ex"></a>
## 更動說明

<a name = "ex-conf"></a>
### configure.ac
更動如下：

* 拿掉`AC_PROG_RANLIB`，這邊libtool會幫忙處理
* 加入`LT_INIT`告訴autotool會使用libtool

```text configure.ac.diff
diff --git a/configure.ac b/configure.ac
index 7763f37..edec020 100644
--- a/configure.ac
+++ b/configure.ac
@@ -6,8 +6,8 @@ AC_INIT([Test_Autotools], [0], [test])
 AM_INIT_AUTOMAKE([foreign -Wall -Werror])
 AC_CONFIG_HEADERS([config.h])
 
-# Use static libary
-AC_PROG_RANLIB
+# USE libtool
+LT_INIT
 
 # Makefiles
 AC_CONFIG_FILES([Makefile src/Makefile libs/Makefile])
```

---
<a name = "ex-libs"></a>
### libs/Makefile.am
更動如下：

* 把`LIBRARIES`改成`LTLIBRARIES`
* 產生的目標為從*.a改成*.la

```text libs/Makefile.am.diff
diff --git a/libs/Makefile.am b/libs/Makefile.am
index f173ac6..230fa34 100644
--- a/libs/Makefile.am
+++ b/libs/Makefile.am
@@ -1,7 +1,8 @@
 AM_CFLAGS = -I../include
-lib_LIBRARIES = liba.a libb.a
-liba_a_SOURCES = liba.c
-libb_a_SOURCES = libb.c
+lib_LTLIBRARIES = liba.la libb.la
+liba_la_SOURCES = liba.c
+libb_la_SOURCES = libb.c
+
 
 include_HEADERS = ../include/liba.h ../include/libb.h
```

---
<a name = "ex-src"></a>
### src/Makefile.am
更動如下：

* 指定link *.la
```text src/Makefile.am.diff
diff --git a/src/Makefile.am b/src/Makefile.am
index 75a5199..b8b29bf 100644
--- a/src/Makefile.am
+++ b/src/Makefile.am
@@ -1,4 +1,4 @@
-LDADD = ../libs/liba.a ../libs/libb.a
+LDADD = ../libs/liba.la ../libs/libb.la
 AM_CFLAGS = -I../include
 
 bin_PROGRAMS = test
```

---
<a name = "ex-result"></a>
### 執行結果
簡單來說，autotool使用libtool產生*.a和*.so檔案

* 執行
```
make distclean; autoreconf --install; ./configure --prefix=/tmp/test; make; make install
```

* 先看/tmp/test安裝的檔案tree view
```
├── bin
│   └── test
├── include
│   ├── liba.h
│   └── libb.h
└── lib
    ├── liba.a
    ├── liba.la
    ├── liba.so -> liba.so.0.0.0
    ├── liba.so.0 -> liba.so.0.0.0
    ├── liba.so.0.0.0
    ├── libb.a
    ├── libb.la
    ├── libb.so -> libb.so.0.0.0
    ├── libb.so.0 -> libb.so.0.0.0
    └── libb.so.0.0.0
```

* make library 結果節錄
```
make[2]: Entering directory `libs'
/bin/bash ../libtool --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I. -I..    -I../include -g -O2 -MT liba.lo -MD -MP -MF .deps/liba.Tpo -c -o liba.lo liba.c
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I.. -I../include -g -O2 -MT liba.lo -MD -MP -MF .deps/liba.Tpo -c liba.c  -fPIC -DPIC -o .libs/liba.o
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I.. -I../include -g -O2 -MT liba.lo -MD -MP -MF .deps/liba.Tpo -c liba.c -o liba.o >/dev/null 2>&1
mv -f .deps/liba.Tpo .deps/liba.Plo
/bin/bash ../libtool --tag=CC   --mode=link gcc -I../include -g -O2   -o liba.la -rpath /usr/local/lib liba.lo  
libtool: link: gcc -shared  -fPIC -DPIC  .libs/liba.o    -O2   -Wl,-soname -Wl,liba.so.0 -o .libs/liba.so.0.0.0
libtool: link: (cd ".libs" && rm -f "liba.so.0" && ln -s "liba.so.0.0.0" "liba.so.0")
libtool: link: (cd ".libs" && rm -f "liba.so" && ln -s "liba.so.0.0.0" "liba.so")
libtool: link: ar cru .libs/liba.a  liba.o
libtool: link: ranlib .libs/liba.a
libtool: link: ( cd ".libs" && rm -f "liba.la" && ln -s "../liba.la" "liba.la" )
```

* 產生執行檔結果
```
gcc -DHAVE_CONFIG_H -I. -I..    -I../include -g -O2 -MT test.o -MD -MP -MF .deps/test.Tpo -c -o test.o test.c
mv -f .deps/test.Tpo .deps/test.Po
/bin/bash ../libtool --tag=CC   --mode=link gcc -I../include -g -O2   -o test test.o ../libs/liba.la ../libs/libb.la 
libtool: link: gcc -I../include -g -O2 -o .libs/test test.o  ../libs/.libs/liba.so ../libs/.libs/libb.so
```

* ls libs/.libs
```
$ ls libs/.libs/
liba.a  liba.la  liba.lai  liba.o  liba.so  liba.so.0  liba.so.0.0.0  libb.a  libb.la  libb.lai  libb.o  libb.so  libb.so.0  libb.so.0.0.0
```

---
<a name = "ref"></a>
## 參考資料

* [How to create a shared library (.so) in an automake script?](http://stackoverflow.com/questions/8916425/how-to-create-a-shared-library-so-in-an-automake-script)
