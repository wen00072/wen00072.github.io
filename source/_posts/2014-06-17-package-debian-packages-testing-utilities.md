---
layout: post
title: '[Debian套件打包] 打包的測試驗證工具簡介'
date: 2014-06-17 06:36
comments: true
categories: Debian
---
Debian New Maintainers' Guide提供了一些打包套件時出錯的建議

* 檢查quilt格式的patch是否有問題，可能是patch出問題、或是對應的套件工具出問題。後者的情況可以檢查並修正對應工具如dh-autoreconf或是更改source/option作為workaround。
* 測試安裝
	* 工具`sudo debi 你套件產生的.changes檔案名稱` 可以協助檢查是否有問題。**目前測試跑不起來，待釐清！！**
  * 要比對你打包的套件是否原本系統其他套件已經有**相同的檔名**。可以下載Contents-i386確認另外有網友建議更好用的方式`apt-file -D 你產生的套件`。不幸發生的話可以做
    * 更改自己套件中衝突的檔名
    * 套件control檔案設定會和自己衝突的套件
    * 設定alternatives方式 **To be studied!**
* 測試script
	* 使用dpkg測試install, remove, purge, 以及updage。手冊建議測試順序：
  		* 安裝上一版套件
      * upgrade成你的套件，並且順便測downgrade
      * purge上一版套件
      * 安裝你的套件
      * remove
      * 再安裝
      * purge
   * 手冊建議初版套件可以自己弄幾個dummy版本套件測試更新退版的狀況。
* 使用lintian測試驗證套件是否有問題
	* `lintian -i -I --show-overrides 你套件產生的.changes檔案名稱`
* 使用debc顯示套件content資訊，驗證套件是否有問題
	* `debc 你套件產生的.changes檔案名`
* 如果您打包的是某套件的新版，可以透過`debdiff`比對新舊版的dsc檔案和changes檔案
* 可以使用`interdiff`指令比對新舊套件的diff差別。
* `mc`工具可以提供更詳細的壓縮或是以打包的檔案檢視，如tarball或是deb檔。(Wen: mc很好用，大力推荐！)
  
  

## 參考資料

* [Debian New Maintainers' Guide: Chapter 7. Checking the package for errors](https://www.debian.org/doc/manuals/maint-guide/checkit.en.html)
