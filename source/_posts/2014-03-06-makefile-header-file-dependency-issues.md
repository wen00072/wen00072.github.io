---
layout: post
title: 'Makefile header file dependency問題'
date: 2014-03-06 01:12
comments: true
categories: [make]
---
自幹Makefile的時候常常忘記把header files加到Prerequisite然後編譯的時候就發生非預期的錯誤。之前整理[GNU: The C Preprocess導讀](http://wen00072-blog.logdown.com/posts/146624-talk-about-c-macros)有看到gcc/cpp(C Preprocessor)的`-MMD`參數，但是還不清楚怎麼使用。直到看到[Using g++ with -MMD in makefile to automatically generate dependencies](http://stackoverflow.com/questions/11855386/using-g-with-mmd-in-makefile-to-automatically-generate-dependencies)才知道怎麼玩。手癢自己也來弄一個驗證一下。

測試檔案及程式碼如下
不想看code只要知道每個檔案都有參考到某個自訂的header file就好了。

```c liba.h
#ifndef LIBA_H_2013
#define LIBA_H_2013
void test_liba(void);
void from_liba(void);
#endif
```

```c liba.c
#include "libb.h"
#include <stdio.h>

void test_liba(void)
{
    from_libb();
}

void from_liba()
{
    printf("%s\n", __PRETTY_FUNCTION__);
}

```

```c libb.h
#ifndef LIBB_H_2013
#define LIBB_H_2013
void test_libb(void);
void from_libb(void);
#endif
```

```c libb.c  
#include "liba.h"
#include <stdio.h>

void test_libb(void)
{
    from_liba();
}

void from_libb()
{
    printf("%s\n", __PRETTY_FUNCTION__);
}
```

```c test.c
#include "libb.h"
#include "liba.h"
#include <stdlib.h>
int main(void)
{
    test_liba();
    test_libb();

    return 0;
}
```

先來複習一下`-MMD`的效果

```text 執行-MMD 結果
$ gcc -MMD -c test.c
$ ls test.*
test.c  test.d  test.o
$ cat test.d 
test.o: test.c libb.h liba.h
```

剩下就是想辦法讓Makefile吃到*.d的內容，因此Makefile長的就像這樣

```makefile Makefile
CFLAGS=-g -MMD

SRCS= test.c liba.c libb.c
OBJS = $(patsubst %.c, %.o, $(SRCS))
DEPS = $(patsubst %.o, %.d, $(OBJS))

TARGET=test

$(TARGET): $(OBJS)
	gcc -g -o $@ $^

clean:
	rm -rf *.o *.a *~ $(TARGET) $(OBJS) $(DEPS)

-include $(DEPS)
```

Makefile說明:

* `patsubst`
	* Makefile 提供的函數，用在pattern 替換，所以上面做了下列的代換
  	* .c轉成.o
    * .o轉成.d
* `$@`
	* Makefile的內建巨集，代表target
* `$^`
	* Makefile的內建巨集，代表prerequisite 
* `-`
	* 告訴make 忽略失敗，trick就在這邊
 
Trick就是上面提到的`-`，第一次make時一定*.d檔案不存在，因此不使用`-`就一定編不過。第一次編過之後因為*.d已經產生，所以header file的dependecy也被處理好了。

其實上面的Makefile可以做更多改進，當project檔案數量大到某種程度就可以感覺改進的效果，例如

* 把*.o 和*.d 指定產生到特定目錄，而不是散落各處
* 加上distclean，只有在這種情況下才去刪除*.d的檔案
