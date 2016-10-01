---
layout: post
title: '從Autotools tarball打包deb套件: 不嚴謹style'
date: 2014-05-28 10:08
comments: true
categories: [Debian]
---
deb 檔案是Debian系列安裝套件檔案格式。這次將示範使用不嚴謹的兩種方式將一個auto tool打包的tarball打包成deb檔案。這兩種方式的差別只有在產生套件相關的source code tarball的方式不同。

**注意，這些方法完全沒有去更動Debain相關設定檔，也就是說很多細節都被省略而使用預設設定**
## 目錄
* [概論](#ov)
* [方法一](#m1)
* [方法二](#m2)
* [參考資料](#ref)

---
<a name="ov"></a>
## 概論
簡單敘述產生deb套件流程如下

* 取得tarball，依照Debian規範設定目錄名稱
* 解壓縮後，執行`dh_make`在解壓縮目錄下面產生debian目錄
* 更改debian目錄下面的設定
* 執行`dpkg-buildpackage`產生deb檔和其他檔案

debian目錄下面雖然我測試時偷懶都不改，不過還是值得提一下。

* 重要檔案
	* control – 套件metadata如相依性，套件官方網站，套件描述等
	* rules – 編譯套件的規範
	* copyright – 版權宣告，預設為GPL2。可以自行更動。另外dh-make可以下參數指定版權宣告如bsd等
	* changelog – 套件更動訊息
* 其他如安裝script，想要apply的patch等

從一個autotool的tarball xxx.tar.gz經過打包工具後最少要產生下面四個檔案：

* xxx_原始套件版本號-Debian套件版本號.dsc
	* 套件的metadata
* xxx_原始套件版本號.orig.tar.gz
	* 原始tarball
* xxx_原始套件版本號-Debian套件版本號.debian.tar.gz
	* 前面dh_make產生的debian目錄壓縮檔
* xxx_原始套件版本號-Debian套件版本號_平台.deb
	* 生成的套件
* xxx_原始套件版本號-Debian套件版本號_平台.changes
	* 描述整個產生的檔案資訊，要上傳到Debian套件主機會用到。

以下是使用[之前](http://wen00072.github.io/blog/2014/05/13/study-on-gnu-build-system-autotools)的auto tools為範例產生出來的檔案列表：這邊我們可以看到

* 原始套件版本號為`0`
* Debian套件版本號為`1`

```text Two level tree view
$ tree -L 2
.
├── testautotools-0
│   ├── aclocal.m4
│   ├── config.guess
│   ├── config.h
│   ├── config.h.in
│   ├── config.log
│   ├── config.status
│   ├── config.sub
│   ├── configure
│   ├── configure.ac
│   ├── debian
│   ├── depcomp
│   ├── include
│   ├── install-sh
│   ├── libs
│   ├── libtool
│   ├── ltmain.sh
│   ├── Makefile
│   ├── Makefile.am
│   ├── Makefile.in
│   ├── missing
│   ├── src
│   └── stamp-h1
├── testautotools_0-1_amd64.changes
├── testautotools_0-1_amd64.deb
├── testautotools_0-1.debian.tar.gz
├── testautotools_0-1.dsc
└── testautotools_0.orig.tar.gz
```

---
<a name="m1"></a>
## 方法一

* 在原本的auto tools開發套件目錄中`make dist`產生tarball，搬移到測試的空目錄
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
<a name="m2"></a>
## 方法二

* 在原本的auto tools開發套件目錄中`make dist`產生tarball
* 直接解壓縮tarball 到測試的空目錄
* **務必要確認目錄名稱要符合`xxx-原始套件版本號`的格式！不符合請自行rename**
* 切換到解壓縮目錄
* `echo -e "\n" | dh_make -s --createorig` 
	* `-s`表示tarball產生全部的binary都會放在同一份deb檔案
  * `--createorig`會讓工具幫你產生orig的tarball
  * 執行完畢會產生debian目錄，存放和套件有關的metadata，應該要手動確認修改資料，如套件說明、原來軟體官方網頁、聯絡方式等。
* `dpkg-buildpackage`
	* 產生套件相關檔案，因為無法sign所以不嚴謹。更嚴謹可以使用`debuild`，它會做許多額外的檢查，這種情況下我的測試tarball是無法通過`debuild`檢查的。自用可以加入`-uc -us`參數省略sign。

---
<a name="ref"></a>
## 參考資料

* [Debian Packaging Tutorial (PDF)](http://www.debian.org/doc/packaging-manuals/packaging-tutorial/packaging-tutorial)
* [deb (file format)](http://en.wikipedia.org/wiki/Deb_%28file_format%29)
* [DebPackaging(正體中文)](http://wiki.ubuntu-tw.org/index.php?title=DebPackaging)
