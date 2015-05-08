---
layout: post
title: '[Debian套件打包] debian目錄初探 (1)'
date: 2014-06-10 04:26
comments: true
categories: Debian
---
debian目錄存放描述deb套件的行為和資訊。也是從原始套件（通常是tarball)中產生deb套件檔案的重要資料。

從[Autotools tarball打包deb套件: 不嚴謹style](http://wen00072-blog.logdown.com/posts/201844-package-deb-packages-loose-style)有提到透過執行dh_make在解壓縮目錄下面產生debian目錄。這次的主題是debian目錄下面的資訊，主要是參考[Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)的第四章。

Debian套件強制規定debian目錄下面要有control, copyright, changelog三個檔案以及rules這個執行檔。分別討論說明如下。
---
## 目錄

* [debian/control檔](#ctrl)
	* [和其他相關套件的關係描述](#ctrl-rel)
* [debian/copyright檔](#crt)
* [debian/changelog檔](#cl)
* [debian/rules執行檔](#rl)
* [參考資料](#ref)

---
<a name="ctrl"></a>
## debian/control檔
從[Autotools tarball打包deb套件: 不嚴謹style](http://wen00072-blog.logdown.com/posts/201844-package-deb-packages-loose-style)產生的control內容為：
```text control
Source: testautotools
Section: unknown
Priority: extra
Maintainer: unknown <user@unknown>
Build-Depends: debhelper (>= 8.0.0), autotools-dev
Standards-Version: 3.9.2
Homepage: <insert the upstream URL, if relevant>
#Vcs-Git: git://git.debian.org/collab-maint/testautotools.git
#Vcs-Browser: http://git.debian.org/?p=collab-maint/testautotools.git;a=summary

Package: testautotools
Architecture: any
Depends: ${shlibs:Depends}, ${misc:Depends}
Description: <insert up to 60 chars description>
 <insert long description, indented with spaces>
```

詳細檔案規範可以參考：[Debian Policy Manual Chapter 5 - Control files and their fields](https://www.debian.org/doc/debian-policy/ch-controlfields.html)。這邊只列出幾點說明

* `Source`: 套件名稱
* `Section`: Debian套件分類有兩層
  * 第一層
    * `main` : 自由軟體，一般來說沒有特別指定的預設值。也就是說如果是`main/devel`直接使用`devel`就可以了。
  	* `non-free`
    * `contrib` : 基於`non-free`軟體衍生的軟體
  * 第二層
    * `admin`
    * `devel`
    * `doc`
    * `libs`
    * `mail`
    * `net`
    * `x11`
    * ...
* `Priority`:上傳時管理者有權限修改套件的優先權。
	* `required`
  * `important`
  * `standard`
  * `optional`: 新的套件**不和上面優先權套件衝突**時通常使用該權限。
  * `extra`: 新的套件**和上面優先權套件衝突**時使用該權限。
* `Maintainer`: Bug tracking system發生錯誤時聯絡的信件。
* `Build-Depends`: 從原始碼編譯成binary時需要的套件，例如libtool或是autoconf等。
* `Standards-Version`: 遵循的Debian Policy Manual版本
* `Homepage`:不解釋
* `Package`: 套件名稱
* `Architecture`:
	* `any`: 任何平台皆可，通常是需要編譯的套件。
  * `all`: 套件和平台無關，可能是文件、圖片會是直譯式的語言套件。
* `Description`: 80字元寬度，總字數不建議超過60字元。
* 套件原始碼VCS: 以git為例
	* `Vcs-Git`: Git repository URL
  * `Vcs-browser`: 瀏覽器可瀏覽的Git repository URL

---
<a name="ctrl-rel"></a>
### 和其他相關套件的關係描述

* `Depends`: Must have.
* `Recommends`: Nice to have. 安裝套件程式如apt-get或是dpkg會出現提示訊息詢問使用者是否安裝`Recommends`的套件。
* `Suggests`: apt-get或是dpkg不會提示這些套件，但是aptitude會詢問。
* `Pre-Depends`: 少用的關係描述，比`Depends`強。
* `Conflicts`: 除非衝突的套件有被反安裝，否則不與安裝。
* `Breaks`: 安裝該套件將會把`Breaks`裏面的套件搞爛。通常表示`Breaks`裏面的套件該被更新到新的版本。
* `Provides`: 因為版本相容考量或是其他因素，一個系統中可以由不同套件提供相同功能。這就是[Virtual packages](https://www.debian.org/doc/debian-policy/ch-binary.html#s-virtual_pkg)概念。不同的套件可以透過`Provides`提供類似的功能。
* `Replaces`
* 和其他相關套件的關係表示法為：
* `<<`: 低於
* `<=`: 低於或等於
* `=` : 等於
* `>=`: 高於或等於 
* `>>`: 高於
* 範例：
```
Depends: foo (>= 1.2), libbar1 (= 1.3.4)
Conflicts: baz
Recommends: libbaz4 (>> 4.0.7)
Suggests: quux
Replaces: quux (<< 5), quux-foo (<= 7.6)
```  

* 另外 ${shlibs:Depends}, ${perl:Depends}, ${misc:Depends}這樣的描述是給debhelper呼叫相對的計算相依性程式。有興趣的人可以`man dh_shlibdeps`或是`man dh_perl`等取得更多資訊。
  
---
<a name="crt"></a>
## debian/copyright檔
原始套件的版權聲明。除了版權聲明外，還有一些maintain資訊。maintainer可以直接修改dh_make產生的範本。

常見的版權可以透過`dh_make --copyright 版權(ex: gpl2, bsd)`自動產生debian/copyright檔。如果沒有原本套件的版本，可以搜尋`/usr/share/common-licenses/`下面是否有符合的版權宣告。再沒有就必須手動修正該檔了。

```text dh_make產生的範本
Format: http://dep.debian.net/deps/dep5
Upstream-Name: testautotools
Source: <url://example.com>

Files: *
Copyright: <years> <put author's name and email here>
           <years> <likewise for another author>
License: <special license>
 <Put the license of the package here indented by 1 space>
 <This follows the format of Description: lines in control file>
 .
 <Including paragraphs>

# If you want to use GPL v2 or later for the /debian/* files use 
# the following clauses, or change it to suit. Delete these two lines
Files: debian/*
Copyright: 2014 unknown <user@unknown>
License: GPL-2+
 This package is free software; you can redistribute it and/or modify
 it under the terms of the GNU General Public License as published by
 the Free Software Foundation; either version 2 of the License, or
 (at your option) any later version.
 .
 This package is distributed in the hope that it will be useful,
 but WITHOUT ANY WARRANTY; without even the implied warranty of
 MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 GNU General Public License for more details.
 .
 You should have received a copy of the GNU General Public License
 along with this program. If not, see <http://www.gnu.org/licenses/>
 .
 On Debian systems, the complete text of the GNU General
 Public License version 2 can be found in "/usr/share/common-licenses/GPL-2".

# Please also look if there are files or directories which have a
# different copyright/license attached and list them here.
```

---
<a name="cl"></a>
## debian/changelog檔
dpkg或是其他套件管理工具會參考這檔案取得版本號碼等資訊。另外這個檔案會被存放到`/usr/share/doc/套件名稱/changelog.Debian.gz`。

詳細規範可以參考[Debian Policy Manual, 4.4 "debian/changelog"](https://www.debian.org/doc/debian-policy/ch-source.html#s-dpkgchangelog)。簡單說明如下

* 第一行提供下面資訊:
	* 套件名稱。
  * 套件版本。
  * 套件應該上傳到FTP的dists的哪個目錄，如unstable或是testing，新的套件通常是unstable。
  * 緊急度：新的套件通常是low  。
* 接下來是以`＊`開頭的更動說明。第一次的更動，可以看到`ITP`。ITP是Intent To Package，新的套件會在[Debian bug tracking system](https://www.debian.org/Bugs/)發出需求。所以會有對應的issue number。詳細可以參考[Work-Needing and Prospective Packages](https://www.debian.org/devel/wnpp/)需要maintainer或是期望的新套件認領流程。

```text dh_make產生的範本
testautotools (0-1) unstable; urgency=low

  * Initial release (Closes: #nnnn)  <nnnn is the bug number of your ITP>

 -- unknown <user@unknown>  Tue, 10 Jun 2014 12:42:30 +0800
```

---
<a name="rl"></a>
## debian/rules執行檔
這是dpkg-buildpackage會呼叫到的script，這些script本質上就是Makefiles。需提供下面的target:

* 必備target
  * clean
  * build
  * build-arch
  * build-indep
  * binary
  * binary-arch
  * binary-indep
* 可有可無的target
  * install
	* get-orig-source
  
很恐怖嗎？基本上dh_make完在debian就會順便幫你產生一個預設的rules執行檔範例。基本上就是呼叫dh執行檔，列出如下
```makefile debian/rules
#!/usr/bin/make -f
# -*- makefile -*-
# Sample debian/rules that uses debhelper.
# This file was originally written by Joey Hess and Craig Small.
# As a special exception, when this file is copied by dh-make into a
# dh-make output file, you may use that output file without restriction.
# This special exception was added by Craig Small in version 0.37 of dh-make.

# Uncomment this to turn on verbose mode.
#export DH_VERBOSE=1

%:
	dh $@ 
```

dh是一個wrapper，會根據參數呼叫其他的script如dh_testdir, dh_auto_clean, dh_clean等。另外產生binary時手冊上面使用`fakeroot`模擬root權限，主要是將要安裝到系統的執行檔、函式庫、設定檔等安裝到**假的**root filesystem然後打包成deb檔。手冊的範例如下：

* `fakeroot debian/rules binary`
* `fakeroot debian/rules binary-arch`

請注意的dh_make產生的rules範本在面對複雜的套件可能會有非預期的錯誤，此時人肉修改rules script將會是無可避免的事。

**關於rules和dh的詳細用法這邊偷懶省略，請讀者自行參考手冊。**

---
<a name="ref"></a>
## 參考資料

* [Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)
* [Debian Policy Manual Chapter 5 - Control files and their fields](https://www.debian.org/doc/debian-policy/ch-controlfields.html)
