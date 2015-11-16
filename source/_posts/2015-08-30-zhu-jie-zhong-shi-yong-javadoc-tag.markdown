---
layout: post
title: "註解中使用Javadoc tag並將其轉成pdf 說明文件"
date: 2015-08-30 14:02:01 +0800
comments: true
categories: [javadoc]
---
## 目錄
* [前言](#jdoc-preface)
* [測試環境和程式](#jdoc-example)
* [javadoc 專用註解](#jdoc-comments)
    * [javadoc 專用註解tag節錄](#jdoc-comments-tags)
    * [套用 javadoc 範例](#jdoc-comments-ex)
* [參考資料](#jdoc-ref)


<a name="jdoc-preface"></a>
## 前言
[javadoc](http://www.oracle.com/technetwork/articles/java/index-jsp-135444.html) 是一套java程式碼文件產工具。它透過在程式碼註解置入javadoc專用的tag，讓自動產出的文件內容提供更詳細的資訊和美觀的呈現方式。這邊整理資訊如下

在開始加入前，我們可以想一下你要看SDK文件會想看什麼，我自己會想到

* 方塊圖/元件簡介
* 函數詳細說明如
    * 函數名稱
    * 函數功用
    * 函數參數說明
    * 函數回傳值
    * 函數使用注意事項
    * 和函數功能相關的其他函數。比如說看到`play`你會想看`open`、`pause`、`stop`、`seek`等。

接下來看看我查到的javadoc工具能夠幫我們做到那邊吧？

註：時間和能力有限，有些功能也許沒找到是因為自己沒看到而不代表javadoc辦不到，請讀者自行斟酌。

<a name="jdoc-example"></a>
## 測試環境和程式

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty
```

### 檔案和目錄結構

```
$ tree
.
├── com
│   └── test
│       └── test.java
├── config.properties
├── Makefile
└── pdfdoclet-1.0.3
    ├── build.xml
       ....
    └── jar
        └── pdfdoclet-1.0.3-all.jar
```

### Makefile
就單純編譯一個java檔，以及對應的pdf文件，並使用Ubuntu 上面預設的pdf閱讀程式開啟之。pdf文件使用[pdfdoclet](http://pdfdoclet.sourceforge.net/)產生。該套件需自行下載，我將下載的pdfdoclet解壓縮到測試程式同一層目錄中。

```makefile Makefile
SRC_PATH=com/test
TARGET=$(SRC_PATH)/test
SRC_FILE=$(addsuffix .java, $(TARGET))

$(TARGET): $(SRC_FILE)
	javac $^
	javadoc -docletpath pdfdoclet-1.0.3/jar/pdfdoclet-1.0.3-all.jar \
            -doclet com.tarsec.javadoc.pdfdoclet.PDFDoclet $^ -verbose \
            -pdf $@.pdf -config config.properties
	evince $(TARGET).pdf

clean:
	rm -rf $(SRC_PATH)/*.class $(TARGET).pdf
``` 

### pdfdoclet config檔案

```text config.properties
# Prints @author tags if set to "yes".
tag.author=yes

# Prints @version tags if set to "yes".
tag.version=yes

# Prints @since tags if set to "yes".
tag.since=yes

# Show the Summary Tables if set to "yes".
summary.table=yes

# Encrypts the document if set to "yes".
encrypted=yes

# The following property is ignored
# if "encrypted" is not set to yes.
allow.printing=yes

# Creates hyperlinks if set to "yes".
# For print documents, use "no", so
# there will be no underscores.
create.links=yes

# Creates an alphabetical index of all
# classes and members at the end of the
# document if set to "yes".
create.index=yes

# Creates a navigation frame (or PDF
# outline tree) if set to "yes".
create.frame=yes

# Creates a title page at the beginning
# of the document if set to "yes".
api.title.page=yes

# Defines the title on the title page if
# no external HTML page is used.
api.title=Test pdfdoeclet

# Defines the author text on the
# title page.
api.author=by Wen Liao
```

### 測試原始碼
印出九九乘法表，我不會Java，更不會OO。這個程式示範了在class中呼叫另外一個class method。如果有人有發現寫不好的請指正，感恩！

```java com/test/test.java
package com.test;

public class test {
    public class mul {
        public int multiply(int i, int j) {
            return i * j;
        }
    }

    public static final int MAX_NUM = 9;
    public static void main(String[] args) {
        test my_test = new test();
        my_test.hello();
    }

    private void hello() {
        mul my_mul = new mul();
        for (int i = 1; i <= MAX_NUM; i++) {
            for (int j = 1; j <= MAX_NUM; j++) {
                System.out.println(i + " x " + j + " = " + my_mul.multiply(i, j));
            }
        }
    }
}
```

<a name="jdoc-comments"></a>
## javadoc 專用註解說明
還沒加入javadoc註解的pdf檔[如link，應該沒病毒吧?](/files/javadoc-20150830/test_init.pdf)。接下來讓我們開始吧。

要引入javadoc的註解，要加入javadoc的註解專用格式如下。

```java javadoc註解
/**
* 兩個星號開頭的註解，這行的第一個*可以省略
*/
```

在介紹tag前，從參考資料中整理了一些注意事項

1. 一個javadoc 註解可以大概分成兩個部份
    * 主要文字描述部份，描述特定的class, interface, method是做什麼，操作時有哪些需要注意的事項。
    * tag section，除了註解開頭的`*`外，一行以`@`開頭的javadoc tag註解。所有的主要文字描述**必須**要放在tag section前面。
        * tag又可以分為
            * block tag，就是上面講的`@`放在行首的情形
            * inline tag，以`{@java_tag_名稱 參數}`表示
2. 第一個`@`符號出現在單行最開頭之前的所有註解，視為該註解往下最接近的method說明。
3. 註解中兩個段落以`<p>`間隔。
4. javadoc註解的開頭使用簡潔的語句描述下面實體如class/method等。想像成git commit log第一行對於git log的重要性。
5. 註解中可以插入HTML語法，包括`<a href...>`、`<img src=...>`和`<pre>...</pre>`。因為這樣，要顯示`<`，`&`之類字元的要使用[HTML Entities](http://www.w3schools.com/html/html_entities.asp)如`&amp`。
6. javadoc註解文件分類
    * doc 文件，在class, interface, method等宣告前的註解，用來解釋該class、interface等。
    * overview 文件，整個產出文件的簡介說明。
    * package 文件，單個package的簡介文件。

以下是典型的javadoc 註解範例

```java 
/**
* 說明你的method
* @param 參數說明，這就是一個javadoc tag，*可有可無。
*/
```

上列第一項的規範，引起另外兩個問題：

* 一個package的要怎麼說明呢？
* 更大的狀況，整個套件有多個package要怎麼說明呢？

關於上面的情況，參考文件有提到：

* package的說明有兩種，請二擇一
    * 寫在package同目錄中的 `package-info.java`
    * 寫在package同目錄中的 `package.html`
        * 請把javadoc的註解格式放在`<body></body>`之間
* 整個套件的說明有兩種，需要寫一個html檔案，把你的說明放在裏面。並且在執行`javadoc`加入`-overview 你的套件html檔名`

很複雜嘛？看個範例，更改上面的範例

* Makefile，新增參數是指定`-overview`的檔案

```makefile 新增package和overview文件的 Makefile
SRC_PATH=com/test
TARGET=$(SRC_PATH)/test
SRC_FILE=$(addsuffix .java, $(TARGET))
SRC_FILE+=package-info.java
OVERVIEW_FILE=overview.html

$(TARGET): $(SRC_FILE)
	javac $^
	javadoc -docletpath pdfdoclet-1.0.3/jar/pdfdoclet-1.0.3-all.jar \
            -doclet com.tarsec.javadoc.pdfdoclet.PDFDoclet $^ -verbose \
            -pdf $@.pdf -overview $(OVERVIEW_FILE) -config config.properties
	evince $(TARGET).pdf

clean:
	rm -rf $(SRC_PATH)/*.class $(TARGET).pdf
```


新增兩個檔案

* overview.html

```html overview.html
<html>
<body>
This a overview document for testing. It should supports javadoc overview tags. However, somehow my pdfdoclet did not get javadoc tag to work while javadoc generated file has no problem at all. There might some missing actions.

</body>
</html>
```

* package-info.java

```java package-info.java
/**
 * This is a page to describe your package
 */

package test;
```

現在產出的pdf檔[如link，應該沒病毒吧?](/files/javadoc-20150830/test_overview_package_info.pdf)

<a name="jdoc-comments-tags"></a>
### javadoc 專用註解tag節錄
偷懶不列support 版本，需要自行找尋相關資料

<table border="1">
<tr>
<th>tag名稱</th>
<th>功能</th>
</tr>
<tr>
<td>@author 作者名稱</td>
<td>請望文生義，預設不會放在產出網頁，需要透過command line才會出現在文件中</td>
</tr>
<tr>
<td>{@code 程式碼}</td>
<td>塞入程式碼</td>
</tr>
<tr>
<td>{@docRoot}</td>
<td>將會被展開替換成文件的最上層目錄，需要link到本機相對路徑檔案時很好用</td>
</tr>
<tr>
<td>@deprecated 作廢說明文字</td>
<td>作廢的API說明</td>
</tr>
<tr>
<td>{@inheritDoc}</td>
<td>當函數宣告的參數、回傳值或是expcetion沒說明時，這邊可以指定從那邊繼承。這些繼承關係也有些預設選項，一言難盡。這邊先跳過，有興趣的請自行找資料。</td>
</tr>
<tr>
<td>@param 參數名稱 說明</td>
<td>請望文生義</td>
</tr>
<tr>
<td>@return 回傳值說明</td>
<td>請望文生義</td>
</tr>
<tr>
<td>@{value}</td>
<td>顯示註解下方變數的常數</td>
</tr>
<tr>
<td nowrap="nowrap">@since 文字，通常是版號</td>
<td>指出該method, package, class等在哪個版號後有效</td>
</tr>
</table>




由於有些tag放到表格不易閱讀，獨立說明如下

* `{@link package_名稱.class_名稱#member_函數名稱 顯示在文件的字串}`
    * 在文件中產生package_名稱.class_名稱#member_函數名稱的link
* `{@linkplain package_名稱.class_名稱#member_函數名稱 顯示在文件的字串}` 
    * 在文件中產生package_名稱.class_名稱#member_函數名稱的link。但是顯示的文字不會有link
* `@exception class-名稱 exception_說明`
    * 應該可以望文生義，另外@throw 也是同樣效果 
* `@see` 家族，有三種
    * `@see "字串"` 
        * 請讀者參考字串
    * `@see <a href="URL#value">Link顯示名稱</a>` 
        * 請讀者參考連結
    *  `@see package_名稱.class_名稱#member_函數名稱 顯示名稱`
        * 建立本地連結到package_名稱.class_名稱#member_函數名稱


 
<a name="jdoc-comments-ex"></a>
### 套用 javadoc 範例

```text 最後的檔案和目錄結構
$ tree
.
├── com
│   └── test
│       └── test.java
├── config.properties
├── Makefile
├── overview.html
├── package-info.java
└── pdfdoclet-1.0.3
    ├── build.xml
       ....
    └── jar
        └── pdfdoclet-1.0.3-all.jar
```

```java 加了javadoc的程式碼
package com.test;

/**
 * A example to test javadoc tags
 * <p>
 * @see mul
 * @see test.mul#multiply
*/
public class test {
    /**
     * multiply class, used by {@link test class test}
    */
    public class mul {
        /**
         * @param i operand 1
         * @param j operand 2
         * @return result of i * j
        */
        public int multiply(int i, int j) {
            return i * j;
        }
    }
    /**
     * Number of loop in multiplication: {@value}
    */
    public static final int MAX_NUM = 9;
    
    public static void main(String[] args) {
        test my_test = new test();
        my_test.hello();
    }

    private void hello() {
        mul my_mul = new mul();
        for (int i = 1; i <= MAX_NUM; i++) {
            for (int j = 1; j <= MAX_NUM; j++) {
                System.out.println(i + " x " + j + " = " + my_mul.multiply(i, j));
            }
        }
    }
}
```

最終產出的pdf檔[如link，應該沒病毒吧?](/files/javadoc-20150830/test_final.pdf)


<a name="jdoc-ref"></a>
## 參考資料

* [javadoc - The Java API Documentation Generator](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/javadoc.html)
* [How to Write Doc Comments for the Javadoc Tool](http://www.oracle.com/technetwork/articles/java/index-137868.html)
* [Running pdfdoclet](http://pdfdoclet.sourceforge.net/running.html)
