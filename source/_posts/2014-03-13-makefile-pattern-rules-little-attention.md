---
layout: post
title: 'Makefile的Pattern Rules小小注意事項'
date: 2014-03-13 07:15
comments: true
categories: 
---
在[Makefile 自動將產生的檔案指定到特定目錄](http://wen00072-blog.logdown.com/posts/184830-makefile-automatically-assign-a-file-to-a-specific-directory)中，我犯了一個錯誤，造成更動header檔案發生錯誤(原文已修復)。

狀況如下

```text Error log
$ make
mkdir -p build/libs/
cc -g -MMD -I ./include -c libs/liba.c -o build/libs/liba.o
mkdir -p build/libs/
cc -g -MMD -I ./include -c libs/libb.c -o build/libs/libb.o
mkdir -p build/
cc -g -MMD -I ./include -c test.c -o build/test.o
cc -o build/test build/libs/liba.o build/libs/libb.o build/test.o

$ make
make: `build/test' is up to date.

$ touch include/*

$ make
mkdir -p build/libs/
cc -g -MMD -I ./include -c libs/liba.c include/libb.h -o build/libs/liba.o
cc: fatal error: cannot specify -o with -c, -S or -E with multiple files
compilation terminated.
make: *** [build/libs/liba.o] Error 4
```

從錯誤中可以看到，明明在編liba.c code怎麼會有libb.h來亂呢？請再看下面的分析。

有問題的Makefile如下

```makefile 有問題的Makefile
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
	$(CC) $(CFLAGS) -c $^ -o $@

clean:
	rm -rf $(OUT_DIR)

-include $(DEPS)

```

問題出在`$(CC) $(CFLAGS) -c $^ -o $@`，再次回顧一下
<a name="dep"></a>

* -MMD效果
```
$ cat build/libs/liba.d 
build/libs/liba.o: libs/liba.c include/libb.h
```

而在Makefile中我們會使用`-include $(DEPS)` include 這些*.d檔案，所以更改Makefile部份敘述。
```makefile
$(OUT_DIR)/%.o:%.c
	mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c $^ -o $@
```

第一次make後，當header改變, 有的prerequisite會更動，進而觸發Pattern Rule，但是`$^`是所有的prerequisite，所以$(CC)會把所有的prerequisite全部帶入造成錯誤。已[這邊](#dep)的例子，prerequisite就是`libs/liba.c`和`include/libb.h`了

解法很簡單，把`$^`換成`$<`就可以了。而`$<`是什麼呢？就是第一個prerequisite，對照[這邊](#dep)可以看到，第一個就是第一個prerequisite就是`libs/liba.c`檔案了。

## 參考資料

* [GNU make: 10.5 Defining and Redefining Pattern Rules](http://www.gnu.org/software/make/manual/make.html#Pattern-Rules)
* [GNU make: 10.5.3 Automatic Variables](http://www.gnu.org/software/make/manual/make.html#Automatic-Variables)
