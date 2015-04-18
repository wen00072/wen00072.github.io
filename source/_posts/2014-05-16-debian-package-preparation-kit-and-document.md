---
layout: post
title: 'Debian打包準備套件和文件'
date: 2014-05-16 10:08
comments: true
categories: [debian, C, Linux]
---
看[Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/start.en.html)順便整理一下Debain官方文件建議打包套件可以安裝的[套件](#pkg)和[文件](#doc)。

<a name="pkg"></a>
## 套件

* build-essential
* autoconf
* automake
* autotools-dev
* debhelper
	* 協助產生套件skeleton
* dh-make
	* 協助產生套件skeleton
* devscripts
* fakeroot
	* 模擬root權限。
* file
* git
* gnupg
	* 簽核用
* lintian
	* 協助檢查Debian套件打包錯誤
* patch
* patchutils
* pbuilder
	* Debian 套件用的personal package builder
* perl
* python
* quilt
	* 協助管理大量的patch file
* xutils-dev
	* 通常是X11會用到的工具如抽出巨集。
* gpc
	* Pascal編譯器，依專案需要(Ubuntu 已經不maintain)
* gfortran 
	* Fortran編譯器，依專案需要
 
懶人包如下：（不包含gpc）

```text 
sudo apt-get install build-essential autoconf automake autotools-dev debhelper dh-make devscripts fakeroot file git gnupg lintian patch patchutils pbuilder perl python quilt xutils-dev gfortran 
```
 
---
<a name="doc"></a>
## 文件

* [Debian Policy Manual](http://www.debian.org/doc/devel-manuals#policy)
* [Debian Developer's Reference](http://www.debian.org/doc/devel-manuals#devref)
* [Autotools Tutorial](http://www.lrde.epita.fr/~adl/autotools.html)
* [GNU Coding Standards](http://www.gnu.org/prep/standards/html_node/index.html)
* [Debian Packaging Tutorial](http://www.debian.org/doc/packaging-manuals/packaging-tutorial/packaging-tutorial)

## 其他資源

* [Debian Bug Tracking System](https://www.debian.org/Bugs/)

## 參考資料：

* [Debian New Maintainers' Guide - Chapter 1. Getting started The Right Way](https://www.debian.org/doc/manuals/maint-guide/start.en.html)
