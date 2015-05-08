---
layout: post
title: '使用debootstrap 在本機中操作其他Ubuntu 版本 (命令列)'
date: 2015-01-17 01:00
comments: true
categories: [Linux, debootstrap]
---
好吧，這個標題我也不滿意，不過就將就一下吧。

簡單來說在Ubuntu內只要可以找得到的distribution，你就有辦法讓它在你的Ubuntu使用。使用順序如下：

## 下載你想要執行的套件
命令：

```
debootstrap --arch=你要跑的target平台 --variant=minbase 你要跑的版本代號 自己PC的版本代號 [mirror site]
```

要怎麼知道arch有哪些呢，你可以連到任意一個Ubuntu archive中的ubuntu/dists/[版本代號]/main中就可以看到了。可以看[範例網頁](http://archive.ubuntu.com/ubuntu/dists/trusty/main/)，這邊可以看到有`i386`和`amd64`兩種架構。

而版本代號可以在任意一個Ubuntu archive中的ubuntu/dists/看到。[範例網頁](http://archive.ubuntu.com/ubuntu/dists)可以看到10.04 (lucid), 12.04 (precise)等。

另外--variant問男人可以看到有
* minbase
	* 最少安裝
* buildd
	* 多安裝build code需要的套件
* fakechroot
	* 只安裝不需要root的套件 (怪怪的？？）
* scratchbox
	* 看不懂，跳過

我使用的範例如下：我要在x86-64位元的Ubuntu 14.04 (trusty) 下面安裝32位元(i386)的10.04 (lucid)最少套件的話，就會使用下面的指令。

```
$ sudo debootstrap --arch=i386 --variant=minbase lucid trusty ftp://ftp.tku.edu.tw/ubuntu/
I: Retrieving Release 
I: Retrieving Release.gpg 
I: Checking Release signature
...
I: Unpacking apt...
I: Configuring the base system...
I: Configuring apt...
I: Configuring libc-bin...
I: Base system installed successfully.
```

debootstrap會把下載並解開的套件放在你的指定的target名稱目錄中，這次範例使用trusty，所以我們看看trusty目錄有什麼東西？

```
$ cd trusty
$ tree -L 1 -d
.
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── mnt
├── opt
├── proc
├── root
├── sbin
├── selinux
├── srv
├── sys
├── tmp
├── usr
└── var
```

可以看到他就是一個root file system。接下來要做的就是。

## 使用chroot切換到用debootstrap安裝的root file system
先確認我電腦上的Ubuntu版本
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.1 LTS
Release:	14.04
Codename:	trusty

$ file /bin/ls
/bin/ls: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=64d095bc6589dd4bfbf1c6d62ae985385965461b, stripped
```

切換指令如下

```
$ cd trusty # 假設你還沒切換過去
$ sudo mount --bind /dev ./dev
$ sudo mount --bind /dev/pts ./dev/pts
$ sudo mount --bind /proc ./proc
$ sudo mount --bind /sys ./sys
$ sudo chroot .
```

好啦，口說無憑，我們先來看看是不是真的換到lucid，32位元版本吧

```
## apt-get install lsb-release # 因為最少安裝所以沒有lsb_release
Reading package lists... Done
Building dependency tree... Done
...

## lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 10.04 LTS
Release:	10.04
Codename:	lucid

## file /bin/ls
/bin/ls: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.15, stripped
```

這東西能做什麼呢？有時候開發套件需要在不同的Ubuntu版本測試，這時候後這招頂著先應該會比用VM再來得有效率一些。

另外要注意的是，我測試的時候自己的電腦應該有裝過很多東西了。這表示你要按表操課可能會因為缺乏套件所以不會成功，例如debootstrap可能就要自行安裝了我猜。

嘛，這就是人參，出現這種狀況就當作磨練吧。

## 參考資料

* man debootstrap
