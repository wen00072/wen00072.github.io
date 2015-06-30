---
layout: post
title: 'Linux中使用C語言載入data object 檔案資料 (續）'
date: 2014-04-16 02:01
comments: true
categories: [Linux, C, bintuils]
---
## 動機
[前面](http://wen00072.github.io/blog/2014/04/15/loads-the-data-object-using-the-c-language-archives-data-in-linux)討論了Linux中使用C語言載入data object 檔案資料，裏面提到資料轉成object，裏面會有三個symbol: `_binary_objfile_start`, `_binary_objfile_end`, `_binary_objfile_size`。

## Object symbol
`man objcopy`找`_size`可以看到
```text
       -B bfdarch
       --binary-architecture=bfdarch
           Useful when transforming a architecture-less input file into an object file.  In this case the output architecture can be set to bfdarch.  This
           option will be ignored if the input file has a known bfdarch.  You can access this binary data inside a program by referencing the special
           symbols that are created by the conversion process.  These symbols are called _binary_objfile_start, _binary_objfile_end and
           _binary_objfile_size.  e.g. you can transform a picture file into an object file and then access it in your code using these symbols.
```

## 測試: Segmentation fault版
簡單來說，和平台無關的檔案轉成obj檔時，可以透過`_binary_objfile_start`, `_binary_objfile_end`, `_binary_objfile_size`去存取資料。所以我再修改了一下測試程式，印出這些symbol 的內容


```c test_obj.c
#include <stdio.h>
#include <string.h>

#define MAX_CHARS (32)

extern char _binary_my_data_txt_start;
extern char _binary_my_data_txt_end;
extern int _binary_my_data_txt_size;

int main(int argc, char **argv)
{
    char line[MAX_CHARS] = {0};
    char *my_data_start_addr = (char *) &_binary_my_data_txt_start;
    int str_len = (char) *my_data_start_addr - 0x30; /* 0x30 => '0' */

    if (str_len > MAX_CHARS) {
        str_len = MAX_CHARS;
    }

    strncpy(line, my_data_start_addr + 1, str_len);

    printf("_binary_my_data_txt_start: %c\n", _binary_my_data_txt_start);
    printf("_binary_my_data_txt_end: %c\n", _binary_my_data_txt_end);
    printf("_binary_my_data_txt_size: %d\n", _binary_my_data_txt_size);
    printf("Read string is %s\n", line);

    return 0;
}
```
而測試檔案為
```text my_data.txt
5Hello
```


一跑起來會發生Segmentation fault如下
```
$ ./test_obj 
_binary_my_data_txt_start: 5
_binary_my_data_txt_end: 
Segmentation fault (core dumped)
```

## 分析
使用gdb可以看到當在存取`_binary_my_data_txt_size`這行，我們還可以進一步來看這個**變數**

```text gdb 結果
_binary_my_data_txt_start: 5
_binary_my_data_txt_end: 

Program received signal SIGSEGV, Segmentation fault.
main (argc=1, argv=0x7fffffffe1d8) at test_obj.c:24
24	    printf("_binary_my_data_txt_size: %d\n", _binary_my_data_txt_size);
(gdb) p _binary_my_data_txt_size
Cannot access memory at address 0x7
```
疑？變數位址是0x7?那我把測試檔案內容改一下，增加3個字元，再跑一次gdb。

```text my_data.txt
8HelloABC
```

```text gdb 結果
(gdb) p _binary_my_data_txt_size
Cannot access memory at address 0xa
```
可以看到位址從`0x7`跑到`0xa`，也就是10，這表示這個數字增加三了。這讓我懷疑這個symbol並不是一個變數，而是一個數值。因此我們可以將
* `printf("_binary_my_data_txt_size: %d\n", _binary_my_data_txt_size);`

	改成
* `printf("_binary_my_data_txt_size: %p\n", &_binary_my_data_txt_size);`

另外一個有趣的地方是`_binary_my_data_txt_end`並不是`C`，而是`\0`。而`_binary_my_data_txt_end`前一個字元是`\n`。使用`ghex`去看my_data.txt可以看到最後一個資料其實是`\n`。但是`\0`怎麼出現的，目前還不清楚。

## 參考資料
* `man objcopy`
* [Linking raw data with C code](http://bytbox.net/blog/2012/11/linking-raw-data.html)
