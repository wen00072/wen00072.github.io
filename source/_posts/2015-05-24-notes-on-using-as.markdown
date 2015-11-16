---
layout: post
title: "Using as 手冊筆記"
date: 2015-05-24 20:48:02 +0800
comments: true
categories: [Assembler, Assembly, Linux]
---
**先承認我自己很不滿意這篇，太亂了。只能當工具查keyword用。不過as 手冊的確就是指令和語法。原本是以英文字母順序說明，我只是把這些用自認的方式重新分類。很多地方也真的只有句意翻譯。就把他當作看手冊的導讀，有找的需要的再進去看[手冊](https://sourceware.org/binutils/docs/as/index.html)吧。**

本篇只討論ELF部份，其他binary format跳過。

## 目錄
* [as參數](#as_param)
* [名詞解釋](#as_term)
    * [常數](#as_term_const)
    * [Section](#as_term_sec)
        * [undefined section](#as_term_sec_undef)
    * [relocation](#as_term_rel)
* [Expression](#as_expr)
    * [Empty expression](#as_expr_emp)
    * [Integer expression](#as_expr_int)
        * [Arguments](#as_expr_int_arg)
        * [Operators](#as_expr_int_op)
* [directives](#as_dit)
    * [變數相關](#as_dit_var)
    * [Symbol的描述](#as_dit_sym_desc)
    * [Symbole type](#as_dit_sym_type)
    * [其他Symbol 相關](#as_dit_sym_other)
    * [Section](#as_dit_sec)
    * [條件以及控制相關](#as_dit_cond)
    * [巨集](#as_dit_mac)
        * [altmacro](#as_dit_mac_alt)
    * [ELF相關](#as_dit_elf)
        * [ELF section stack](#as_dit_elf_stack)
        * [ELF visibility](#as_dit_elf_vis)        
    * [除錯相關](#as_dit_dbg)
    * [未分類](#as_dit_misc)
* [參考資料](#as_ref)

<a name="as_param"></a>
## `as`參數
只提幾個我有興趣的部份

* `-Z`：硬上，就算有錯誤照樣組譯沒有錯的部份。
* `--gstabs+`：好東西，可以幫你加入debug資訊，然後直接用gdb除錯。
* 如果檔案副檔名為`.s`，就是普通的組合語言原始檔。
* 如果檔案副檔名為`.S`，就可以使用`cpp`（還記得c preporcessor吧?)來處理前置處理。

<a name="as_term"></a>
## 名詞解釋

* `symbol`：由字母、數字、和`_`、`.`、`$`組成的字串。不得以數字開頭。
    * `label`：  `symbol`後面加`:`
* `.`開頭的symbol是gas 的`directive`
* `expression`：運算式，結果代表不是位址就是單純的數字
* 原始碼不是以上的情況，由英文字母開頭組成的字串就是instruction
* 原始碼最後一行一定要是`\n`。目前網友Carl有提供為什麼這樣規定的[link](chrome-extension://klbibkeccnjlkjkiokjodocebajanakg/suspended.html#uri=http://unix.stackexchange.com/questions/18743/whats-the-point-in-adding-a-new-line-to-the-end-of-a-file)。

<a name="as_term_const"></a>
### 常數
* 字元常數：
    * `'字元`
    * 顯示\： `'\\`
* 字串：`"字串"`

<a name="as_term_sec"></a>
### Section
一個連續的記憶體空間。這段連續空間都是為了處理某些單一特定的任務如執行程式碼、存放global變數等。

題外話，`.bss`存在的目的是節省儲存空間，沒有初始的全域變數當然不需要在檔案中保留儲存空間。

<a name="as_term_sec_undef"></a>
#### undefined section
在組譯的時段只要位址無法決定的symbol，一律放到undefined section。然後祈禱linker幫你搞定。

<a name="as_term_rel"></a>  
### relocation
前面的文有提到，linker功能之一就是把不同的object檔案**黏**成一個執行檔。要怎麼**黏**呢？

每個object 檔案的起始點都是address 0。由linker計算並設定每個object檔案最後在執行檔放置的address，避免這些object的內容互相覆蓋。
 
而linker要怎麼搬移和設定最後的位址呢？這是因為object檔案內已經有規範好的不同名稱的好幾個連續空間，也就是section。所以linker把這些object檔案中相同section名稱的連續空間搬到執行檔內相同名稱的空間，並且保證執行檔內這些section的空間也是連續的。而搬移的動作並設定section的runtime address就稱為`relocation`。

Linker在relocation時需要考慮的問題，as也幫他處理了，這些問題是
 
* 目前這個位址要對應到object檔案的哪個地方?
* 這個位址會需要佔用多少byte的空間?不懂？int和char吃的空間總會不一樣吧。
* 目前位址對應到的是哪個section? 這個位址和對應section的offset為何？
* 目前的位址是絕對位址還是和program counter相對的位址?

另外要注意的是，大部分的位址可以表示成
```asm
     (section) + (offset into section)
```
     

<a name="as_expr"></a>
## Expression
expression的結果代表不是位址就是單純的數字。這些數字要嘛是絕對位址、要嘛就是某個section的offset。而expression之間可以有空白。


<a name="as_expr_emp"></a>
### Empty expression
空白字元或是null，其值會被設為0

<a name="as_expr_int"></a>
### Integer expression
由一個以上的argument和operator組成的expression

<a name="as_expr_int_arg"></a>
#### Arguments
包含 symbols, numbers 或subexpressions，分別討論

 * symbol：結果將會是 {section setction的offset數值}，數值會是32位元的二的補數（就是有正負值啦）
 * numbers：一般來說，是正整數。如果你要處理浮點數或是大數（超過32位元的數字）as會噴警告。你需要自己處理這種情況。
 * subexpressions：指的是
    * (expression)
    * prefix operator 伴隨一個 argument
    
<a name="as_expr_int_op"></a>
#### Operators
用來協助運算section中的offset位址。

* Infix Operators
    * 就一般的binary operator如`+`, `-`等
* Prefix Operators
    * `-`：負號
    * `~`：補數，就是將argument的每個位元inverse

Infix也和C語言一樣，有優先順序、符號定義也大致相同，列出如下

* 最優先
    * `*`, `/`, `>>`, `<<`
* 第二順位
    * `|`, `&`, `^`, `!`
* 第三順位
    * `+`, `-`, `==`, `<>`, `!=`, `>`, `<`, `>=`, `<=`
        * `<>`就是`!=`
* 最低順位
    * `&&`, `||`

<a name="as_dit"></a>
## directives
重頭戲。directive又稱pseudo-ops，一律以`.`開頭。照字面理解，這東西是用來協助使用開發，而不是真正的CPU instruction。這邊我只列出~~看得懂~~我感興趣的部份。有興趣請參考[出處](https://sourceware.org/binutils/docs/as/Pseudo-Ops.html#Pseudo-Ops)。另外和硬體相依的directive請參考[這邊](https://sourceware.org/binutils/docs/as/Machine-Dependencies.html#Machine-Dependencies)。

<a name="as_dit_var"></a> 
### 變數相關

* `.ascii "字串"`：可以用多個字串，中間以`,`隔開。這些字串最終會被一起放在連續的記憶體中。
* `.asciz "字串"`：和樓上的差別是字串後面會自動填`\0`，和C語言的字串表示方式相同。
* `.balign[wl] abs-expr, abs-expr, abs-expr`：和`.align`差別在b是`byte`，w是`2-byte`，l是`4-byte`。這代表什麼呢？代表要pad的數字(如果有指定的話)要注意fill byte數量。如`.balignw 8, 0xbeef`。
* `.byte expressions`：`expression`數量可以從0個到多個，中間以`,`隔開。這些`expression`會依照順序排列。那麼要幹什麼用呢？你可以這樣玩。

```asm
.byte 0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x00 /* "Hello" */
```
還有其他玩法，請參考[這邊](http://stackoverflow.com/questions/7290318/use-of-byte-assembler-directive-in-gnu-assembly)和[這邊](http://www-ug.eecg.toronto.edu/msl/assembler.html)

* `.int expressions`
* `.long expressions`
    * 上面兩個有同樣效果，`expression`為16-bit寬度。可以用`,`隔開。和`.byte`用法類似。長度以及order會和CPU架構相關。
* `.hword expressions`
* `.short expressions`
    * 上面兩個有同樣效果，`expression`為16-bit寬度。可以用`,`隔開。和`.byte`用法類似。
* `.double flonums`：就浮點數，可以用`,`隔開。和`.byte`用法類似。表示方式要看target CPU架構。
* `.float flonums`：就浮點數，可以用`,`隔開。和`.byte`用法類似。表示方式要看target CPU架構。
* `.lcomm symbol, length`：為`symbol`保留`length`的空間，該symbol型態不會是`global`，並且會被放在`.bss` section。
* `.octa 大數字`：為16-byte寬度。可以用`,`隔開。和`.byte`用法類似。
* `.quad 大數字`：為8-byte寬度。可以用`,`隔開。和`.byte`用法類似。
* `.string "字串"`：將字串放到object file中，看不出來和`.ascii`差在那。
* `.string16 "字串"`：將字串放到object file中，字串中的單個字元將會展開成2個bytes。看不出來和`.ascii`差在那。
* `.string32 "字串"`：將字串放到object file中，字串中的單個字元將會展開成4個bytes。看不出來和`.ascii`差在那。
* `.string64 "字串"`：將字串放到object file中，字串中的單個字元將會展開成8個bytes。看不出來和`.ascii`差在那。
* `.set symbol, expression`：將`symbol`的值設成`expression`的值。
* `.size symbol, expression`：設定`symbol`空間為`expression`的值。

<a name="as_dit_sym_desc"></a>
### Symbol的描述
visibility：local, global or weak

* `.extern`：單純是相容性使用，特地列出來只是因為手冊說as將所有`undefined symbols`視為`extern`
* `.global symbol`
* `.globl symbol`
    * 以上兩個同樣效果，就是讓`linker`看得到這個`symbol`，也就是說透過`nm`觀察binary也可以看得到這些`symbol`。
* `.local symbol`：讓`linker`看<font color=red>不</font>到這個`symbol`。手冊上另外有提到`.local `[不支援alignment的問題和解法](https://sourceware.org/binutils/docs/as/Local.html#Local)。<font color="red">我看不懂</font>，有興趣自行去連結參考。
* `.weak symbol`：組譯器找不到`symbol`會產生一個。

<a name="as_dit_sym_type"></a>
### Symbol type
* `.type symbol, type`：type 描述方式有[五種](https://sourceware.org/binutils/docs/as/Type.html#Type)。我只用我看順眼的那種說明。
    * `"function"`：這個`symbol`是個function
    * `"object"`：這個`symbol`用來存放資料
    * `"tls_object"`：這個`symbol`用來存放thread local資料
    * `"notype"`：沒有指定
    * `"gnu_unique_object"`：保證該`symbol`是唯一的symbol
    * `"gnu_indirect_function"`：看不懂

<a name="as_dit_sym_other"></a>
### 其他Symbol 相關
* `.desc symbol, abs-expression`：提供描述symbol的特性，細節請參考[前面](#as_dit_sym_desc)的說明。
* `.equ symbol, expression`：將`symbol`設成`expression`的值
* `.equiv symbol, expression`：和上面類似，但是如果該`symbol`之前已經定義過，就會噴錯誤。

<a name="as_dit_sec"></a>
### Section
* `.data`：不解釋
* `.test`：不解釋
* `.section name`：讓as把以下的東西組成`name`的section。名字雖然可以亂取，但是也要看binary format有沒有支援。如`a.out`就沒有這東西。

#### ELF 下的Section directive
ELF的話，這個directive有加料。說明如下： `[]`表示optional

* `.section name [, "flags"[, @type[,flag_specific_arguments]]]`
    * `flags`：可由下面的flag合體組成
        * `a`：allocatable，就是要在記憶體內吃空間，但是loader不一定會載入東西到該section
        * `e`：非executable或是shared library的section
        * `w`：可寫入
        * `x`：可執行
        * `M`：可被merge
        * `S`：該section有 zero terminated 字串
        * `G`：屬於某個section group
        * `T`：給thread local存放東西用 (存放三小？)
        * `?`：看不懂，跳過
    * `type`
        * `@progbits`：section有資料 (怎麼有種廢話的感覺？)
        * `@nobits`：沒有資料，如`.bss`這樣的section
        * `@note`：不是給程式執行的時候使用的section
        * `@init_array`：該section 有個pointer arrary 指到init 函數([補充說明1](http://stackoverflow.com/questions/10468531/whats-init-array-section-in-elf-binary) [補充說明2](https://sourceware.org/ml/binutils/2009-01/msg00286.html))
        * `@fini_array`：該section 有個pointer arrary 指到fini 函數
        * `@preinit_array`：該section 有個pointer arrary 指到pre-init 函數

由於`@`在某些平台如ARM上是註解的符號，這種情況需要用`%`替代。        

`G`和`M`有特別規範，必須隔離在雙引號外面。而同時要用這兩個flag要以`MG`順序擺放，範例如下：

* `.section name , "flags"MG,...`

Section group目前先假裝沒看到，有機會又看到再回來討論。
        
<a name="as_dit_cond"></a>
### 條件以及控制相關

* if 部份有點雜亂，懶得想範例測試，想像成C語言的`#ifdef`。剩下自己看[手冊](https://sourceware.org/binutils/docs/as/If.html)。
* `.irp symbol,values...`：和巨集概念很類似，把`.irp ... `到`.endr`之間的instruction用到`symbol`的部份全部換成value。範例如下。

.irp item, 2, 3, 4
    mov    %r\item, $\item
.endr

會展開成
```asm
    mov %r2, $2
    mov %r3, $3
    mov %r4, $4
```

* `.irpc  symbol,values...`：手冊上面的說明幾乎和`irp`相同，悲劇的是範例和`.irp`完全一致。唯一差別是`.iprc`中有提到character，只能猜測c是character。
* `.offset loc`：將locale counter設定成loc。
* `.org new-lc, fill`：同樣是更動locale counter，但是只能在同一個section中移動。另外一個要注意的是這個指令只能增加locale counter，硬要減少是不可能的。當locale couter移動後，中間的空白會填入`fill`的值。不加上`, fill` as會填0。
* `.rept 次數`：重複`.rept`到`.endr`指定的次數。
* `.skip size, fill`：產生`size`長度，`fill`值的資料。
* `.fill repeat, size, value`：產生`value`，佔用空間為`size`。是否要產生多個，否的話`repeat`填`0`，是的話`repeat`填要產生的個數。`size`和`value`為optional，`size`預設為`1`，`value`預設為`0`。
    * `.fill 2,,`
    * `.fill 2,,10`
    * `.fill 2,4,`

* `.warning "string"`：印出警告訊息。
* `.err`：噴錯誤，除非as有-Z指令，不然別想產生obj檔。
* `.error "錯誤訊息"`：印出錯誤訊息然後GG。不帶錯誤訊息as會印出檔案名稱和用了`.error`那行。

* `.fail expression`：`expression`值大於五百噴警告，小於五百噴錯誤。用在複雜的巢狀巨集或是條件式組合語言中。
* `.print "字串"`：組譯的時候stdout會印出字串。
* `.end`：表示組合語言程式結束

<a name="as_dit_mac"></a>
### 巨集
跳過，自行看[手冊](https://sourceware.org/binutils/docs/as/Macro.html)

<a name="as_dit_elf"></a>
### ELF相關

* `.symver symbol, symbol2@nodename`：指定symbol的版本號碼，一般用在shared library中。[詳細說明](https://sourceware.org/binutils/docs/as/Symver.html#Symver)懶得看，那天GG再回來看。

<a name="as_dit_elf_stack"></a>
#### ELF section stack

* `.subsection name`：把目前的section push到section stack中，並且把目前的subsection置換成`name`。
* `.popsection`：從section stack中pop最上面的section去覆蓋目前的section
* `.pushsection name [, subsection] [, "flags"[, @type[,arguments]]]`：把目前的section push到section stack中，並且把目前的section置換成`name`以及`subsection`，`type`和`argument`和`.section`的參數相同。


<a name="as_dit_elf_vis"></a>
#### ELF visibility

* `.protected symbol`：不但外部看不到該symbol，連內部要使用讀取該symbol的另外一個symbol也要在內部定義。直接舉個虛擬C語言。

```c ex.c
static int whatever = 1;
void func(void)
{
    int local = whatever;
}
```
* `.hidden symbol`：想像C語言在function前面加上`static`，觀念類似，讓該symbol無法被其他component看見。手冊這樣的symbol通常被視為[`.protect symbol`](https://sourceware.org/binutils/docs/as/Hidden.html#Hidden)，目前懶得寫程式測試。單純猜測這兩個有不同，不然幹嘛要分成兩個指令。
* `.internal symbol`：手冊上提到除了和`.hidden`有同樣效果外，不同的CPU會針對這個symbol做特別處理，到底是哪些特別處理，手冊沒說。

<a name="as_dit_dbg"></a>
### 除錯相關
大部分跳過，太多背景需要補完。

* `.def`
* `.endef`
* `.dim`：給compiler產生除錯用。
* `.file 檔案行號 檔名`：[DWARF2](http://en.wikipedia.org/wiki/DWARF)用的除錯，除錯時對應的原始碼行號。 
* `.func name[,label]`：只有開啟除錯有效，必須在結尾加入`.endfunc`。`label`就是組合語言內的`label`，也就是該function的進入點。不填的話，就在`name`加上prefix 字元當作進入點，通常prefix字元為`_`。

* `.loc fileno lineno [column] [options]`：[DWARF2](http://en.wikipedia.org/wiki/DWARF)用的除錯。整理如下
    * <font color="red">手冊假設我們很瞭debug內部資訊，但是我不會。看下來他們有提到
        * `.debug_line` 狀態機
        * `.debug_line` line number matrix
        * 不明暫存器：`is_stmt` register，`isa` register等</font>
    * 資訊放在binary 的`.debug_line` section。
    * 在debuger(?)載入`.debug_line`資訊時，讀到該行，會把參數`fileno`, `lineno`,等參數一併載入。
    * options:
        * `basic_block`：設定 `.debug_line`狀態為`basic_block`
        * `prologue_end`：設定 `.debug_line`狀態為`prologue_end`
        * `epilogue_begin`：設定 `.debug_line`狀態為`epilogue_begin`
        * `is_stmt value`：設定 `is_stmt` register 在`.debug_line`狀態為`value`，合法數值只有`0`或`1`。
        * `isa value`：設定 `isa` register 在`.debug_line`狀態為`value`，合法數值只有`0`或`1`。
        * `discriminator value`：設定 `discriminator` register 在`.debug_line`狀態為`value`，合法數值只有`0`或`1`。
* `.loc_mark_labels enable`：是否enble，`basic_block register`，<font color="red">細節完全看不懂</font>。只知道和debug line number entry有關。
* `.stabs symbol, type, other, desc, value`：用來提供資訊給symbolic debuger。詳細資訊請看[手冊](https://sourceware.org/binutils/docs/as/Stab.html#Stab)
* `.tag structname`：compiler產生的輔助directive。用來從symbol table中找出`structname`的instance。
* `.val addr`：看不懂。自己看[手冊](https://sourceware.org/binutils/docs/as/Val.html#Val)。看起來是紀錄`addr`的值，但是怎麼會和symbol table扯上關係？？


<a name="as_dit_misc"></a>
### 未分類
* `.include "file"`：從目前的位置，把`file`全部原封不動地放到之後的位置。
* `.align abs-expr1, abs-expr2, abs-expr3`：`local counter` (請參考前面linker script文)結束要對齊的位址倍數。
    * `abs-expr1`：必填。要對齊的數字。根據CPU，數字代表可能是byte，有些代表的是bit。所以要對齊8有的CPU要填8，有些CPU要填3。心理的OS，欠揍。
    * `abs-expr2`：optional。如果需要填空，可以指定填入的數值。不填就使用預設值，0。
    * `abs-expr3`：optional。指定跳過數字最多可以幾個，超過就直接不對齊了。<font color="red">(手冊用skip而不用pad讓我在想這到底差別在那？)</font>

如果後面兩個都不想填，可以直接下`.align abs-expr1,,`收工。
 
* `.comm symbol, length`：我是這樣理解啦，就是很多C語言檔案要用同一個全域變數。先摸到的先贏。可以看[我以前整理](http://wen00072.github.io/blog/2014/12/09/global-variables-from-common-symbol-on-the-c-programming-language/)的說明。另外有兩點要注意
    * 如果有同樣的symbol，在不同檔案中，設定的長度又不同，gas會選最大的。
    * ELF有隱藏的第3參數，用來指定alignment。

* `.gnu_attribute tag, value`：GNU屬性[自己查](https://sourceware.org/binutils/docs/as/GNU-Object-Attributes.html#GNU-Object-Attributes)
* `.ident 字串`：不同binary format有不同處理，在ELF中會把字串放到`.comment` section中。要注意，`file`包括command line參數中`-I`指定的路徑。
* `.incbin "file"[,skip[,count]]`：從目前的位置，把`file`原封不動地放到之後的位置。你可以透過`skip`指定從檔案起始地幾個byte後跳過。另外你也可以透過`count`指定檔案最多include幾個bytes。

另外一點要注意，`file`包括command line參數中`-I`指定的路徑。

* `.version "string"`：產生`.note` section並且將字串放入該section。


<a name="as_ref"></a>
## 參考資料
 
* [Using as](https://sourceware.org/binutils/docs/as/index.html#Top)
