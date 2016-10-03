---
layout: post
title: "使用gdb 靜態分析C 語言名稱和參數相同函數的原始檔位置"
date: 2016-10-03 14:36:00 +0800
comments: true
categories: [Linux, C, gdb, 組裝]
---
## 前言
感謝[Scott](http://scottt.tw/)大大的資訊。

忘記那邊看過的名言：「手上有了新玩具，就會想要馬上拿來用。」

## 動機
組裝軟體的時候，有一件事很讓我困擾。那就是當整包SDK有兩個以上名稱和參數相同的函數的時候。當這種情況發生時，trace code就很麻煩，你必須要花時間釐清到底最後會使用到那一個函數。而這些config可能用下面兩種方式切換這些名稱和參數相同的函數：

1. 巨集
    * `#if`
    * `#ifdef`
2. Makefile
    * 在檔案中根據不同變數編譯不同的檔案

我以前會視情況用下面兩種方法找到該函數編譯實際使用的原始檔位置

1. 下毒藥，只要相同名字的函數都塞入`#error 可以辨別的字串`。編譯時根據錯誤訊息判斷使用哪個函數
2. gdb設斷點，runtime透過中斷函數的方式取得函數的檔案和行號

不過[Scott](http://scottt.tw/)大大的今天給的資訊讓我可以更省力的處理這個問題了。

如果只想要知道用法，看完下面指令就可以收工回家了。

```sh
gdb -ex 'file 你的執行檔或是shared library檔' -ex 'info line 要查的函數' -ex 'quit'
```

當然用`gdb`編譯時不要忘記加debug option。

## 目錄
* [測試環境](#gds_env)
* [測試檔案](#gds_files)
    * [test_same_func.c](#gds_files_1)
    * [same_func1.c](#gds_files_2)
    * [same_func2.c](#gds_files_3)
    * [Makefile](#gds_files_mk)
* [測試結果](#gds_test)
* [延伸資料](#gds_ref)


<a name="gds_env"></a>
## 測試環境
```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
```

<a name="gds_files"></a>
## 測試檔案
簡單來說，就是實作切換兩種方式同樣名稱和參數的函數：

1. 巨集
2. Makefile

所以我們會有

1. [測試檔案進入點](#gds_files_1)
2. [測試檔一](#gds_files_2)
    * 實作使用巨集`OP1`切換同樣名稱和參數
    * 和測試檔二完全一模一樣
3. [測試檔二](#gds_files_3)
    * 實作使用巨集`OP1`切換同樣名稱和參數
    * 和測試檔一完全一模一樣
4. [Makefile](#gds_files_mk)
    * 除了編譯以外，還提供兩個變數，由command line傳入
        * USE_FILE=1
            * 沒傳入時預設編譯<font color="red">**測試檔二**</font>，當該參數傳入`USE_FILE=1`時會變成編譯<font color="red">**測試檔一**</font>
        * EN_OP1=1
            * 當該參數傳入時才會開啟`OP1`巨集

<a name="gds_files_1"></a>
### test_same_func.c

* 測試檔案進入點，呼叫func1()。func1()在compile time才被決定

```c test_same_func.c
#include <stdio.h>
extern void func1(void);

int main()
{
    printf("Hello World\n");
    func1();

    return 0;
}
```

<a name="gds_files_2"></a>
### same_func1.c

* 實作使用巨集`OP1`切換同樣名稱和參數
* 和[same_func2](#gds_files_2)完全一模一樣

```c same_func1.c
#include <stdio.h>

#if OP1
void func1(void)
{
    printf("%s: %s_OP1\n", __FILE__, __PRETTY_FUNCTION__);
}
#else
void func1(void)
{
    printf("%s: %s_NOT_OP1\n", __FILE__, __PRETTY_FUNCTION__);
}
#endif
```

<a name="gds_files_3"></a>
### same_func2.c

* 實作使用巨集`OP1`切換同樣名稱和參數
* 和[same_func1](#gds_files_1)完全一模一樣


```c same_func2.c
#include <stdio.h>

#if OP1
void func1(void)
{
    printf("%s: %s_OP1\n", __FILE__, __PRETTY_FUNCTION__);
}
#else
void func1(void)
{
    printf("%s: %s_NOT_OP1\n", __FILE__, __PRETTY_FUNCTION__);
}
#endif
```

<a name="gds_files_mk"></a>
### Makefile

提供兩個變數，由command line傳入

* USE_FILE=1
    * 沒傳入時預設編譯<font color="red">**測試檔二**</font>，當該參數傳入`USE_FILE=1`時會變成編譯<font color="red">**測試檔一**</font>
* EN_OP1=1
    * 當該參數傳入時才會開啟`OP1`巨集

```makefile
CFLAGS+=-Wall -Werror -g3
TARGET=test_same_func
SRCS=test_same_func.c

ifeq ($(USE_FILE), 1)
    SRCS += same_func1.c
else
    SRCS += same_func2.c
endif

ifeq ($(EN_OP1), 1)
    CFLAGS += -DOP1=1
endif

OBJS=$(patsubst %.c, %.o, $(SRCS))

$(TARGET): $(OBJS)
	$(CC) -o $@ $^
	gdb -ex 'file $@' -ex 'info line func1' -ex 'quit'

%.o: %.c
	$(CC) $(CFLAGS) -c $^

clean:
	rm *.o *~ $(TARGET) -f
```
<a name="gds_test"></a>
## 測試結果

前面提過，`func1()`的實作受兩個變數影響，分別為

1. 巨集`OP1`是否有設定
    * 只有`OP1` 被設定**並且**非零時才會進入`OP1`的`func1()`
2. Makefile變數`USE_FILE`是否有設定
    * 只有`USE_FILE`為1的時候才會使用`same_func1.c`，其他情形都編譯`same_func2.c`

所以我們make 指令有下面四種變化

1. [巨集`OP1`和`USE_FILE`都沒設定]()
2. [設定巨集`OP1`，`USE_FILE`沒設定]()
3. [巨集`OP1`沒設定，但是設定`USE_FILE`]()
4. [全部設定巨集`OP1`和`USE_FILE`]()


<a name=""></a>
### 巨集`OP1`和`USE_FILE`都沒設定

gdb驗證結果的確是

1. 編譯`same_func2.c`
2. 使用非`OP1`版本的`func1()` 

```sh
$ make clean 
default settings: OP1 disable and use same_func1.c
rm *.o *~ test_same_func -f

$ make
cc -Wall -Werror -g3 -c test_same_func.c
cc -Wall -Werror -g3 -c same_func2.c
cc -o test_same_func test_same_func.o same_func2.o

$ gdb -ex 'file test_same_func' -ex 'info line func1' -ex 'quit'
...
Line 11 of "same_func2.c" starts at address 0x400597 <func1> and ends at 0x40059b <func1+4>.
Hello World
same_func2.c: func1_NOT_OP1
```

<a name=""></a>
### 設定巨集`OP1`，`USE_FILE`沒設定

gdb驗證結果的確是

1. 編譯`same_func2.c`
2. 使用`OP1`版本的`func1()` 

```sh
$ make clean
rm *.o *~ test_same_func -f

$ make EN_OP1=1
cc -Wall -Werror -g3 -DOP1=1 -c test_same_func.c
cc -Wall -Werror -g3 -DOP1=1 -c same_func2.c
cc -o test_same_func test_same_func.o same_func2.o

$ gdb -ex 'file test_same_func' -ex 'info line func1' -ex 'quit'
...
Line 5 of "same_func2.c" starts at address 0x400597 <func1> and ends at 0x40059b <func1+4>.
Hello World
same_func2.c: func1_OP1
```

<a name=""></a>
### 巨集`OP1`沒設定，但是設定`USE_FILE` 

gdb驗證結果的確是

1. 編譯`same_func1.c`
2. 使用非`OP1`版本的`func1()` 

```sh
$ make clean
rm *.o *~ test_same_func -f

$ make USE_FILE=1
cc -Wall -Werror -g3 -c test_same_func.c
cc -Wall -Werror -g3 -c same_func1.c
cc -o test_same_func test_same_func.o same_func1.o

$ gdb -ex 'file test_same_func' -ex 'info line func1' -ex 'quit'
...
Line 11 of "same_func1.c" starts at address 0x400597 <func1> and ends at 0x40059b <func1+4>.
Hello World
same_func1.c: func1_NOT_OP1
```

<a name=""></a>
### 全部設定巨集`OP1`和`USE_FILE`

gdb驗證結果的確是

1. 編譯`same_func1.c`
2. 使用`OP1`版本的`func1()` 


```sh
$ make clean
rm *.o *~ test_same_func -f

$ make EN_OP1=1 USE_FILE=1
cc -Wall -Werror -g3 -DOP1=1 -c test_same_func.c
cc -Wall -Werror -g3 -DOP1=1 -c same_func1.c
cc -o test_same_func test_same_func.o same_func1.o

$ gdb -ex 'file test_same_func' -ex 'info line func1' -ex 'quit'
...
Line 5 of "same_func1.c" starts at address 0x400597 <func1> and ends at 0x40059b <func1+4>.
Hello World
same_func1.c: func1_OP1
```

<a name="gds_ref"></a>
## 延伸資料

* [Using GDB to Help You Read Linux Kernel Code without Running It](https://docs.google.com/document/d/1w1nPmCLpeRN3kEYglzHwozhWXY0ddT9oe6JCZocsPNE/edit)
    * [Scott](http://scottt.tw/)大大的文件，描述使用gdb 靜態分析Linux kernel的技巧
