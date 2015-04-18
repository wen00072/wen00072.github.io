---
layout: post
title: '學習: Markdown語法簡介'
date: 2013-09-15 13:53
comments: true
categories: [Markdown, GitHub Flavored Markdown]
---
對於技術文件寫作來說，能夠省去複雜語法和繁複操作，同時又能方便的寫出好的架構和論述很重要。因此我需要
- 直接在本文中加入簡單的Tag語法省去滑鼠操作時間
- 明顯地文章段落顯示方式
- 方便的支援巢狀條列語法
- 美觀地顯示程式語言代碼

為了做出好的文件，了解並學習Markdown語法是必經的路程。就讓我邊查資料現學現賣吧。

<a name="content"></a>
## 目錄

- [參考資料](#ref)
- [語法說明](#syntax)
    - [標題](#header)
    - [Bullet List](#blist)
    - [數字 List](#nlist)
    - [撒尿牛丸 List](#mlist)
    - [分隔線](#seperater)
    - [字型](#font)
    - [連結](#link)
    - [Code](#code)
    - [引用](#quote)
    - [段落用斷行](#pbreak)
- [如何做出目錄](#content_how)
- [GitHub Flavored Markdown語法](#gfm_syntax)
     - [刪除線](#del_line)
     - [Fenced code blocks](#fcb)
        - [GitHub Flavored Markdown語法](#gfmfcb)
        - [Octopress語法](#opfcb)

<a name="ref"></a>
## 參考資料

 - [Markdown 官方網站(英)](http://daringfireball.net/projects/markdown/)
 - [Markdown Wiki說明(英)](http://en.wikipedia.org/wiki/Markdown)
 - [Stackoverflow Markdown Help(英)](http://stackoverflow.com/editing-help)
 - [Markdown Cheatsheet(英)](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)
 - [GitHub Flavored Markdown(英)](https://help.github.com/articles/github-flavored-markdown)
 - [Octopress: Sharing Code Snippets(英)](http://octopress.org/docs/blogging/code/)
 - [自由軟體鑄造場的Markdown簡介(中)](http://www.openfoundry.org/tw/resourcecatalog/Program-Development/Markup-Languages/markdown)
 - [Markdown 語法說明(中)](http://markdown.tw/)

<a name="syntax"></a>
## 語法說明
**注意！有些效果可以透過不同語法達成。**
**本文只列出我感興趣的語法，並沒有列出完整的語法。**
<a name="header"></a>
### 標題
	## 主標題
	### 次要標題
	#### 更次要標題，最多到6個#
## 主標題
### 次要標題
#### 更次要標題，最多到6個#

<a name="blist"></a>
### Bullet List
	- 第一層List
	    - 第二層List，和第一層隔4個空白字元或是tab
	        - 第三層List，和第二層隔4個空白字元或是tab

- 第一層List
    - 第二層List，和第一層隔4個空白字元或是tab
        - 第三層List，和第二層隔4個空白字元或是tab

<a name="nlist"></a>
### 數字 List
	1. 第一層List
	    1. 第二層List，和第一層隔4個空白字元或是tab
	        1. 第三層List，和第二層隔4個空白字元或是tab

1. 第一層List
    1. 第二層List，和第一層隔4個空白字元或是tab
        1. 第三層List，和第二層隔4個空白字元或是tab
   
<a name="mlist"></a>
### 撒尿牛丸 List
	1. 第一層List
	    - 第二層List，和第一層隔4個空白字元或是tab
	        1. 第三層List，和第二層隔4個空白字元或是tab

1. 第一層List
    - 第二層List，和第一層隔4個空白字元或是tab
        1. 第三層List，和第二層隔4個空白字元或是tab

<a name="seperater"></a>        
### 分隔線
	***
***

<a name="font"></a> 
### 字型
	*斜體字，字和星號中間不能空白*
*斜體字，字和星號中間不能空白*

	**粗體字，字和星號中間不能空白**
**粗體字，字和星號中間不能空白**

<a name="link"></a> 
### 連結
	[本站](http://wen00072.github.io)
[本站](http://wen00072.github.io)

<a name="code"></a> 
### Code
- 因為要顯示程式碼，所以不會Markdown的語法
    - 方式
        1. 行首以tab或8個空白開始
        2. 使用成對的`把程式碼包起來

　- 這個bullit 效果沒有出來
　- 這個bullit 效果沒有出來

　̀我是程式碼　̀
`我是程式碼`

<a name="quote"></a> 
### 引用
	> 第一層引用
	>> 第二層引用
  
> 第一層引用
>> 第二層引用

<a name="pbreak"></a>
### 段落用斷行

	行尾沒有空白字元
	第二行
  
行尾沒有空白字元
第二行

	行尾加上兩個以上空白字元  
	第二行
  
行尾加上兩個以上空白字元  
第二行

<a name="content_how"></a>
## 如何做出目錄

1. 在文件中嵌入HTML的anchor tag
2. 同樣的文件中指定連結

```
<a name="test"></a>
[連連看](#test)
```
<a name="test"></a>
[連連看](#test)

<a name="gfm_syntax"></a>
## GitHub Flavored Markdown

<a name="del_line"></a>
### 刪除線
	~~打我啊笨蛋，~和句子中間一樣不能有空白~~
~~打我啊笨蛋，~和句子中間一樣不能有空白~~

<a name="fcb"></a>
###  Fenced code blocks
<a name="gfmfcb"></a>
#### GitHub Flavored Markdown語法
使用<pre>```</pre>把程式包起來
```
 ```
 #include <stdio.h>
 int main(void)
 { 
	printf("Hello World\n");
 	return 0;
 }
 ```
```
```
 #include <stdio.h>
 int main(void)
 { 
	printf("Hello World\n");
 	return 0;
 }
```
<a name="opfcb"></a>
#### Octopress的語法

[Octopress的語法](http://octopress.org/docs/blogging/code/)讓使用者提供

- 指定語言Highlight
- 標題
- URL
- URL文字說明

```
 ```c 標題 http://wen00072-blog.logdown.com/ URL文字說明
 #include <stdio.h>
 int main(void)
 { 
	printf("Hello World\n");
 	return 0;
 }
 ```
```
```c 標題 http://wen00072-blog.logdown.com/ URL文字說明
 #include <stdio.h>
 int main(void)
 { 
	printf("Hello World\n");
 	return 0;
 }
```
