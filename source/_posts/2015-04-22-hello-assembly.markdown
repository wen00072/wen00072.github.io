---
layout: post
title: "hello Linux ARM 組合語言"
date: 2015-04-22 21:34:16 +0800
comments: true
categories: [Assembly, ARM]
---

## 目錄
* [前言](#hello-asm-syntax)
* [測試環境](#hello-asm-env)
* [範例：版本一](#hello-asm-ex1)
* [範例：版本二](#hello-asm-ex2)
* [範例：版本三](#hello-asm-ex3)
* [範例：版本四](#hello-asm-ex4)
* [總結](#hello-asm-conl)
* [延伸閱讀及致謝](#hello-asm-ext)
* [參考資料](#hello-asm-ref)

<a name="hello-asm-syntax"></a>
## 前言
之前的文章有不少在討論執行檔該長怎麼樣。簡單來說，一個執行檔會有

* Sections：程式行為和資料會分開放在不同的sections
* 進入點，也就是system call開始執行你的程式的地方

以這樣的觀點，來看組合語言，會比較有感覺。

這次主要想要試看看如何使用組合語言印出Hello world。學過作業系統的朋友應該知道OS真正提供給使用者的介面叫作system call。有興趣的朋友可以使用`strace`研究執行檔呼叫了那些system call。這次的Hello world我有兩個線索

* 在command line執行的process會有3個馬上可以使用的file descriptor（不知道那啥的請自行估狗）分別是
  * `0`: standard in
  * `1`: standard out
  * `2`: standard error
* 有一個system call叫作`write`，你可以透過他把任何資料寫到指令的file descriptor

綜合以上，我們要幹的事就是透過組合語言做出

```c
write(1, "Hello world\n", 12);
```

這又表示組合語言中我們要做

* 呼叫system call
* 帶參數給system call，這部份需要有
  * ABI的背景知識
  * 定址方式，更精確的說，如何宣告`"Hello world\n"`，讓runtime時放在在process address space中，並將它的位址傳給system call

<a name="hello-asm-env"></a>
## 測試環境

* Host
```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty
```

* Guest OS on Qemu
  * 這邊很奇怪我的kernel用更新過的版本Qemu完全無法開機。目前裝死中。
```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux 8.0 (jessie)
Release:	8.0
Codename:	jessie

$ cat /proc/cpuinfo 
Processor	: ARM926EJ-S rev 5 (v5l)
BogoMIPS	: 643.48
Features	: swp half thumb fastmult vfp edsp java 
CPU implementer	: 0x41
CPU architecture: 5TEJ
CPU variant	: 0x0
CPU part	: 0x926
CPU revision	: 5

Hardware	: ARM-Versatile AB
Revision	: 0000
Serial		: 0000000000000000
```


<a name="hello-asm-ex1"></a>
## 範例：版本一
我本來想說慢慢來，先來個完全沒意義的r0 = 0; r1 = 1; r2 = r0 + r1。程式如下：
```asm hello.s
.text
.global _start
_start:
    mov %r0, $0
    mov %r1, $1
    add %r2, %r0, %r1
.end
```
幾點說明：

* `$`或是`#`代表一個數字([出處](https://sourceware.org/binutils/docs/as/ARM_002dChars.html#ARM_002dChars))
* `%r1`代表ARM的`r1`暫存器，但是為何用`%`目前沒找到手冊上有說明。
* `.text`前面的文章有看應該覺得很眼熟，就是告訴編譯器以下是程式行為。
* `global`是讓symbol可以外露，白話來說就是`nm`等binutil可以看的到這個symbol。
* `_start`是一個程式執行的起始點，有看過[之前文章](http://wen00072.github.io/blog/2015/02/14/main-linux-whos-going-to-call-in-c-language/)就會覺得很眼熟。
* [.end表示程式結束點](https://sourceware.org/binutils/docs/as/End.html#End)，不過目前用起來有沒有加好像沒有差別。

* Makefile

```makefile Makefile
TARGET=hello
AS_FILE=$(addsuffix .s, $(TARGET))

$(TARGET): $(AS_FILE)
	$(AS) $^ -o $@

clean:
	rm -rf $(TARGET)
```

想法很簡單，就是直接編譯應該可以跑，雖然完全不會有畫面。<font color="red">錯！</font>跑出來會這樣

```text
$ make
as hello.s -o hello

$ ls -gG
total 12
-rw-r--r-- 1 588 Apr 20 18:04 hello
-rw-r--r-- 1  83 Apr 20 17:58 hello.s
-rw-r--r-- 1 113 Apr 20 17:58 Makefile
```
這代表什麼，hello編譯完後的binary本身沒有執行權限。改改權限看可不可以跑？

```text
$ chmod +x hello
$ ./hello 
bash: ./hello: cannot execute binary file: Exec format error
```

怎麼回事？分析一下

```text
$ readelf hello -h
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          268 (bytes into file)
  Flags:                             0x5000000, Version5 EABI
  Size of this header:               52 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           40 (bytes)
  Number of section headers:         8
  Section header string table index: 5
```
看不出來對不對？我是這樣啦，所以先比對`/bin/ls`的輸出吧

```text
$ readelf /bin/ls -h
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x14354
  Start of program headers:          52 (bytes into file)
  Start of section headers:          99372 (bytes into file)
  Flags:                             0x5000202, has entry point, Version5 EABI, soft-float ABI
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         9
  Size of section headers:           40 (bytes)
  Number of section headers:         27
  Section header string table index: 26
```

仔細看一下`Type:`，`/bin/ls`是`EXEC`而`hello`是`REL`，`man elf`可以看到`REL`是relocate file，那是啥呢？根據[System V Application Binary Interface - DRAFT - 10 June 2013第四章的簡介](http://www.sco.com/developers/gabi/latest/contents.html)，簡單來說就是object檔案，也就是說link時吃的檔案。所以我們加入Link吧。

<a name="hello-asm-ex2"></a>
## 範例：版本二
單純加入linker看看會怎樣？
```makefile Makefile
TARGET=hello
AS_FILE=$(addsuffix .s, $(TARGET))

$(TARGET): $(AS_FILE)
	$(AS) $^ -o $@.o
	$(LD) $@.o -o $@

clean:
	rm -rf $(TARGET)
```

kerker，一樣GG。

```text
$ make
as hello.s -o hello.o
ld hello.o -o hello
$ ./hello 
Illegal instruction
```

估狗到的組合語言的[Hello world範例](http://peterdn.com/post/e28098Hello-World!e28099-in-ARM-assembly.aspx)最後會呼叫`exit` system call，照著呼叫`exit`就可以正常結束，這就是第三版。至於為何會出現`Illegal instruction`，根據[Scott Tsai大大](http://scottt.tw/)的提示，當你的程式碼跑完後，接下來記憶體有啥CPU就跑啥，跑到不認識的opcode當然就GG了。

<a name="hello-asm-ex3"></a>
## 範例：版本三
```asm hello.s
.text
.global _start
_start:
    mov     %r0, $0
    mov     %r7, $1
    svc     $0
.end
```

單純叫了`exit`而已，有幾點注意的
根據[Debian ARM system call interface](https://wiki.debian.org/ArmEabiPort#System_call_interface)，可以知道

* r0 ~ r6是函數的帶入參數
* r7 是system call number，而system call number是啥呢？就是你要的system call 對應的數字。

所以要呼叫`exit(0)` system 表示

* 傳一個參數，數值為`0`
* 要設定`exit`對應的system call number為`1`。

為何system call number是`1`呢？可以看看[unistd.h](https://git.kernel.org/cgit/linux/kernel/git/rpi/linux-rpi.git/tree/arch/arm/include/asm/unistd.h)裏面system call number的定義，exit的system call number為`1`。

設定完傳給`exit`參數後呼叫了一個組合語言指令[`svc`](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0489g/Cihidabi.htmll)，這個指令主要是切換到Supervisor模式。Linux下面也許太過複雜，不太容易從user mode一路追到kernel然後又看懂這些system call的行為。沒關係成大資工作業有比較看得懂的範例可以參考。例如[包裝呼叫system call的函數](https://github.com/embedded2014/rtenv/blob/master/syscall.s#L6)以及[Kernel中對應的system call服務實作](https://github.com/embedded2014/rtenv/blob/master/kernel.c#L1135)。

根據[unistd.h](https://git.kernel.org/cgit/linux/kernel/git/rpi/linux-rpi.git/tree/arch/arm/include/asm/unistd.h)定義的system call number，`write`的system call number 是`4`，所以我們可以開始寫最後的版本了。
    
<a name="hello-asm-ex4"></a>
## 範例：版本四
開始之前，先來看男人怎麼介紹write system call的

```c    
ssize_t write(int fd, const void *buf, size_t count);
```

這代表

* 有三個參數要傳給system call
* 有回傳值可以吃
* 其中一個參數是位址，這個位址我們會放`"Hello World\n"`字串

那麼我先來看看怎麼放字串到記憶體

```
.data
hello_str:
    .ascii "Hello World\n"
```
`.data`如果有看我以前的文章，就知道這是放有初始值全域變數的地方。

而`hello_str`呢?嗯，對`_start:`有印象嗎？

這叫作label，是GNU組語中symbol的一種，有興趣可以看[這邊](https://sourceware.org/binutils/docs/as/Labels.html#Labels)。根據手冊，label還有一個功能，代表目前跑到的位址。所以`_start:`就是`.text` section的起始位址。而`hello_str`就是`.data` section的起始位址。從這兩個label可以看到label只是一個位址，可以指向函數或是資料，這和C語言的指標有異曲同工之妙。有興趣的朋友可以去找function pointer和C語言的callback函數。

而[`.ascii`](https://sourceware.org/binutils/docs/as/Ascii.html#Ascii)，單純就是宣告字串指令。


呼叫`write` system call來還有兩個問題要處理

* 如何取得`hello_str`對應的位址放到暫存器`r1`上面？
* 要怎麼算出`hello_str`字串的長度?

關於第一個問題，[GNU ARM組合語言有中將數值或位址放到暫存器的虛擬指令](https://sourceware.org/binutils/docs/as/ARM-Opcodes.html#ARM-Opcodes)

```asm
ldr <register> , = <expression>
```

[expression](https://sourceware.org/binutils/docs/as/Expressions.html#Expressions)是一種表示**位置**或是**數值**的方式。

恰巧[symbol也算是一個expression](https://sourceware.org/binutils/docs/as/Arguments.html#Arguments)，所以可以表示成：

```asm
ldr %r1, =hello_str
```

第二個問題呢？要介紹[`.`](https://sourceware.org/binutils/docs/as/Dot.html#Dot)了。之前看過linker script的朋友應該對於locale counter還有印象。locale counter代表目前的位置。加上[`expression`也支援運算](https://sourceware.org/binutils/docs/as/Prefix-Ops.html#Prefix-Ops)。利用`hello_str`是`.data`
開頭，我們可以這樣做:

```
.data
hello_str:
    ascii "Hello World\n"
hello_len = . - hello_str
```

綜合上面的討論，版本四組合語言會是
```asm hello.s
.data
hello_str:
    .ascii "Hello World\n"
hello_len = . - hello_str

.text
.global _start
_start:
    /* %r0 = write(1, hello_str, hello_len); */
    mov     %r0, $1
    ldr     %r1, =hello_str
    ldr     %r2, =hello_len
    mov     %r7, $4
    svc     $0

    /* exit(%r0) */
    mov     %r7, $1
    svc     $0

.end
```

等等，不是說有回傳值？還是要看[Procedure Call Standard for the ARM Architecture
ABI r2.09手冊](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ihi0042e/index.html)，上面提到回傳資料會放到`r0`，剛好接下來的`exit` system call帶的第一個參數也要存放在r0，那麼我們可以直接觀察執行後回傳值如下：

```text
$ make
as hello.s -o hello.o
ld hello.o -o hello

$ ./hello 
Hello World
$ echo $?
12
```
### 補充
```
ldr <register> , = <expression>
```
這是個有趣的指令，這個指令事實上是個虛擬指令。怎麼說呢，因為這個指令的目標是**把數值塞到指定的暫存器**。這個數值是位址還是啥死人骨頭並不重要。重要的是，由於opcode的限制，把數值塞到指定的暫存器會有限制滴。例如`ARM Cortex M0`的`MOV`的數值只有8-bit，要塞32-bit的數值就需要配合其他的指令做連續技。因此
```
ldr <register> , = <expression>
```
這樣的指令就可以讓你寫起來比較輕鬆。

另外一個值得一提是，如果組譯器無法把`ldr <register> , = <expression>`虛擬指令轉換成`MOV`或`MVN`指令，把你的數值塞到暫存器的話。組譯器會把你的值放在一塊記憶體中，使用**真的**ldr把這塊記憶體的值載入到暫存器中。這個方法稱為literal pool，細節可以看[這邊](http://infocenter.arm.com/help/index.jsp?topic=%2Fcom.arm.doc.dui0473c%2FBgbccbdi.html)。

<a name="hello-asm-conl"></a>
## 總結
本文從會GG的組合語言一路改到可以印出Hello World，並且在程式結束後回傳字串長度。在文章中簡單提到了GNU as的

* 組合語言中section
* 組合語言的編譯方式
* 組合語言中的symbol和字串表示方式
* 組合語言中的expression
* ABI實例

希望對有需要的朋友有所幫助。

<a name="hello-asm-ext"></a>
## 延伸閱讀及致謝
感謝[Scott Tsai大大](http://scottt.tw/)提供的資料以及指出文章中錯誤的地方。另外他也有提到其他有趣的部份，當作以後的作業。先列出如下

* [從組合語言直接呼叫header file內的system call](http://stackoverflow.com/questions/28831763/hello-world-program-in-nasm-x86-64-prints-hello-world-continuously/28837680#28837680)
* [Kernel Memory Layout on ARM Linux](http://www.arm.linux.org.uk/developer/memory.txt)
  * 這邊主要討論的是`objdump -d`發現obj檔和執行檔差別只在進入點位址的差別，而進入點位址如何決定呢？這邊有規範，另外也可以看default linker script看看如何設定的。
* [Scott大大的範例程式](https://gist.github.com/scottt/a57220efc15d15569a2e)
  * 可以看到，這個版本和我參考寫出來的版本差異有
     * 把`"Hello world"`放在`.rodata` section中，這比`.data`更實際，因為這個字串的確沒有必要設成全域變數。
     * 使用了preprocess方式。  
     * 提供了反組譯的結果
     * 提供X86-64版本的組合語言比較   

感謝[Viler Hsiao大大](http://vh21.github.io)寫文回答我的問題。

* [Viler Hsiao: 閱讀assembly code](http://vh21.github.io/assembly/2015/04/20/read-assembly-code.html)

<a name="hello-asm-ref"></a>
## 參考資料
* [『Hello World!』 in ARM assembly](http://peterdn.com/post/e28098Hello-World!e28099-in-ARM-assembly.aspx)
  * 本篇程式碼大量參考這篇。 
* [GNU Manual: Using as](https://sourceware.org/binutils/docs/as/index.html)
