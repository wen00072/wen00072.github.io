---
layout: post
title: 'Linux中使用C語言載入data object 檔案資料'
date: 2014-04-15 16:20
comments: true
categories: [Linux, C, bintuils]
---
之前看[別人的程式](https://github.com/southernbear/rtenv)，看到作者在ROMFS實作時把資料轉成object檔案，然後在C語言中直接存取資料。他的作法是

* 寫一個host程式，將某個目錄下面的檔案和目錄轉成單一個binary
* 將這個binary轉成object檔案
* 更改link script，讓最後的binary可以存取object檔案的section

這個project使ARM的架構Realtime 作業系統。看完手癢也想看看Linux下面要怎麼做到類似的功能。本來想說是不是要動到link script，後來看到[這邊](http://descent-incoming.blogspot.tw/2012/11/elf.html)的參考資料後，發現只要存取symbol就可以使用了。克服這個問題後就可以來寫個小程式驗證一下。

首先我們先弄個測試資料，很簡單就是一個數字顯示後面字串長度後再存放字串。
```text my_data.txt
5Hello
```

接下來就是Makefile部份

```makefile Makefile
CFLAGS=-Wall -Werror -g

DATA_SRC=my_data.txt
DATA_OBJ=my_data.o

TARGET=test_obj
OBJS=$(patsubst %, %.o, $(TARGET)) $(DATA_OBJ)

BFN=elf64-x86-64
ARCH=i386

$(TARGET): $(OBJS)
	$(CC) -o $@ $(OBJS)

$(DATA_OBJ): $(DATA_SRC)
	objcopy -I binary -O $(BFN) -B $(ARCH) --prefix-sections '.mydata' $^ $@

clean:
	rm *.o *~ $(TARGET) -f

```

Makefile說明:
* `patsubst`
	* Makefile 提供的函數，用在pattern 替換，所以上面做了下列的代換
  	* test_obj轉成test_obj.o
* `$@`
	* Makefile的內建巨集，代表target
* `$^`
	* Makefile的內建巨集，代表prerequisite 

重點是把資料檔案轉成object的部份，節錄該部份並說明如下
`objcopy -I binary -O $(BFN) -B $(ARCH) --prefix-sections '.mydata' $^ $@`

* `-I binary` 
	* 指定input檔案binary格式為binary
* `-O elf64-x86-64` 
	* 指定output binary 格式為elf64-x86-64
* `-B i386`
	* 指定架構為i386
* `--prefix-sections '.mydata'`
	* 指定放在.mydata的section

至於要怎麼知道binary格式和架構呢？您可以用`objcopy --info |less`配合`file test_obj.o`找出平台上的參數。

我們先來看一下section是不是真的放到.mydata?
```text objdump結果
$ objdump -h my_data.o

my_data.o:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .mydata.data  00000007  0000000000000000  0000000000000000  00000040  2**0
                  CONTENTS, ALLOC, LOAD, DATA
```

我只知道.mydata的section，裏面的資訊目前還不清楚有什麼意義。

我們再用nm來看檔案內有哪些symbol

```text nm結果
$ nm  my_data.o
0000000000000007 D _binary_my_data_txt_end
0000000000000007 A _binary_my_data_txt_size
0000000000000000 D _binary_my_data_txt_start
```
flag說明
* `D`
	* 資料放在初始的資料section
* `A`
	* Symbol的值已固定，之後link也不會更動。目前還不知道這樣透露出有什麼額外的資訊

接下來就是存取的方式了，測試程式如下。主要就是從section讀出字串再印出來。

```c test_obj.c
#include <stdio.h>
#include <string.h>

#define MAX_CHARS (32)

extern char _binary_my_data_txt_start;
int main(int argc, char **argv)
{
    char line[MAX_CHARS] = {0};
    char *my_data_start_addr = (char *) &_binary_my_data_txt_start;
    int str_len = (char) *my_data_start_addr - 0x30; /* 0x30 => '0' */

    if (str_len > MAX_CHARS) {
        str_len = MAX_CHARS;
    }

    strncpy(line, my_data_start_addr + 1, str_len);

    printf("Read string is %s\n", line);

    return 0;
}
```

程式說明如下
* 前面nm看到的symbol `_binary_my_data_txt_end`, `_binary_my_data_txt_start`, 和`_binary_my_data_txt_size`只是一個sybmol，這邊我們存的是字元所以宣告成`char`
* _binary_my_data_txt_start存放的是該section開始的資料，我們可以用gdb驗證一下

```text gdb 節錄
Breakpoint 1, main (argc=1, argv=0x7fffffffe1d8) at test_obj.c:10
10	    char *my_data_start_addr = (char *) &_binary_my_data_txt_start;
(gdb) p _binary_my_data_txt_start
$1 = 53 '5'
```
* 所以我們可以取得該資料位址後，讀出字串長度後，再複製後面的字串並列印出來。

最後執行結果如下
```text 執行結果
$ ./test_obj 
Read string is Hello

```

## 參考資料
* [rtenv](https://github.com/southernbear/rtenv)
* [將檔案系統塞到 elf 執行檔](http://descent-incoming.blogspot.tw/2012/11/elf.html)
