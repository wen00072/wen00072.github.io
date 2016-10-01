---
layout: post
title: "Ubuntu 14.04下使用dbdoclet 產生pdf手冊"
date: 2015-08-27 21:48:07 +0800
comments: true
categories: [javadoc, dbdoclet]
---
Sept/1/2015 更新：下面的方法有個問題，目錄的頁面會有從`ii`開始的羅馬數字頁碼。當known issue吧，猜測要從xsl修正，`/usr/share/xml/docbook/stylesheet/docbook-xsl/fo/docbook.xsl`應該有線索。

## 前言

Javadoc是一個掃描Java原始碼自動產生網頁文件的工具。除此之外，Javadoc還提供了API讓人實作doclet的功能。Javadoc把產出資料餵給doclet，doclet自行決定該怎麼處理，如

* 轉成pdf文件
* 轉成LaTeX文件
* 轉成docbook文件
* ...

回到主題，為了要產生PDF手冊，並且需要符合下面的要求：

* 封面
* 版權宣告、版本變動紀錄
* 目錄
* SDK簡介，包含方塊圖
* API使用方式
* 範例

## doclet 評估

目前找到幾個免費產生pdf的doclet，簡單的試玩結果如下

* pdfdoclet
    * 直接輸出pdf，沒有目錄、針對手冊需求增加修改資料極度困難
* AurigaDoclet
    * 和樓上的主要差別在多了目錄
* ltxdoclet
    * 產出LaTeX檔案，可透過pdflatex輸出pdf。有亂碼，美觀程度不如上面兩個
* TeXdoclet
    * 產出LaTeX，不支援pdflatex，直接放棄。
* dbdoclet
    * 產出docbook，透過一些方式可以達到目標，雖然美觀程度還是比不上pdfdoclet和AurigaDoclet，但是比doxygen和ltxdoclet好，最後用這個產生手冊。
* 其他族繁不及記載    
## 使用dbdoclet 產生pdf手冊步驟

詳細指令大家自行估狗，這邊單純提供key word。大略步驟如下：

1. 透過任何方式(openoffice, LaTeX, etc)產生封面、版權宣告的pdf的檔案，假設為`cover.pdf`。
2. 撰寫針對你要產出手冊的XSL style sheet，我寫了三個東西。這邊要注意的是建議直接在XSL import docbook.xsl。
    * 人肉產生page break的指令，原因docbook不夠聰明，有些東西還是得人肉加page break。如一個段落之後就是範例程式的程式碼。這時候範例程式的程式碼放在頁面開頭的話，閱讀應該比較輕鬆。
    * header的描述
    * footer的描述，可塞入圖片
3. 以docbook格式撰寫你要塞的內容如
    * 簡介
    * 範例程式
4. 透過javadoc執行dbdoclet，我這邊跑完會產生`Reference.xml`。
5. 使用sed或是你順手的字串工具
    * 修改產出的`Reference.xml`，找對的地方include自己寫的內容
    * 如果有必要，修改針對你要產出手冊的XSL style sheet
6. 透過`xsltproc`產生fo檔。幾點要注意的
    * 要加--xinclude，不然程式不會去include你自己寫的xml
    * /usr/share/xml/docbook/stylesheet/docbook-xsl/fo/docbook.xsl不用指定，因為你自己寫的XSL style sheet已經import了，這個問題卡了我一陣子時間。
7. 使用`fop`將fo轉成pdf檔，假設為`draft.pdf`。
8. 使用`pdftk`做
    * 把你的`draft.pdf`目錄前面頁面全部幹掉，另存新檔如`temp.pdf`
    * 把`cover.pdf`和`temp.pdf`合併，就會有封面和含目錄以及你設定的手冊了。
