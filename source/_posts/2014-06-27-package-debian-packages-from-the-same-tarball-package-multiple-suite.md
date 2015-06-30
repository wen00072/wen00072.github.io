---
layout: post
title: '[Debian套件打包] 從同一套tarball打包多個套件'
date: 2014-06-27 07:25
comments: true
categories: [Debian]
---
## 緣由
使用apt套件管理可以看到，安裝函式庫通常是使用apt-get install libxxx。但是如果編譯時就會需要額外安裝libxxx-**dev**套件。該套件通常就是多了靜態函式庫以及header files。由於libxxx和libxxx-dev應該從同一包tarball產生，好奇之餘整理一下如何從一包tarball產生多個套件的方式。

## 步驟

* 更改debian/control，加入新的打包名稱，以及相依套件。
	* 範例：原本的套件是libxxx，那麼舊新增libxxx-dev，並且和相依於libxxx。
* 新增debian/`套件名稱.install`，並且寫套件想要安裝的檔案。
	* 範例：debian/libxxx.install和debian/libxxx-dev.install

## 測試
直接使用[之前的測試方法](http://wen00072.github.io/blog/2014/05/28/package-deb-packages-loose-style)裏面的套件，照上面的方式修改。

```text debian/control新增testautotools-dev套件
16,21d15
< 
< Package: testautotools-dev
< Architecture: any
< Depends: ${shlibs:Depends}, ${misc:Depends} testautotools
< Description: <insert up to 60 chars description>
<  <insert long description, indented with spaces>
```

另外兩個install 檔案列出如下
```text debian/testautotools.install
usr/lib/*.so.0*
usr/bin/my_test
```

```text debian/testautotools-dev.install
usr/lib/*.a
usr/lib/*.so
usr/lib/*.la
usr/include
```

更改後跑`dpkg-buildpackage -uc -us`產生的檔案如下

```text 產生的檔案列表
testautotools_0-1.dsc
testautotools_0-1_amd64.changes  
testautotools_0-1_amd64.deb      
testautotools-dev_0-1_amd64.deb
testautotools_0-1.debian.tar.gz
```
