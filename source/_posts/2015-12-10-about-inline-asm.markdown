---
layout: post
title: "關於GNU Inline Assembly"
date: 2015-12-10 22:16:49 +0800
comments: true
categories: [gcc, Linux, asm, ARM] 
---
以前稍微接觸過GNU Inline Assembly，對於那些奇怪的符號總是覺得匪夷所思。這次找時間把他整理一下。雖然釐清了一些觀念，不過卻產生更多的疑惑，也許以後有機會看到範例會慢慢有感覺吧。

## 目錄
* [前言](#ia-preface)
* [測試環境](#ia-env)
* [語法](#ia-syntax)
    * [Output operands](#ia-output)
    * [Input operands](#ia-input)
    * [Clobbered registers list](#ia-clo-reg)
    * [Constraints](#ia-constr)
* [參考資料](#ia-ref)

<a name="ia-preface"></a>
## 前言
我自己對於GNU Inline Assembly的看法。

* 編譯器 夠聰明，所以暫存器分配可以安心交給編譯器處理。也就是說語法上面要處理這塊。
* 暫存器、變數有些資訊仍然要讓編譯器知道，讓編譯器產生object binary遵守這樣的規則，如
    * 這個operand是一個暫存器
    * 這個operand是一塊記憶體
    * 這個operand是浮點常數
    * ...
 * 不想讓編譯器幫你安排暫存器，而是在Inline Assembly指定暫存器的話，就要明確的列出來。讓編譯器知道這些暫存器有被改過資料，進而針對這些暫存器做適當的處理。

<a name="ia-env"></a>
## 測試環境
我使用ARMv7為主的Banana Pi開發版加上Lubuntu 14.04作為測試環境。

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty

$ dmesg
...
[    0.000000] Linux version 3.4.90 (bananapi@lemaker) (gcc version 4.6.3 (Ubuntu/Linaro 4.6.3-1ubuntu5) ) #2 SMP PREEMPT Tue Aug 5 14:11:40 CST 2014
[    0.000000] CPU: ARMv7 Processor [410fc074] revision 4 (ARMv7), cr=10c5387d
...

$ gcc -v
...
Target: arm-linux-gnueabihf
...
gcc version 4.8.4 (Ubuntu/Linaro 4.8.4-2ubuntu1~14.04) 
```

<a name="ia-syntax"></a>
## 語法


inline assembler關鍵字是`asm`，不過`__asm__`也可以使用([註](#ia_ps0))。

根據目前(Dec/2015)的[gcc手冊](https://gcc.gnu.org/onlinedocs/gcc/index.html#Top)，inline assembler有分為`basic`和`extended`兩種。雖然我使用的平台是gcc 4.8.4，而且gcc 4.8.5[手冊](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/index.html#Top)(官方網站上沒有4.8.4手冊)並沒有提到這個部份。但是目前**語法上**測試的確沒有問題，但是有些說明上面卻很難驗證是否可以套用到4.8.5上(例如最佳化的說明、需要注意常犯的錯誤)，請自行斟酌。

以下是整理自最新的手冊說明，請自行斟酌您使用的gcc版本是否有符合。

### Basic inline assembler
```c
     [ volatile ] asm("Assembler Template");
```

以下是整理自最新(Dec/2015)的手冊說明節錄，請自行斟酌您使用的gcc版本是否有符合。

* basic inline assembler 預設就是volatile
* 基本上編譯器只是把引號內的東西抄錄，所以只要組譯器支援的語法，就可以寫入Assembler Template內
* 和extended inline assembler的差異
    * extended inline assembler 只允許在函數內使用
    * 有`naked`屬性的函數必須使用basic inline assembler(見[註解](#ia_ps1))
    * basic inline assembler就是把template內的字串作為組合語言組譯。而`%`字元在extended inline assembler有特別意義，然而有些組合語言如x86中`%`是暫存器語法的一部份。以至於`%`字元要在extended inline assembler中改為`%%`才是真正的意思，舉個例子`%eax`->`%%eax`
* 有要使用C 語言的資料，使用extended inline assembler比較妥當
* GCC 最佳化時是有可能把你的inline assembler幹掉或是和你想的不一樣，請注意
* 你不可以從一個`asm(..)`裏面跳到另外一個`asm(..)`的label

最簡單的<del>廢話</del>範例如下
```c
    asm("nop"); /* 啥事都不要做 */
```

在沒有使用C 語言的變數下，就和一般的組合語言沒有差太多。
更複雜一點的例子可以看[rtenv](https://github.com/embedded2014/rtenv/blob/master/kernel.c#L10)裏面的使用方式：

```c
size_t strlen(const char *s) __attribute__ ((naked));
size_t strlen(const char *s)
{
    asm(
        "    sub  r3, r0, #1            \n"
        "strlen_loop:               \n"
        "    ldrb r2, [r3, #1]!        \n"
        "    cmp  r2, #0                \n"
        "   bne  strlen_loop        \n"
        "    sub  r0, r3, r0            \n"
        "    bx   lr                    \n"
        :::
    );
}
```


要注意`__attribute__ ((naked));`是有意義的。這是為何這段範例沒有直接指名用到C 語言函式變數名稱的關鍵點。有興趣請看[這邊](https://gcc.gnu.org/onlinedocs/gcc/ARM-Function-Attributes.html#ARM-Function-Attributes)，請直接找字串`naked`。

### Extended inline assembler
```c
     asm [volatile] ( AssemblerTemplate
                        : OutputOperands   // optional
                      [ : InputOperands    // optional
                      [ : Clobbers ]       // optional
                      ])
```

Assembler Template基本上就是你要寫的組語加上 Inline Assembler 專用的符號。要注意的是，在編譯的過程中，你寫的inline assembler可能由於最佳化考慮不會被組譯。如果你確認你inline assembler一定要被組譯，請加上`volatile` keyword。

Assembler 專用的符號節錄如下：

<table  style="width:100%">
    <tr>
        <th>符號</th>
        <th>說明</th>
    </tr>
    <tr>
        <td width=15%">%%</td>
        <td>單一%字元</td>
    </tr>
    <tr>
        <td>%{</td>
        <td>單一{字元</td>
    </tr>
    <tr>
        <td>%}</td>
        <td>單一}字元</td>
    </tr>
    <tr>
        <td>|{</td>
        <td>單一|字元</td>
    </tr>
    <tr>
        <td>%=</td>
        <td>只知道並驗證過會產生唯一的數字。用途部份看不懂，英文真是奧妙的東西啊。</td>
    </tr>
</table> 

AssemblerTemplate

由於[前言](#ia-preface)提到的三項個人猜測，造成inline assembler要使用C 語言變數時語法會出現很多令人眼花撩亂的符號。

由於編譯器提供協助分配暫存器和記憶體，也就是說需要有對應的語法指定目前指令的operand是什麼。GCC 有兩種方式指定，分別是

* 編號指定，從零開始編號
* Symbolic name指定: GCC 3.1以後支援[出處](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Extended-Asm.html#Extended-Asm)

分別給個範例讓各位感受一下

#### 編號指定，從零開始編號
這邊`%0`, `%1`就是編號。後面operand可以看到就是指定變數、以及變數的限制。這邊簡單解釋一下`=`表示這是一個輸出、而`r`表示變數要放在暫存器中、`m`表示變數是放在記憶體中。有興趣比對編譯出來的binary反組譯時的組合語言請看[這邊](#ia_ps2)。不過編號和指令中的operand似乎很隨意，我沒有看到特殊規範。只能交叉比對assembler template和input/output operands才能看出端倪。我猜更複雜的情況你還要比對反組譯出來的結果。

```c
#include <stdio.h>

int main(void)
{
    int var1 = 12;
    int var2 = 10;

    asm("mov %0, %1 \n  \
         add %1, %0, $1" : "=r"(var1), "=r"(var2) : "r"(var2), "r"(var1):);
    printf("var1 = %d, var2 = %d\n", var1, var2);

    asm("ldr r5, %0 \n":           : "m"(var1): "r5");
    asm("str r4, %0"   : "=m"(var2):          : "r4");
    return 0;
}
```

#### Symbolic name指定
編號的缺點就是可讀性比較差，所以gcc 3.1出現使用symbolic name的方式。至於那一個比較好，看你自己習慣。

直接把上面的範例更改一下。[GCC 4.8.5手冊](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Extended-Asm.html#Extended-Asm)上面說symbolic name隨便取，甚至和變數同名稱都可以，**只要單一asm(...)內的 symbolic name不要重複就好**。有興趣比對編譯出來的binary反組譯時的組合語言請看[這邊](#ia_ps3)。

```c
#include <stdio.h>

int main(void)
{
    int var1 = 12;
    int var2 = 10;

    asm("mov %[my_var1], %[my_var2] \n  \
         add %[my_var3], %[my_var4], $1" : 
            [my_var1] "=r" (var1), [my_var3] "=r" (var2) :
            [my_var2] "r"  (var2), [my_var4] "r"  (var1) :);
    printf("var1 = %d, var2 = %d\n", var1, var2);

    asm("ldr r5, %[my_var1] \n":: [my_var1] "m"(var1): "r5");
    asm("str r4, %[my_var1]": [my_var1] "=m" (var2):: "r4");
    return 0;
}
```


接下來來看每個欄位吧。

<a name="ia-output"></a>
#### Output operands

```c
    [ [asmSymbolicName] ] constraint (cvariablename)
```

`[asmSymbolicName]` 是GCC 3.1以後支援語法，如前所述，不用Symbolic Name就用編號方式對應assembler template operand。

指定結果要存在C 語言中的那個變數。要注意的除了要設定對的資訊（constraints，[下面](#ia-constr)會節錄) 以外，operand的prefix一定要是`=`或`+`這兩個constraint。

隨便舉幾個範例

* `=r(var1)`：變數請寫入並放在暫存器中
* `=m(var1)`：變數請寫入並存到記憶體中

<a name="ia-input"></a>
#### Input operands

```c
    [ [asmSymbolicName] ] constraint (cvariablename)
```

`[asmSymbolicName]` 是GCC 3.1以後支援語法，如前所述，不用Symbolic Name就用編號方式對應assembler template operand。

指定要從在C 語言中的那個變數取出資料。主要是要設定對的資訊（constraints，[下面](#ia-constr)會節錄) 。

* `r(var1)`：變數請放在暫存器中
* `m(var1)`：變數是在記憶體中

<a name="ia-clo-reg"></a>
#### Clobbered registers list
先講結論，在`asm("語法")`中明確地指定暫存器名稱的話，要在這邊列出。

現在我會習慣查單字。`Clobbered`查英文單字會發現就是把東西用力地砸毀。所以翻譯成中文就是「砸爛的暫存器列表」。什麼是爛掉的暫存器？就是本節前面的結論囉。

另外從Dec/2015的gcc 手冊還有找到下面[語法](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Clobbers)，一樣請注意版本問題

<table  style="width:100%">
    <tr>
        <th>符號</th>
        <th>說明</th>
    </tr>
    <tr>
        <td width="15%">"cc"</td>
        <td>和狀態有關的flag暫存器會被修改</td>
    </tr>
    <tr>
        <td>"memory"</td>
        <td>這段組合語言會讀寫列出operand以外的記憶體內容，因此編譯器會視情況備份暫存器或讀寫記憶體</td>
    </tr>
</table>


<a name="ia-constr"></a>
### Constraints

```c
<Constraints>       ::= <Constraint Modifier> <Other Constraints> | <Other Constraints>
<Other Constraints> ::= <Simple Constraints> | <Machine Constraints>

; /* 以上BNF是我整理的，terminal symbol請自行看手冊 */
```

節錄整理我<del>看得懂</del>感興趣的部份。

##### Simple Constraints
<table  style="width:100%">
    <tr>
        <th>符號</th>
        <th>說明</th>
    </tr>
    <tr>
        <td width="15%">空白字元</td>
        <td>會被忽略，排版用</td>
    </tr>
    <tr>
        <td>m</td>
        <td>operand 存放在記憶體中</td>
    </tr>
    <tr>
        <td>r</td>
        <td>operand 將被放在暫存器中</td>
    </tr>
    <tr>
        <td>i</td>
        <td>operand 是一個整數常數，該常數包含下面的情形(symbolic name)：`#define MAX_LINE (32)`</td>
    </tr>
    <tr>
        <td>n</td>
        <td>operand 是一個整數常數，只允許填入數字</td>
    </tr>
    <tr>
        <td>E</td>
        <td>operand 是一個浮點數常數，不清楚和`F`的差異</td>
    </tr>
    <tr>
        <td>F</td>
        <td>operand 是一個浮點數常數，不清楚和`E`的差異</td>
    </tr>
    <tr>
        <td>g</td>
        <td>operand 存在暫存器(r)或是記憶體內(m)，或是這是一個整數常數</td>
    </tr>    
    <tr>
        <td>X</td>
        <td>不用檢查operand</td>
    </tr>
</table>
　　 

你可以使用組合技如`"rim"`，如果這樣寫的話，意思是要編譯器幫你挑一個最適合的方式處理對應於assembler template內的operand。
　　      
##### Constraint Modifier

<table  style="width:100%">
    <tr>
        <th>符號</th>
        <th>說明</th>
    </tr>
    <tr>
        <td width="15%">=</td>
        <td>表示這是一個write only的operand，必須為contraint開始字元。</td>
    </tr>
    <tr>
        <td>+</td>
        <td>表示這個 operand 在指令中是同時被讀寫的，必須為contraint開始字元。</td>
    </tr>
    <tr>
        <td>&</td>
        <td>該operand 為earlyclobber。earlyclobber就是在instruction讀取該operand前，該operand會被寫入。雖然如此，到底是多久前？是和data hazard有關嘛？還是跟資料一致性有關？或者是和編譯器 最佳化造成非預期結果有關？真是一團謎<font color="red">完全搞不懂做啥用，也不清楚使用時機。</font><a href="http://lxr.free-electrons.com/source/arch/arm/include/asm/uaccess.h#L364">這邊有範例</a>，一樣搞不懂為什麼要有+, &的modifier</td>
    </tr>
    <tr>
        <td>%</td>
        <td>該operand 可以讓編譯器 決定這個operand是否和後面的operand交換(commutative)，<font color="red">完全搞不懂做啥用</font></td>
    </tr>
</table>
　　      
##### ARM 專用的Constraint
我參考的是gcc 4.8.5手冊(因為和測試環境的gcc版本最接近)，可能有版本的問題，這些我都沒有做實驗測試，請自行斟酌。


<table  style="width:100%">
    <tr>
        <th>符號</th>
        <th>說明(一般模式)</th>
    </tr>
    <tr>
        <td width="20%">w</td>
        <td width="80%">VFP 浮點運算</td>
    </tr>
    <tr>
        <td>G</td>
        <td>浮點運算的0.0</td>
    </tr>
    <tr>
        <td>I</td>
        <td>8 bit正整數</td>
    </tr>
    <tr>
        <td>K</td>
        <td>I contraint 的invert (一的補數)，Wen: 不知道為什麼要扯到I constraint？</td>
    </tr>
    <tr>
        <td>L</td>
        <td>I contraint 的負數 (二的補數)，Wen: 不知道為什麼要扯到I constraint？</td>
    </tr>
    <tr>
        <td>M</td>
        <td>0 ~ 32的正整數</td>
    </tr>
    <tr>
        <td>Q</td>
        <td>要參考的記憶體位址存放在一個暫存器內</td>
    </tr>
    <tr>
        <td>R</td>
        <td>operand是一個const pool內的東西，不要問我const pool是啥，估狗到都和Java有關</td>
    </tr>
    <tr>
        <td>S</td>
        <td>operand 目前檔案中.text內的一個symbol</td>
    </tr>
    <tr>
        <td>Uv</td>
        <td>VFP load/store 指令可存取的記憶體</td>
    </tr>
    <tr>
        <td>Uy</td>
        <td>iWMMXt load/store 指令可存取的記憶體</td>
    </tr>
    <tr>
        <td>Uq</td>
        <td>ARMv4 ldrsb 指令可存取的記憶體</td>
    </tr>
</table> 


完整列表在這邊，要注意的是2015年12月的手冊又多了一些新的contstraint。請自行參考。

* [gcc 4.8.5: Constraints for Particular Machines](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Machine-Constraints.html#Machine-Constraints)
    * 請自行參考你的硬體平台
* [Dec/2015手冊: 6.44.4.4 Constraints for Particular Machines](https://gcc.gnu.org/onlinedocs/gcc/Machine-Constraints.html#Machine-Constraints)


<a name="ia-ref"></a>
## 參考資料
* 中文
    * [(BIG5)用Open Source工具開發軟體: 新軟體開發關念: Chapter 4. GNU Compiler Collection](http://www.study-area.org/cyril/opentools/opentools/x969.html)
        * 題外話，寫這位文件的作者個人非常佩服，但是網路上似乎關於這位作者只有這份文件。真是神祕的人物
    * [Nano雞排: Inline Assembly](http://nano-chicken.blogspot.tw/2010/12/inline-assembly.html)
* 英文
    * [gcc: How to Use Inline Assembly Language in C Code](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html#Using-Assembly-Language-with-C)
        * [手冊]( https://gcc.gnu.org/onlinedocs/gcc/index.html#Top)上寫是給`gcc 6.0`，我目前從GGG release網站上看到[最新版本](https://gcc.gnu.org/releases.html)是5.3，怪。
        * 為什麼列出這個，因為我原本找的gcc-4.8.5 對於assembler template說明沒有特別列出gcc 支援的inline assembler符號。另外這份的文件結構的確比4.8.5清楚。
    * [gcc-4.8.5: Assembler Instructions with C Expression Operands](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Extended-Asm.html#Extended-Asm)
    * [gcc-4.8.5: Constraints for asm Operands](https://gcc.gnu.org/onlinedocs/gcc-4.8.5/gcc/Constraints.html#Constraints)
    * [ARM GCC Inline Assembler Cookbook](http://www.ethernut.de/en/documents/arm-inline-asm.html)
        * 相當推荐，不論是給的說明和範例，更厲害的是關於contraint部份寫的比手冊清楚，一樣我沒測過就是了。
    * [A Tiny Guide to GCC Inline Assembly](http://ericw.ca/notes/a-tiny-guide-to-gcc-inline-assembly.html)
    * [GCC-Inline-Assembly-HOWTO](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)
    * [OSDev: Inline Assembly](http://wiki.osdev.org/Inline_Assembly)
    * [Introduction to GCC inline assembler](http://asm.sourceforge.net/articles/rmiyagi-inline-asm.txt)
        * 似乎有點古老。


## 附錄
<a name="ia_ps0"></a>

* C 語言標準有提到編譯器可以使用`asm` keyword，而且沒有定義語法。有興趣可以找`C11`、`C99`、`C89`的標準，直接搜尋`asm`就可以看到了。

---
<a name="ia_ps1"></a>

* `naked`使用basic inline assembler和extended inline assembler比較

下面兩個函數，`strcmp1`沒有任何extended inline assembler而`strcmp2`硬塞了一個下去：
```c
int strcmp1(const char *a, const char *b) __attribute__ ((naked));
int strcmp1(const char *a, const char *b)
{
    asm(
        "strcmp_lop1:                \n"
        "   ldrb    r2, [r0],#1     \n"
        "   ldrb    r3, [r1],#1     \n"
        "   cmp     r2, #1          \n"
        "   it      hi              \n"
        "   cmphi   r2, r3          \n"
        "   beq     strcmp_lop1      \n"
        "    sub     r0, r2, r3      \n"
        "   bx      lr              \n"
        :::
    );
}

int strcmp2(const char *a, const char *b) __attribute__ ((naked));
int strcmp2(const char *a, const char *b)
{
        int i;
    asm(
        "strcmp_lop2:                \n"
        "   ldrb    r2, [r0],#1     \n"
        "   ldrb    r3, [r1],#1     \n"
        "   cmp     r2, #1          \n"
        "   it      hi              \n"
        "   cmphi   r2, r3          \n"
        "   mov     %1, $1 \n"
        "   beq     strcmp_lop2      \n"
        "    sub     r0, r2, r3      \n"
        "   bx      lr              \n"
        :"=r"(i)::
    );
}
```

我們可以比較一下下面兩個函數最後編譯出來的指令，`strcmp2`顯然和我們預期的差很多。
```c
000083f4 <strcmp1>:


int strcmp1(const char *a, const char *b) __attribute__ ((naked));
int strcmp1(const char *a, const char *b)
{
    asm(
    83f4:	f810 2b01 	ldrb.w	r2, [r0], #1
    83f8:	f811 3b01 	ldrb.w	r3, [r1], #1
    83fc:	2a01      	cmp	r2, #1
    83fe:	bf88      	it	hi
    8400:	429a      	cmphi	r2, r3
    8402:	d0f7      	beq.n	83f4 <strcmp1>
    8404:	eba2 0003 	sub.w	r0, r2, r3
    8408:	4770      	bx	lr
        "   beq     strcmp_lop1      \n"
        "    sub     r0, r2, r3      \n"
        "   bx      lr              \n"
        :::
    );
}
    840a:	4618      	mov	r0, r3

0000840c <strcmp2>:
        "   beq     strcmp_lop2      \n"
        "    sub     r0, r2, r3      \n"
        "   bx      lr              \n"
        :"=r"(i)::
    );
}
```
---
<a name="ia_ps2"></a>

* 範例一的反組譯節錄

```c
$ objdump -d -S asm
...
000083f4 <main>:
#include <stdio.h>

int main(void)
{
...
    int var1 = 12;
    83fa:	230c      	movs	r3, #12
    83fc:	603b      	str	r3, [r7, #0]

    int var2 = 10;
    83fe:	230a      	movs	r3, #10
    8400:	607b      	str	r3, [r7, #4]

    asm("mov %0, %1 \n  \
         add %1, %0, $1" : "=r"(var1), "=r"(var2) : "r"(var2), "r"(var1):);
    8402:	687b      	ldr	r3, [r7, #4]
    8404:	683a      	ldr	r2, [r7, #0]
    8406:	461a      	mov	r2, r3
    8408:	f102 0301 	add.w	r3, r2, #1
    840c:	603a      	str	r2, [r7, #0]
    840e:	607b      	str	r3, [r7, #4]
...
    asm("ldr r5, %0 \n":           : "m"(var1): "r5");
    8424:	683d      	ldr	r5, [r7, #0]

    asm("str r4, %0"   : "=m"(var2):          : "r4");
    8426:	607c      	str	r4, [r7, #4]
...
}
...
```

---
<a name="ia_ps3"></a>

* 範例二的反組譯節錄

```c
$ objdump -d -S asm
...
000083f4 <main>:
#include <stdio.h>

int main(void)
{
...
    int var1 = 12;
    83fa:	230c      	movs	r3, #12
    83fc:	603b      	str	r3, [r7, #0]

    int var2 = 10;
    83fe:	230a      	movs	r3, #10
    8400:	607b      	str	r3, [r7, #4]

    asm("mov %[my_var1], %[my_var2] \n  \
         add %[my_var3], %[my_var4], $1" : 
            [my_var1] "=r" (var1), [my_var3] "=r" (var2) :
            [my_var2] "r"  (var2), [my_var4] "r"  (var1) :);
    8402:	687b      	ldr	r3, [r7, #4]
    8404:	683a      	ldr	r2, [r7, #0]
    8406:	461a      	mov	r2, r3
    8408:	f102 0301 	add.w	r3, r2, #1
    840c:	603a      	str	r2, [r7, #0]
    840e:	607b      	str	r3, [r7, #4]
...
    asm("ldr r5, %[my_var1] \n":: [my_var1] "m"(var1): "r5");
    8424:	683d      	ldr	r5, [r7, #0]

    asm("str r4, %[my_var1]": [my_var1] "=m" (var2):: "r4");
    8426:	607c      	str	r4, [r7, #4]
...
}
```
