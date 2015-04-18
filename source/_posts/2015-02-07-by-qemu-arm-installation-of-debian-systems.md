---
layout: post
title: '在X86 Linux下透過Qemu安裝ARM的Debian系統'
date: 2015-02-07 00:40
comments: true
categories: 
---
安裝順序如下

## 測試環境
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.1 LTS
Release:	14.04
Codename:	trusty
```

## 安裝Qemu
```
$ sudo apt-get install qemu-system-arm
```

## 拉ARM安裝相關檔案
* 首先先來看Qemu有支援那些ARM平台
```
$ qemu-system-arm -machine help
Supported machines are:
versatileab          ARM Versatile/AB (ARM926EJ-S)
versatilepb          ARM Versatile/PB (ARM926EJ-S)
lm3s811evb           Stellaris LM3S811EVB
...
```

安裝ARM版本Debian需要

* kernel
* initrd
* ISO
首先到提供Debian下載的網站中的debian/dists找一個你想安裝的版本，我選了7.8 (Wheezy)。
假設Debain下載網站叫host

先抓平台相關的kernel和initrd，路徑如下
<pre>http://host/debian/dists/Debian7.8/main/installer-armel/20130430/images/</pre>
下面有不同的ARM平台，還記得上面`qemu-system-arm -machine help`，請和這邊目錄下的比對，挑一個順眼的。我使用
versatile，所以就切到下面的目錄
<pre>http://host/debian/dists/Debian7.8/main/installer-armel/20130430/images/versatile/netboot/</pre>
把下面的兩個檔案拉下來

* initrd.gz  
* vmlinuz-3.2.0-4-versatile

接下來在同樣的主機上，下載ISO檔。
<pre>http://host/debian-cd/7.8.0/armel</pre>

## 開始安裝
透過下面的指令安裝虛擬磁碟，請自行決定大小
```
$ qemu-img create debian.img 8G
```

然後叫qemu載入ARM kernel，initrd，以及ISO
```
$ qemu-system-arm -M versatileab -kernel ./vmlinuz-3.2.0-4-versatile -initrd ./initrd.gz -cdrom ./debian-7.8.0-armel-DVD-1.iso -hda debian.img -m 1024
```
這邊可以看到versatileab又出現了，請往上找一下這個字串吧。

## 抽出虛擬磁碟的kernel和initrd
最tricky的地方在這邊，理論上你要透過loopback裝置mount 虛擬磁碟，複製/boot就可以了。但是現實就是，因為磁碟機/root的partition有offset，所以直接mount程式無法辨認Filesystem所以無法mount。正確方式如下

```
$ sudo fdisk -l -u debian.img 

Disk debian.img: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders, total 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000823a0

     Device Boot      Start         End      Blocks   Id  System
debian.img1            2048    15988735     7993344   83  Linux
debian.img2        15990782    16775167      392193    5  Extended
debian.img5        15990784    16775167      392192   82  Linux swap / Solaris
```
有兩個東西要注意

* unit為512 bytes
* root partition offset為2048個unit

所以正確的mount方式如下

```
$ sudo mount -o loop,offset=$((2048 * 512)) debian.img /mnt
```

接下來抽出就簡單了，請在剛才安裝的虛擬磁碟檔案同一個目錄操作。

```
$ mkdir boot
$ cp /mnt/boot/* boot/ -rv
```

## 載入安裝的系統
這邊就照表操課，我有指定localhost將port 2222 forward到Qemu的port 22，以便將來ssh進去

```
$ qemu-system-arm -M versatileab -kernel ./boot/vmlinuz-3.2.0-4-versatile -initrd ./boot/initrd.img-3.2.0-4-versatile -hda debian.img -m 1024 -append "root=/dev/sda1" -redir tcp:2222::22
```

驗收看看是不是真的可以連進去，並且裏面真的是ARM的binary？
```
$ ssh -p 2222 user@localhost
user@localhost's password: 
Linux debian 3.2.0-4-versatile #1 Debian 3.2.65-1+deb7u1 armv5tejl

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have mail.
Last login: Fri Feb  6 09:35:07 2015
user@debian:~$ file /bin/ls
/bin/ls: ELF 32-bit LSB executable, ARM, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.26, BuildID[sha1]=0x5bc97dbca9ac168932d898a5e2eaf68e8fde5e16, stripped
```

## 參考資料

* [Dr. Lee's blog: 安裝 qemu arm 版 Debian Linux](http://pominglee.blogspot.tw/2012/11/qemu-arm-debian-linux.html)
* [Connecting to SSH server on QEmu guest](http://www.bramschoenmakers.nl/en/node/100.html)
* [ArchLinux: Qemu, mounting a raw img to temp file, mount needs a filesystem type.](https://bbs.archlinux.org/viewtopic.php?id=95326)
