---
layout: post
title: '[Debian套件打包] 使用pbuilder驗證套件'
date: 2014-07-31 03:57
comments: true
categories: [Debian, pbuilder]
---
之前的[文章](http://wen00072-blog.logdown.com/posts/205819-package-debian-packages-set-after-list-of-debian-packages)有提到使用pbuilder驗證套件。最近抽空玩了一下。整理到這邊。

[回收以前文章]
pbuilder透過pbuilder套件中的image以及chroot，使用乾淨的環境來產生並測是套件。如此一來可以確認是否debian目錄下面的設定是否真的可以在這些乾淨的環境被編譯和安裝。

##  目錄：

* [環境設定並建立image](#env)
* [使用pbuilder打包source package](#pkg)
* [參考資料](#ref)

---
<a name="env"></a>
##  環境設定並建立image

* 測試環境
```text lsb_release -a畫面
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Debian
Description:	Debian GNU/Linux unstable (sid)
Release:	unstable
Codename:	sid
```

先使用下面指令產生乾淨的Linux環境所對應的tarball，之後會用chroot切換到該環境中。

* `sudo pbuilder create`

```text 執行畫面
$ sudo pbuilder create
W: /root/.pbuilderrc does not exist
I: Distribution is sid.
I: Current time: Thu Jul 31 12:44:44 CST 2014
I: pbuilder-time-stamp: 1406781884
I: Building the build environment
I: running debootstrap
...
I: new cache content grep_2.18-2_amd64.deb added
I: unmounting dev/pts filesystem
I: unmounting run/shm filesystem
I: unmounting proc filesystem
I: creating base tarball [/var/cache/pbuilder/base.tgz]
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build//27394 and its subdirectories
```
從log中可以看到處理的時候會暫時放在`/var/cache/pbuilder/build//27394`中，當執行結束的時候會刪除。

我們可以更進一步看一下`/var/cache/pbuilder`目錄：

```text /var/cache/pbuilder
#  tree -d /var/cache/pbuilder
.
|-- aptcache
|-- build
|-- ccache
|-- pbuildd
|-- pbuilder-mnt
|-- pbuilder-umlresult
`-- result
```

aptcache目錄可以看到裡面就是一堆deb檔案。我們可以合理的推論當測試十有需要額外安裝相依套件下載後會先存放在這邊。

接下來我們可以使用下面的方式玩一下

* `sudo pbuilder --login`

```text sudo pbuilder --login 執行畫面
$ sudo pbuilder --login
W: /root/.pbuilderrc does not exist
I: Building the build Environment
I: extracting base tarball [/var/cache/pbuilder/base.tgz]
I: creating local configuration
I: copying local configuration
I: mounting /proc filesystem
I: mounting /run/shm filesystem
I: mounting /dev/pts filesystem
I: policy-rc.d already exists
I: Obtaining the cached apt archive contents
I: entering the shell
File extracted to: /var/cache/pbuilder/build//10030

root@debian:/# 
```
重點是`File extracted to: /var/cache/pbuilder/build//10030`，所以我們可以在本機中去看`/var/cache/pbuilder/build/10030`，裡面就是一個完整的Linux root

```text 列出目錄：/var/cache/pbuilder/build/10030
##  ls /var/cache/pbuilder/build/10030
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
```

當我們離開pbuilder shell後，可以看到該root目錄也隨之消失。如此一來，可以確保所有的資料一開始都會是從`/var/cache/pbuilder/base.tgz`解出來最乾淨的狀態。

```text 離開pbuilder --login
root@debian:/# exit
logout
I: Copying back the cached apt archive contents
I: unmounting dev/pts filesystem
I: unmounting run/shm filesystem
I: unmounting proc filesystem
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build//10030 and its subdirectories
```

---
<a name="pkg"></a>
##  使用pbuilder打包驗證source package
我自己的方式是先用`debbuild -S`或是`dpkg-buildpackage -S`產生dsc檔案。在同一個目錄下面下下面的指令。

* `sudo pbuilder --build 你的套件.dsc`

當你原始套件debian/control的depend沒寫好，pbuilder乾淨的環境就不會安裝depend的套件而執行失敗。這時候你就要加入缺少的depend套件到debian/control。最後產生的套件會放在`/var/cache/pbuilder/result/`中

執行畫面如下：
```text sudo pbuilder --build testautotools_0-1.dsc
W: /root/.pbuilderrc does not exist
I: using fakeroot in build.
I: pbuilder: network access will be disabled during build
I: Current time: Thu Jul 31 15:28:32 CST 2014
I: pbuilder-time-stamp: 1406791712
I: Building the build Environment
...
I: Obtaining the cached apt archive contents
I: Installing the build-deps
 -> Attempting to satisfy build-dependencies
 -> Creating pbuilder-satisfydepends-dummy package
...
I: Extracting source
dpkg-source: warning: extracting unsigned source package (testautotools_0-1.dsc)
dpkg-source: info: extracting testautotools in testautotools-0
dpkg-source: info: unpacking testautotools_0.orig.tar.xz
dpkg-source: info: unpacking testautotools_0-1.debian.tar.xz
I: Building the package
I: Running cd tmp/buildd/*/ && env PATH="/usr/sbin:/usr/bin:/sbin:/bin" dpkg-buildpackage -us -uc  -rfakeroot
...
I: Copying back the cached apt archive contents
I: unmounting dev/pts filesystem
I: unmounting run/shm filesystem
I: unmounting proc filesystem
I: cleaning the build env 
I: removing directory /var/cache/pbuilder/build//18912 and its subdirectories
I: Current time: Thu Jul 31 15:29:09 CST 2014
I: pbuilder-time-stamp: 1406791749
```

---
<a name="ref"></a>
##  參考資料

* [PbuilderHowto ](https://wiki.ubuntu.com/PbuilderHowto)
* [Debian: Pbuilder](https://wiki.debian.org/Diaspora/Packaging/pbuilder)
* [Debian New Maintainers' Guide: Chapter 6. Building the package](https://www.debian.org/doc/manuals/maint-guide/build.en.html)
