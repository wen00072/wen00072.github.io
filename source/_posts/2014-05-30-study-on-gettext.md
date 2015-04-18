---
layout: post
title: 'C語言中使用gettext'
date: 2014-05-30 07:17
comments: true
categories: [gettext]
---
gettext 是1990年代Sun推出的軟體，用於處理Unix下面**程式訊息的多國語言問題**。後來GNU協會也推出了GNU gettext。本文以GNU的gettext為主。

---
## 目錄
* [概論](#ov)
* [範例](#ex)
  * [套用gettext到C語言程式碼](#ex1)
  * [產生pot檔](#ex2)
  * [產生正體中文po檔](#ex3)
  	* [po檔補充](#ex3-1)
  * [產生mo檔](#ex4)
  * [範例結果](#ex5)
* [參考資料](#ref)

---
<a name="ov"></a>
## 概論
[這張圖](http://en.wikipedia.org/wiki/File:Gettext.svg)和[這張圖](https://www.gnu.org/software/gettext/manual/html_node/Overview.html#Overview)很清楚地說明整個流程。文字解釋如下：

* 使用者在程式碼中加料，讓gettext工具可以認出來要處理的訊息
* 執行`xgettext`，將程式碼中要處理的訊息抽出存到`pot` (Portable Object Template)檔
* 執行`msginit`，指定需要的語系如正體中文、日文、法文等。程式會將`pot`檔案轉成對應的`po`(Portable Object)檔
* 執行`msgfmt`吃`po`檔產生和平台相關的文字訊息的`mo`(machine object)檔
* 接下來如果程式碼又有新的加料訊息，就不是用`msginit`**產生**`pot`檔，而是透過`msgmerge`來**更新**`pot`檔

---
<a name="ex"></a>
## 範例

* 測試環境
	* Ubuntu 12.04 LTS

---
<a name="ex1"></a>
### 套用gettext到C語言程式碼
講一下程式碼

* 要include `locale.h`和`libintl.h`
* 加入`#define _(String) gettext(String)`，當有需要翻譯轉換的訊息使用。也可以直接使用`char * gettext (const char * msgid);`
* 初始部份
  * `setlocale(LC_ALL, "");`:根據環境變數設定locale，有興趣的可以`man locale`
  * `bindtextdomain(PACKAGE, LOCALEDIR);`: PACKAGE, LOCALEDIR由外部傳入的巨集，請參考下面的Makefile。想像成namespace概念，基本上就是引導系統去特定目錄去取得`LOCALEDIR/LC_MESSAGE/PACKAGE.mo`檔。有興趣的可以`man bindtextdomain`
    * 一般來說系統上是存放在`/usr/share/locale`，然而這不是硬性規定。所以本次的Makefile故意放在local的po目錄之中，建議一定要看一下。
  * `textdomain(PACKAGE);`:讀取mo檔案，讓後面的gettext可以取得LC_ALL設定對應的訊息。
* 請注意有些註解是`/*-`，和一般的不一樣，之後執行`xgettext`時可以下`--add-comments=-`抽出註解放到`pot`檔
* 為了易讀性，拔掉錯誤檢查

```c test_gettext.c 拔掉錯誤檢查
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <locale.h>
#include <libintl.h>
#define MAX_CHAR (32)

int main(int argc, char **argv)
{
    char dest[MAX_CHAR];
    char transport[MAX_CHAR];

    /* Init gettext related APIs*/
    setlocale(LC_ALL, "");
    bindtextdomain(PACKAGE, LOCALEDIR);
    textdomain(PACKAGE);

    /*- Set up strings */
    strncpy(dest, gettext("Taipei"), MAX_CHAR);
    strncpy(transport, gettext("bus"), MAX_CHAR);

    /*- Let's print out some message */
    printf(gettext("I will go to %s by %s.\n"), dest, transport);

    return 0;
}

```

```makefile Makefile
TARGET=test_gettext
LOCALE=zh_TW
PO_DIR=$(shell pwd)/po
CFLAGS=-Wall -Werror -g -DPACKAGE=\"$(TARGET)\" -DLOCALEDIR=\"$(PO_DIR)\"
OBJS=$(patsubst %, %.o, $(TARGET))

all: $(TARGET).pot $(LOCALE).po $(TARGET).mo $(TARGET)

$(TARGET).pot: $(TARGET).c
	if [ ! -f $(TARGET).pot ] ; then                                \
		xgettext -o $(TARGET).pot --add-comments=- -k_ $(TARGET).c; \
	fi

$(LOCALE).po: $(TARGET).pot
	if [ -f $(LOCALE).po ] ; then                               \
		msgmerge $(LOCALE).po $(TARGET).pot ;                   \
	else                                                        \
		msginit --locale=$(LOCALE) --input=$^ --no-translator ; \
	fi

$(TARGET).mo: $(LOCALE).po
	mkdir -p $(PO_DIR)/$(LOCALE)/LC_MESSAGES
	msgfmt $^ -o $(PO_DIR)/$(LOCALE)/LC_MESSAGES/$@


%.o: %.c
	$(CC) -o $(patsubst %.o, %, $@) $^

clean:
	rm -f *.o *~ $(TARGET) *.po *.pot *.mo
	@echo To remove po directory, use: rm -rf $(PO_DIR)
```

---
<a name="ex2"></a>
### 產生pot檔

* Makefile中的`xgettext -o $(TARGET).pot --add-comments=- -k_ $(TARGET).c`會產生pot檔。產生的pot如下，我們可以看到剛才`/*-`抽出來的註解會被加入pot檔內。另外不要忘記更改template中的metadata。

如果是多個C檔怎麼辦呢？請用`-D`　指定目錄或是`-f`指定多個檔案

```pot test_gettext.pot
# SOME DESCRIPTIVE TITLE.
# Copyright (C) YEAR THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the PACKAGE package.
# FIRST AUTHOR <EMAIL@ADDRESS>, YEAR.
#
#, fuzzy
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2014-05-30 23:11+0800\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: LANGUAGE <LL@li.org>\n"
"Language: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=CHARSET\n"
"Content-Transfer-Encoding: 8bit\n"

#. - Set up strings
#: test_gettext.c:22
msgid "Taipei"
msgstr ""

#: test_gettext.c:23
msgid "bus"
msgstr ""

#. - Let's print out some message
#: test_gettext.c:26
#, c-format
msgid "I will go to %s by %s\n"
msgstr ""
```

---
<a name="ex3"></a>
### 產生正體中文po檔
兩種情況

* 還沒有po檔產生
	* `msginit --locale=$(LOCALE) --input=$^ --no-translator`
  		* locale會根據指定locale產生對應的po檔，以我們例子就是zh_TW.po
    * `--no-translator`單純是我懶得回答互動問題，這個可有可無。
* 產生po檔，就可以分配給專業的翻譯者處理
    * 已經翻譯過了，但是又有新的pot檔產生的話。此時需要合併以經翻譯的po檔案，產生新的po檔。新的po檔會有已經翻譯過的訊息己及新的未翻譯訊息。
        * `msgmerge $(LOCALE).po $(TARGET).pot `
  
這邊的翻譯過後po檔如下，幾點要注意的

* charset要設成`UTF-8`
* 由於語言特性，參數有可能會和英文順序不同。所以`I will go to Taipei by bus`換成正體中文可以轉成`我搭乘公車到台北`。因此，請注意gettext po檔訊息可以指定參數順序。

```po test_gettext.po
# Chinese translations for PACKAGE package.
# Copyright (C) 2014 THE PACKAGE'S COPYRIGHT HOLDER
# This file is distributed under the same license as the PACKAGE package.
# Automatically generated, 2014.
#
msgid ""
msgstr ""
"Project-Id-Version: PACKAGE VERSION\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2014-05-30 23:03+0800\n"
"PO-Revision-Date: 2014-05-30 23:03+0800\n"
"Last-Translator: Automatically generated\n"
"Language-Team: none\n"
"Language: zh_TW\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: 8bit\n"

#. - Set up strings
#: test_gettext.c:22
msgid "Taipei"
msgstr "台北"

#: test_gettext.c:23
msgid "bus"
msgstr "公車"

#. - Let's print out some message
#: test_gettext.c:26
#, c-format
msgid "I will go to %s by %s.\n"
msgstr "我搭乘%2$s到%1$s。\n"
```
---
<a name="ex3-1"></a>
#### po檔補充
po檔主要是`msgid`和`msgstr`成對，翻譯的文字放在`msgstr`這邊。而註解以`#`開頭。

另外gettext的工具在分析時也會使用特殊註解格式輔助產生正確的mo檔案，他們以`,`開頭，可以串接在同一行註解。列舉一些如下：

* `, fuzzy`: 顯示翻譯可能不正確，可由翻譯者或是在`msgmerge`時產生。翻譯者review確定翻譯沒有問題需要把該註解拿掉。
* `, c-format`：表示訊息會用到C的print format，如`%s`, `%d`等。

---
<a name="ex4"></a>
### 產生mo檔
請參考Makefile部份描述：

```
$(TARGET).mo: $(LOCALE).po
	mkdir -p $(PO_DIR)/$(LOCALE)/LC_MESSAGES
	msgfmt $^ -o $(PO_DIR)/$(LOCALE)/LC_MESSAGES/$@
```

* $^表示prerequisite而$@表示target。
* 一開始我們先建立local的`$(PO_DIR)/$(LOCALE)/LC_MESSAGES/`，將mo黨產生在該目錄。而`$(PO_DIR)/$(LOCALE)`目錄也是一開始傳給測試原始碼的定義。

---
<a name="ex5"></a>
### 範例結果
注意LC_ALL可以從`locale -a`指令列出系統支援的語系，所以這邊我們用`zh_TW.utf8`而不是`zh_TW`，有興趣的人可以自行改為`zh_TW`將會發現不會有中文顯示，有加錯誤檢查還會出現檔案找不到的錯誤訊息。

```text LC_ALL="en_US.utf8"
$ LC_ALL="en_US.utf8" ./test_gettext 
I will go to Taipei by bus.
```

```text LC_ALL="zh_TW.utf8"
$ LC_ALL="zh_TW.UTF8" ./test_gettext
我搭乘公車到台北。
```

而關於`, fuzzy`我們可以做更多的實驗。我們把zh_TW.po的Taipei更改加入`, fuzzy`

```text 加入`, fuzzy`
#, fuzzy
msgid "Taipei"
msgstr "台北"
```

跑起來就結果變成

```
$ LC_ALL="zh_TW.utf8" ./test_gettext
test_gettext
我搭乘公車到Taipei。
```

---
<a name="ref"></a>
## 參考資料
* [Wikipedia: gettext](http://en.wikipedia.org/wiki/Gettext)
* [gettext手冊](http://www.gnu.org/software/gettext/manual/gettext.html)
* [Vala with GNU gettext ](http://blog.roodo.com/rocksaying/archives/15171511.html)
* [How to do I18N through gettext](http://fedoraproject.org/wiki/How_to_do_I18N_through_gettext)
