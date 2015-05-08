---
layout: post
title: '[Meta-Package] Debian 打包系列'
date: 2014-06-08 06:56
comments: true
categories: Debian
---
meta這個字眼好像找不到精確的中文名詞。基本上就是指和一個「主題」有關的「東西」。舉例來說，在多媒體播放當中，「metadata」指的是附於媒體本身的屬性，如影片解析度、作品名稱等。而「meta-programming」指的是寫程式執行輸出結果是另外一段的程式原始碼。

而在Debian套件中meta package就是該套件本身不會包含任何套件，但是該套件會需要相依於一個或多個套件，因而安裝當該meta package就是安裝完一組套件。

回到主題，最近為了解Debian套件的產生方式做了一些study，因此整理相關連結放在這份文件當中。這樣應該也是一種meta package的概念吧。

## Debian套件資源

* [Debian打包準備套件和文件](http://wen00072-blog.logdown.com/posts/199275-debian-package-preparation-kit-and-document)
* [使用dpkg 查詢套件資訊](http://wen00072-blog.logdown.com/posts/199271-study-on-dpkg)

## 編譯自動化

* [GNU Build System: Autotools 初探](http://wen00072-blog.logdown.com/posts/198783-study-on-gnu-build-system-autotools)
* [Libtool初探](http://wen00072-blog.logdown.com/posts/199470-study-on-the-libtool)
* [在Autotools使用Libtool編譯函式庫](http://wen00072-blog.logdown.com/posts/200534-autotools-to-use-libtool-compile-function-library)
* [CMake 初探](http://wen00072-blog.logdown.com/posts/202369-study-on-cmake)

## 打包方式

* [從Autotools tarball打包deb套件: 不嚴謹style](http://wen00072-blog.logdown.com/posts/201844-package-deb-packages-loose-style)
* [從CMake原始碼打包deb套件: 不嚴謹style](http://logdown.com/account/posts/205822-cmake-from-source-packages-deb-package-loose-style/preview)
* [[Debian套件打包] debian目錄初探 (1)](http://wen00072-blog.logdown.com/posts/205562-package-debian-packages-study-on-the-debian-directory)
* [[Debian套件打包] debian目錄初探 (2)](http://wen00072-blog.logdown.com/posts/205689-package-debian-packages-study-on-the-debian-directory-2)
* [[Debian套件打包] 設定好debian目錄後的打包](http://wen00072-blog.logdown.com/posts/205819-package-debian-packages-set-after-list-of-debian-packages)
* [[Debian套件打包] 打包的測試驗證工具簡介](http://wen00072-blog.logdown.com/posts/206365-package-debian-packages-testing-utilities)
* [[Debian套件打包] 如何更新現有套件並重新打包？](http://wen00072-blog.logdown.com/posts/206431-package-debian-packages-how-to-update-the-existing-suite)
* [[Debian套件打包] 從同一套tarball打包多個套件](http://wen00072-blog.logdown.com/posts/207636-package-debian-packages-from-the-same-tarball-package-multiple-suite)
* [[Debian套件打包] 使用pbuilder驗證套件 ](http://wen00072-blog.logdown.com/posts/212087-package-debian-packages-discussion-on-pbuilder)
* [[Debian套件打包] pbuilder 中如何安裝自己打包](http://wen00072-blog.logdown.com/posts/243406-package-debian-packages-using-local-debian-pbuilder-package)

## 其他

* [C語言中使用gettext](http://wen00072-blog.logdown.com/posts/202230-study-on-gettext)
* [quilt初探](http://wen00072-blog.logdown.com/posts/205375-study-on-the-quilt)
* [打包Debian 套件時加入自己的Patch方式](http://wen00072-blog.logdown.com/posts/204026-joined-his-patch-package-for-debian-packages)
