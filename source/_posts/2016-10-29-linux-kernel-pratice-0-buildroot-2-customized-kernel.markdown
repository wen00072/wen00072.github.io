---
layout: post
title: "Linux Kernel Pratice 0: Buildroot (2/2) - 自行編譯kernel"
date: 2016-10-31 09:40:54 +0800
comments: true
categories: [ARM, Buildroot, Linux Kernel, qemu]
---

## 前情提要
[上一篇](/blog/2016/10/29/linux-kernel-pratice-0-buildroot-setup-with-qemu/)提到，設定實習環境的目標有：

1. 可以使用ARM 平台。一方面追求流行，一方面我不想再開x86這個副本
2. 可以方便地建立ARM平台的Linux Rootfs和kernel版本
3. 可以方便地更改指定要編譯的Kernel版本
4. 透過Qemu ，使用2的Rootfs和kernel開機
5. 透過Qemu和搭配的工具可以分析Linux kernel的run time 行為

今天就是來解決3 的項目。更細分的話，這次目標是

1. 使用官方Linux kernel 編譯Vexpress 平台kernel及產生Buildroot支援的開發版device tree
2. 編譯出來的kernel binary可以在Qemu上順利載入
3. 編譯出來的kernel binary可以順利的載入buildroot產生的rootfs，以及網路介面和相關設備

