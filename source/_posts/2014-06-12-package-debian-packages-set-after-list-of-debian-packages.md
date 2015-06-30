---
layout: post
title: '[Debian套件打包] 設定好debian目錄後的打包'
date: 2014-06-12 03:28
comments: true
categories: Debian
---
這邊探討打包的工具。有dpkg-buildpackage, debuild, pbuilder, 和git-buildpackage等工具，分別討論如下：

如果單獨要rebuild套件，可以使用下面的指令。
`fakeroot debian/rules binary`
`fakeroot debian/rules build`

---
## dpkg-buildpackage

[從Autotools tarball打包deb套件: 不嚴謹style](http://wen00072.github.io/blog/2014/05/28/package-deb-packages-loose-style)提到產生並設定好debian後，可以透過dpkg-buildpackage自動打包成deb檔案。而dpkg-buildpackage會做

* 套件大掃除 (debian/rules clean)
* 整理source檔案如apply patch (dpkg-source -b)
* 編譯 (debian/rules build)
* 安裝產生的檔案到**虛擬** root (fakeroot debian/rules binary)並打包
  * 如/usr/bin/xxx，/usr/share/doc/xxx, 等
* 使用gpg sign .dsc 檔案
* 產生並sign .changes 檔案

---
## debuild
除了dpkg-buildpackage外，也有更嚴謹的套件產生工具`debuild`可以使用。之前的測試用程式，用debuild全部會失敗，因為他提供了更多的檢查。列舉一些錯誤訊息如下。

```
E: testautotools: copyright-contains-dh_make-todo-boilerplate
W: testautotools: readme-debian-contains-debmake-template
E: testautotools: description-is-dh_make-template
E: testautotools: maintainer-address-malformed unknown <user@unknown>
E: testautotools: section-is-dh_make-template
W: testautotools: superfluous-clutter-in-homepage <insert the upstream URL, if relevant>
W: testautotools: bad-homepage <insert the upstream URL, if relevant>
```

和dpkg-buildpackage一樣，自幹可以下`-uc -us`跳過sign的步驟。另外你也可以自訂一些初始參數，存到/etc/devscripts.conf或是~/.devscripts下面。手冊建議可以加入：

```
DEBSIGN_KEYID=你的_GPG_keyID
DEBUILD_LINTIAN_OPTS="-i -I --show-overrides"
```

---
## pbuilder
pbuilder透過pbuilder套件中的image以及chroot，使用乾淨的環境來產生並測是套件。如此一來可以確認是否debian目錄下面的設定是否真的可以在這些乾淨的環境被編譯和安裝。

照手冊整理使用方式：**我全部沒測過**，當作以後的課題好了。

* 環境設定
* 建立image
* 使用pbuilder打包source package (套件名稱.orig.tar.gz, 套件名稱.debian.tar.gz, 套件名稱.dsc)
* sign

---
## 透過版本控制版本來 buildpackage

* git-buildpackage
* svn-buildpackage
* cvs-buildpackage

---
## Autobuilder
Autobuilder是Debian提供的服務，會自動rebuild原始套件，產生不同平台如X86, ARM專用的套件。

## 參考資料

* [Debian New Maintainers' Guide: Chapter 6. Building the package](https://www.debian.org/doc/manuals/maint-guide/build.en.html)
* [使用 pbuilder-dist/cowbuilder-dist 協助編譯及打包 Debian package](http://fourdollars.blogspot.tw/2011/07/pbuilder-distcowbuilder-dist-debian.html)
* [利用 Cowbuilder 建立不同的debian Package 編譯環境](http://hychen.wuweig.org/blog/2011/02/13/li-yong-cowbuilder-jian-li-bu-tong-de-debian-package-bian-yi-huan-jing/)
* [四元的學習筆記](http://wi.fd.idv.tw/debian/package)
