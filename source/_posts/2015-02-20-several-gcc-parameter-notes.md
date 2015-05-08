---
layout: post
title: 'gcc的幾個參數筆記'
date: 2015-02-20 14:34
comments: true
categories: [C, Linux]
---
整理一下，反正一定會忘記，就放在這邊，也許以後會再新增

* gcc --save-temp hello.c
	* gcc中間產生的檔案通通存起來
* gcc -dumpspecs
	* 印spec，<font color="red">不過我完全看不懂</font>
* gcc -dumpmachine
  * 印出平台
  
其他關於巨集部份請參考[這邊](http://wen00072-blog.logdown.com/posts/146624-talk-about-c-macros)和[這邊](http://wen00072-blog.logdown.com/posts/183770-makefile-header-file-dependency-issues)，很臭很長，抱歉。
