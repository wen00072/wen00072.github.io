---
layout: post
title: "筆記 - GNU coding style"
date: 2016-12-18 21:34:23 +0800
comments: true
categories: [GNU, C]
---
這是沒事亂看的，主要找和C 語言相關的描述，就不要在意文章組織和可讀性了。

閱讀版本：`July 25, 2016`

## CH 3
* 要保證C程式碼可移植性，compile flag可以使用`--ansi`, `--posix`, `--compatible`
* 可以使用下面的方式取代原本的巨集條件編譯，現在的gcc已經可以產生一樣的結果了。

```c 原本程式碼
#ifdef HAS_FOO
...
#else
...
#endif
```

```c 建議使用方式
if (HAS_FOO)
...
else
...
```


## Ch 4
* `--pedantic`：產生所有ISO C和ISO C++規範的警告訊息，並將所有C的extension退貨
* 冷門：POSIX 2.0規範du/df的unit是512 bytes

### 4.2 Writing Robust Programs
* 使用動態配置資料方式避免資料長度限制，包含檔案名稱長度、一行長度等
* 讀檔案時不得丟棄`NUL`字元以及不可列印字元，並需相容多位元編碼方式如`UTF-8`
* 除非要刻意忽略錯誤，使用system call必定檢查回傳值，並使用`perror`或`strerror`等方式列印出錯誤訊息
* 使用`mallic`及`relloc`一定要檢查回傳值是否為NULL。當使用`relloc`要求分配比原本空間更小時更需要注意，因為實作關係可能最後分配的空間block可能和原本的不同
* 使用`free`後就不要留戀還在裏面的資料了吧
* 一旦`malloc`出現錯誤時，非互動程式請將他視為嚴重錯誤。互動程式就儘早自殺(`abort()`)吧
* 使用`getopt_long`處理命令列參數
* 避免操作低階UNIX介面，以減少相容性問題
* 在程式中檢查到不可能發生的狀況，直接給他死。既然不可能發生了，顯然那邊出大問題，儘早發現儘早處理。因為就是不可能發生，而死掉一定從這邊找起，寫程式時就在這邊提供更多註解和資訊吧
* 使用全域變數或static 變數儘量給初值。（不確定是否有看懂，[出處](https://www.gnu.org/prep/standards/html_node/Semantics.html#Semantics)）
* 不要從回傳值傳回發生錯誤的次數，因為只有256個狀態而已，很容易overflow的。
* 使用存放暫存檔到`$TMP_DIR`環境變數而不是閉著眼睛寫到`/tmp`下面
* 同樣的，暫存檔請將權限設為0600確保資訊不會洩漏

### CH 4.3 Library Behavior

* 儘量讓函數reentrant
* Name covention 很重要，因為library是給別人用，粗心大意就出現name conflict 的問題，以下是GNU的建議
    * external 函數和變數需加`prefix`，prefix為兩個字元以上（後面英文太爛看不懂）
    * external 但是不想讓使用者看到（或是文件不會提到的）的函數以`_`開頭加上prefix用來辨識這是內部函數
    * 內部static 就請自便

### CH 4.4 Formatting Error Messages
* 錯誤訊息格式，我只挑一個，有興趣看全文請參考[這邊](https://www.gnu.org/prep/standards/html_node/Errors.html#Errors)
    * `sourcefile:lineno: message`

### [CH 4.5 Standards for Interfaces Generally](https://www.gnu.org/prep/standards/html_node/User-Interfaces.html#User-Interfaces)
很奇怪的，單字句子都看懂，就是不懂他要表達啥。只能猜測說不要生出一堆很功能類似的執行檔，而是用最少執行檔加上參數取代。以及執行檔儘量device independent

### 4.12 File Usage
檔案有可能會存放在readonly FS，如果要寫東西寫到必定是可寫的目錄如`/var`或是`/tmp`目錄中

##  Writing C

* 單行最多79個字元

### 註解

* 檔案開頭請加註解說明該檔案的用途
* 用英文寫註解
* 每個函數都要說明該函數的用法，參數，回傳值等資訊
* `#endif` 後加上對應的`#ifdef`或`#ifndef` 說明，範例如下：

```c
#ifdef ASDF
...
#else  /* ASDF */
...
#endif /* not ASDF */

#ifnef QWER
...
#endif /* not QWER */
``` 

###  Clean Use of C Constructs

* 所有的宣告都要明確，例如不要因為預設回傳值是`int`就省略要回傳`int`的函數對應宣告
* 視情況自行決定是否要使用嚴格的語法檢查如`-Wall`，編譯器是我們的奴隸，你自己清楚要做什麼就好。不要為了避免嚴格語法檢查硬上一些奇怪的語法導致可讀性變差
* `extern` 要嘛全部放在C檔案的同一塊地方，要嘛集中到header file，不要東一撮西一撮。更嚴禁在函數內使用
* 不要在函數中重複使用一些無意義名稱的變數如`tmp`之類的，個別變數就給予個別有意義的變數名稱，不要偷懶。也可以在需要的最小scope宣告變數增加可讀性。
* 變數要注意是否和global 變數或是更大scope名稱相同，您可以開啟`-Wshadow`協助偵測這類的錯誤
* 多個變數就個別宣告型態，一方面增加可註解的空間，一方面可讀性也較佳。範例如下

```c
/* Bad */
int i_am_bad, i_am_super_bad;

/* Good */
int i_am_good   = 0;
int i_am_better = 0;
```

* 不要在判斷式加入assign，這樣很容易出錯，範例如下

```c
    /* Bad */
    if ((my_id = get_id(my_record)) != NONE) {
        /* Prcess data */
    }
    else {
        return NO_DATA;
    }
```

```c
    /* Good */
    my_id = get_id(my_record);
    if (my_id == NONE) {
        return NO_DATA;
    }
    /* Prcess data */
```

### Naming Variables, Functions, and Files

* Global name 要有意義，不要亂取。
    * ps: 個人會加個`g` prefix提醒一下
* 縮寫如src等可以使用。但是請確定讀者可以知道該縮寫或是該縮寫可能造成讀者模糊混淆
* 使用常數時儘量用`enum`取代巨集

### Portability between CPUs

**<font color="red">完全看不懂，抱歉</font>**

### 剩下的是平台相容性和國際化字元等，沒興趣所以跳過

### 參考資料

* [GNU Coding Standards](https://www.gnu.org/prep/standards/html_node/)
