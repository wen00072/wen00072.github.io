---
layout: post
title: 'DRY: 測試檔案Template'
date: 2014-06-07 04:05
comments: true
categories: 
---
**本頁存在的目的單純是提供其他網頁參考同樣的測試程式使用，不知道算不算是一種metadata?**

[DRY: Don't Repeat Your Self](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)，字面上的解釋就是不要一直做重複的事。

由於前面一系列的文章都會用到同樣的測試程式碼，每次都要剪下貼上。加上之後打算整理的資料還是會用到這些測試程式，因此將它們獨立出來。

最原始的出處：[Makefile header file dependency問題](http://wen00072-blog.logdown.com/posts/183770-makefile-header-file-dependency-issues)。這些測試程式的主體是liba.c和libb.c兩個檔案。liba.c提供兩個API

* from_liba()
	* 印出呼叫的訊息
* test_liba()
	* 呼叫lib.c中的from_libb()

libb.c的API只是把上面的API中的liba改成libb而已。

## liba.h

```c liba.h
#ifndef LIBA_H_2013
#define LIBA_H_2013
void test_liba(void);
void from_liba(void);
#endif
```

## liba.c

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

## libb.h

```c libb.h
#ifndef LIBB_H_2013
#define LIBB_H_2013
void test_libb(void);
void from_libb(void);
#endif
```

## libb.c

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

## test.c
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
