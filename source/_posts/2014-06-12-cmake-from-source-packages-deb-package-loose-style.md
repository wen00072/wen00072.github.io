---
layout: post
title: '從CMake原始碼打包deb套件: 不嚴謹style'
date: 2014-06-12 03:38
comments: true
categories: [Debian]
---
先承認這是拖檯錢，本來以為用法會和autotool差很多，結果用法完全相同。所以下面的也是剪貼原來的文件。

## 方法一

* 把tarball 從xxx.tar.gz更改成xxx_版本號.orig.tar.gz
* 解壓縮
* **務必要確認目錄名稱要符合`xxx-原始套件版本號`的格式！不符合請自行rename**
* 切換到解壓縮目錄
* `echo -e "\n" | dh_make -s`
	* `-s`表示tarball產生全部的binary都會放在同一份deb檔案
  * 執行完畢會產生debian目錄，存放和套件有關的metadata，應該要手動確認修改資料，如套件說明、原來軟體官方網頁、聯絡方式等。
* `dpkg-buildpackage`
	* 產生套件相關檔案，因為無法sign所以不嚴謹。更嚴謹可以使用`debuild`，它會做許多額外的檢查，這種情況下我的測試tarball是無法通過`debuild`檢查的。自用可以加入`-uc -us`參數省略sign。

---
## 方法二

* 直接解壓縮tarball 到測試的空目錄
* **務必要確認目錄名稱要符合`xxx-原始套件版本號`的格式！不符合請自行rename**
* 切換到解壓縮目錄
* `echo -e "\n" | dh_make -s --createorig` 
	* `-s`表示tarball產生全部的binary都會放在同一份deb檔案
  * `--createorig`會讓工具幫你產生orig的tarball
  * 執行完畢會產生debian目錄，存放和套件有關的metadata，應該要手動確認修改資料，如套件說明、原來軟體官方網頁、聯絡方式等。
* `dpkg-buildpackage`
	* 產生套件相關檔案，因為無法sign所以不嚴謹。更嚴謹可以使用`debuild`，它會做許多額外的檢查，這種情況下我的測試tarball是無法通過`debuild`檢查的。自用可以加入`-uc -us`參數省略sign。