## 目錄
* [測試環境](#lk0_1_env)
* [下載Linux Kernel Source](#lk0_1_dl)
* [設定和編譯](#lk0_1_conf)
    * [切換版本](#lk0_1_conf_sw)
    * [設定ARM Vexpress預設config](#lk0_1_conf_arm_def)
    * [更改Kernel Config讓Qemu使用](#lk0_1_conf_qemu)
    * [編譯](#lk0_1_conf_build)
        * [Buildroot](#lk0_1_conf_build_broot)
        * [Linux kernel](#lk0_1_conf_build_lk)
    * [產生Device tree binary](#lk0_1_conf_dtb)
* [測試](#lk0_1_test)
* [參考資料](#lk0_1_ref)
* [附錄](#lk0_1_app)
    * [使用Buildroot 內建套件指定編譯Linux kernel 4.2.2](#lk0_1_app_brot)

<a name="lk0_1_env"></a>
## 測試環境
做組裝的最重要的原則之一就是要能夠reproduce，所以交代測試環境是一定要的

```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty

$ git --version
git version 2.10.0
```

* buildroot 版本
    * commit hash: `14b24726a81b719b35fee70c8ba8be2d682a7313`

<a name="lk0_1_dl"></a>
## 下載Linux Kernel Source
沒啥好講的，就剪貼指令吧

```sh
git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
```

<a name="lk0_1_conf"></a>
## 設定和編譯
東西下載完不是就閉著眼睛開幹，因為我們在開始編譯前需要

1. 切換到你想要研究的版本
2. 如果不是x86，你需要指定平台
3. 細項Kernel config設定

那麼就來見招拆招吧

<a name="lk0_1_conf_sw"></a>
### 切換版本
非常簡單，先`git tag`，切過去就好。我要切到4.4.2就是

```sh
git tag                 # 找你要的版本
git checkout v4.4.2     # 切到tag
git checkout -b v4.4.2  # 理論上會這邊改東改西，就先開branch吧
```

<a name="lk0_1_conf_arm_def"></a>
### 設定ARM Vexpress預設config
先講結論
```sh
make ARCH=arm vexpress_defconfig
```

開始~~沒人要看~~的解釋吧。基本上也亂看找出來的，簡單講一下當初的「脈絡」

1. 我知道我們平台是vexpress，所以就`find | grep vexpress`。從一堆檔案中我看到有趣的檔案`./arch/arm/configs/vexpress_defconfig`。
2. 接下來就是要找make 的時候怎麼和這個檔案勾起來。網路上找一下會發現一個變數`ARCH`，剩下就試看看`make ARCH=arm vexpress_defconfig`，能不能動，可以動所以打完收工。

然後你就知道

1. Linux kernel source中有些平台會提供default config
2. 透過`ARCH`可以讓make時自動參考這些檔案產生config

<a name="lk0_1_conf_qemu"></a>
### 更改Kernel Config讓Qemu使用
<font color="red">**建議不要把buildroot compile cache打開。我花了很多時間在kernel 編譯後Qemu還是沒有使用編譯後的kernel的問題，最後發現關閉buildroot compile cache問題就消失了。**</font>

如果閉著眼睛開始[編譯](#lk0_1_conf_build)，你會很高興地發現可以開機了，但是接下來就會很失望的發現出現mount完rootfs找不到`/dev/ttyAMA0`，以至於沒辦法進入login畫面。這是因為雖然serial driver偵測到設備，但是/dev下面沒有相對的device node。解法就是確認下面kernel option有開啟。想要知道真正的原因[手冊這邊](https://buildroot.org/downloads/manual/manual.html#_dev_management)有提到，請參考`Dynamic using devtmpfs only`段落。

* Device Drivers -> Generic Driver Options ->
    * Maintain a devtmpfs filesystem to mount at /dev
    * Automount devtmpfs at /dev, after the kernel mounted the rootfs

建議順便巡一下其他kernel選項，用不到的可以關一下。比如說一堆有的沒的網卡，音效支援之類的。

<a name="lk0_1_conf_build"></a>
### 編譯

<a name="lk0_1_conf_build_broot"></a>
#### Buildroot

1. `make menuconfig`
    * Toolchain -> Custom kernel headers series
        * 改成你現在Linux 版本
2. `make`

<a name="lk0_1_conf_build_lk"></a>
#### Linux kernel
指令如下

```sh
make CROSS_COMPILE=/tmp/buildroot/output/host/usr/bin/arm-buildroot-linux-gnueabihf- ARCH=arm V=1 bzImage
```

這其實就是`make bzImage`的囉唆版，多了

* `ARCH=arm`
    * 指定ARM平台
* `CROSS_COMPILE=..`
    * Cross compile prefix，既然我們使用buildroot內建toolchain，就用他們來編譯kernel
* `V=1`
    * 身為組裝工，沒看到編譯指令訊息跳出來就會沒安全感


<a name="lk0_1_conf_dtb"></a>
### 產生Device tree binary
由於Kernel的演進，可以存放平台硬體相關設定讓kernel啟動時存取。此方式稱為device tree，詳細資訊可以參考這份[簡介(pdf投影片)](https://events.linuxfoundation.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf)。

以論述文來說，儘量先說結論再解釋。因此懶人包如下，假設在kernel top 目錄中：

```text
sudo apt-get install device-tree-compiler
dtc -O dtb -o vexpress-v2p-ca9.dtb arch/arm/boot/dts/vexpress-v2p-ca9.dts
```

~~沒人要看的~~說明如下，懶得看推論的就剪貼上面就好
對於組裝工來說，我關心的是

1. Kernel 軟體包中是否有存在已經有的device tree?
2. 有的話，我要選那一個？
3. 怎麼產生出最後成果？
4. Qemu怎麼使用device tree？

要回上面的問題，最簡單的方式就是回顧buildroot中啟動qemu Vepress的命令參數，你就會發現有個東西似乎和我們關心的device tree有關聯

* `-dtb output/images/vexpress-v2p-ca9.dtb`

這邊顯示了幾個資訊

* 有一個檔案叫dtb
* 檔名的v2p-ca9有可能和平台有關係

那麼我們在Kernel中找一下檔名中有vexpress-v2p-ca9的檔案

```
user@host:/tmp/kernel/linux-stable$ find | grep vexpress-v2p-ca9
./arch/arm/boot/dts/vexpress-v2p-ca9.dts
```

這邊一樣透露了這是一個和`dtb`很類似的檔案，那麼我們進一步做一些確認

```text
$ # dtb 是binary檔案
$ file output/images/vexpress-v2p-ca9.dtb
output/images/vexpress-v2p-ca9.dtb: data

$ # dts是文字檔
$ file ./arch/arm/boot/dts/vexpress-v2p-ca9.dts
./arch/arm/boot/dts/vexpress-v2p-ca9.dts: ASCII text

$ # cat 後會發現是一種描述檔
```

剩下就是估狗大法，發現需要編譯器才能把dts轉換成dtb檔案。
所以就用上面的方式產稱dtb檔囉。

## 測試
剩下就剪貼了

**我在buildroot top目錄執行的**，你要嘛就切到buildroot目錄下，要嘛就指定`-drive file`到你自己rootfs的路徑

```sh
qemu-system-arm -M vexpress-a9 -smp 1 -m 256 -kernel /tmp/kernel/linux-stable/arch/arm/boot/zImage -dtb /tmp/kernel/linux-stable/vexpress-v2p-ca9.dtb  -drive file=output/images/rootfs.ext2,if=sd,format=raw -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net user
```

提出來兩個參數表示這是我編譯出來的kernel而不是buildroot的

* `-kernel /tmp/kernel/linux-stable/arch/arm/boot/zImage`
* `-dtb /tmp/kernel/linux-stable/vexpress-v2p-ca9.dtb` 

開機畫面節錄如下

```text
Booting Linux on physical CPU 0x0
Initializing cgroup subsys cpuset
Linux version 4.4.2 (user@host) (gcc version 4.8.5 (Buildroot 2016.11-git-00439-g14b2472) ) #10 SMP Sat Oct 29 15:25:36 CST 2016
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
Machine model: V2P-CA9
Memory policy: Data cache writeback
CPU: All CPU(s) started in SVC mode.
PERCPU: Embedded 12 pages/cpu @8fdbd000 s18188 r8192 d22772 u49152
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 65024
Kernel command line: console=ttyAMA0,115200 root=/dev/mmcblk0
log_buf_len individual max cpu contribution: 4096 bytes
log_buf_len total cpu_extra contributions: 12288 bytes
log_buf_len min size: 16384 bytes
log_buf_len: 32768 bytes
early log buf free: 14956(91%)
PID hash table entries: 1024 (order: 0, 4096 bytes)
Dentry cache hash table entries: 32768 (order: 5, 131072 bytes)
Inode-cache hash table entries: 16384 (order: 4, 65536 bytes)
Memory: 252732K/262144K available (4816K kernel code, 155K rwdata, 1384K rodata, 284K init, 152K bss, 9412K reserved, 0K cma-reserved)
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xffc00000 - 0xfff00000   (3072 kB)
    vmalloc : 0x90800000 - 0xff800000   (1776 MB)
    lowmem  : 0x80000000 - 0x90000000   ( 256 MB)
    modules : 0x7f000000 - 0x80000000   (  16 MB)
....
buildroot login:
```

<a name="lk0_1_ref"></a>
## 參考資料
* [Buildroot 官方手冊](https://buildroot.org/downloads/manual/manual.html)
* [Device Tree for Dummies! (pdf投影片)](https://events.linuxfoundation.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf)
* [Stackoverflow: COMPILING source device tree file](http://stackoverflow.com/questions/21670967/compiling-source-device-tree-file)


<a name="lk0_1_app"></a>
## 附錄

<a name="lk0_1_app_brot"></a>
### 使用Buildroot 內建套件指定編譯Linux kernel 4.2.2
當初會去做這個主要是因為開始編譯獨立的Linux kernel前要先驗證buildroot自己編的Linux 4.4.2是否可以用qemu開機。另外的好處的就是buildroot編譯出來的kernel config (在output/build/linux-4.4.2/.config) 可以和你自己的kernel config比對。這邊只要在buildroot中make menuconfig中和kernel有關設定指定4.4.2即可，比Versatile簡單太多了故省略，如果有人遇到問題我再補上這邊說明。


