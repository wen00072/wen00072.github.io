---
layout: post
title: '使用dpkg 查詢套件資訊 '
date: 2014-05-16 12:25
comments: true
categories: [Debian, Linux]
---
`dpkg`全名是Debian package，望文生義可以知道就是Debian套件管理程式。使用過Ubuntu/Debian 應該看過`deb`檔案。這邊列出幾個常用的功能。測試環境是Ubuntu 12.04.4

* 顯示套件狀態：
	* dpkg -s 套件名稱
* 顯示套件安裝到系統的檔案分佈
  * dpkg -L 套件名稱
* 列出套件說明
	* dpkg -l 套件名稱
* 搜尋檔案屬於那些套件
  * dpkg -S 搜尋的檔案如stdio.h
  
同場加碼：apt-file這個套件也可以查詢檔案屬於哪個套件。
