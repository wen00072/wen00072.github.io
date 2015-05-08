---
layout: post
title: 'Makefile 自動將產生的檔案指定到特定目錄'
date: 2014-03-12 13:35
comments: true
categories: [make]
---
**Mar/13/2014已更正錯誤：原本錯誤分析請看[這邊](http://wen00072-blog.logdown.com/posts/184950-makefile-pattern-rules-little-attention)**

[之前](http://wen00072-blog.logdown.com/posts/183770-makefile-header-file-dependency-issues)分享了Makefile header file dependency問題解決方式之一，裏面提到把*.o 和*.d 指定產生到特定目錄，而不是散落各處的改進方向。過了幾天網路上看到[別人的解法](https://github.com/southernbear/rtenv/blob/master/Makefile)，手癢自己也來玩一下。

[範例程式細節在這邊](http://wen00072-blog.logdown.com/posts/203304-dry-test-file-template)，不想看code只要知道每個檔案都有參考到某個自訂的header file就好了。

<a name="tree"></a>

這次加碼一下，把檔案散落在不同目錄如下
```text tree view
$ tree
.
├── include
│   ├── liba.h
│   └── libb.h
├── libs
│   ├── liba.c
│   └── libb.c
├── Makefile
└── test.c
```

Makefile更改如下

```makefile Makefile
CFLAGS=-g -MMD -I ./include

# binaries
LIB_SRC=$(shell ls libs/*.c)
SRCS= $(LIB_SRC) test.c

OBJS = $(patsubst %.c, %.o, $(SRCS))

# Output
OUT_DIR = build
OUT_OBJS=$(addprefix $(OUT_DIR)/, $(OBJS))
DEPS = $(patsubst %.o, %.d, $(OUT_OBJS))

# Build Rules
TARGET=test

$(OUT_DIR)/$(TARGET): $(OUT_OBJS)
	$(CC) -o $@ $^

$(OUT_DIR)/%.o:%.c
	mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $< -o $@


clean:
	rm -rf $(OUT_DIR)

-include $(DEPS)
```
整個Makefile主要是

1. 找出所有要編譯的c檔案並放在變數`SRCS`
2. 把變數`SRC`中的*.c換成*.o並存到變數`OBJS`
3. 再把變數`OBJS`裏面的字串每一個都加上`build/`的prefix並存到變數`OUT_OBJS`
4. 再把`OUT_OBJS`中的*.o換成*.d並存到變數`DEP`
5. 編譯*.c到*.o的時候，先產生對應於C code的目錄並將*.o *.d存到裏面

參數說明如下

* `-MMD`請參考[這邊](http://wen00072-blog.logdown.com/posts/183770-makefile-header-file-dependency-issues)
* `shell`請參考[這邊](http://wen00072-blog.logdown.com/posts/174562-makefile-with-command-results-examples)
* `patsubst`請參考[這邊](http://wen00072-blog.logdown.com/posts/183770-makefile-header-file-dependency-issues)
* `addprefix`，照字面就是加上prefix
* `dir`，取得檔案的dir
* `$@`
	* Makefile的內建巨集，代表target
* `$<`
	* Makefile的內建巨集，代表第一個prerequisite
* `-`
	*告訴make 忽略失敗
* `%.o:%.c`
	* [Pattern Rules](http://www.gnu.org/software/make/manual/make.html#Pattern-Rules)
  * %表示萬用字元，而prerequisite的%會和target的% match到的相同。所以就是說所有*.o結尾的檔案depends on *.c檔案時要做下面的事
  * 舉例來說`$(OUT_DIR)/%.o:%.c`，表示build/xxx/file1.o 會依賴對應到的xxx/file1.c
* 其實重點是`mkdir -p $(dir $@)`會自動在build產生對應的目錄存放*.d和*.o

跑起來結果如下
```text result
$ make
mkdir -p build/libs/
cc -g -MMD -I ./include -c libs/liba.c -o build/libs/liba.o
mkdir -p build/libs/
cc -g -MMD -I ./include -c libs/libb.c -o build/libs/libb.o
mkdir -p build/
cc -g -MMD -I ./include -c test.c -o build/test.o
cc -o build/test build/libs/liba.o build/libs/libb.o build/test.o

$ tree
.
├── build
│   ├── libs
│   │   ├── liba.d
│   │   ├── liba.o
│   │   ├── libb.d
│   │   └── libb.o
│   ├── test
│   ├── test.d
│   └── test.o
├── include
│   ├── liba.h
│   └── libb.h
├── libs
│   ├── liba.c
│   └── libb.c
├── Makefile
└── test.c

```

## 參考資料

* [southernbear / rtenv: Makefile](https://github.com/southernbear/rtenv/blob/master/Makefile)
* [GNU make: 8.2 Functions for String Substitution and Analysis](http://www.gnu.org/software/make/manual/make.html#Text-Functions)
* [GNU make: 10.5 Defining and Redefining Pattern Rules](http://www.gnu.org/software/make/manual/make.html#Pattern-Rules)
* [GNU make: 10.5.3 Automatic Variables](http://www.gnu.org/software/make/manual/make.html#Automatic-Variables)
