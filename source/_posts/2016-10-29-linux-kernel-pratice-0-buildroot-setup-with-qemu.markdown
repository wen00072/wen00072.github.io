---
layout: post
title: "Linux Kernel Pratice 0: Buildroot (1/2)"
date: 2016-10-29 09:43:20 +0800
comments: true
categories: [ARM, Qemu, Linux Kernel, Buildroot]
---

理論上不應該要邊移動邊開火，延長戰線。不過計劃趕不上變化，既來之則安之。

最近因為特別因素開始學習Linux kernel，看能不能Linux kernel和STM32兩邊都不要漏掉。不管怎樣，學習和實習絕對分不開，所以還是從環境架設開始吧。這次的實習環境架設的目標是：

1. 可以使用ARM 平台。一方面追求流行，一方面我不想再開x86這個副本
2. 可以方便地建立ARM平台的Linux Rootfs和kernel版本
3. 可以方便地更改指定要編譯的Kernel版本
4. 透過Qemu ，使用2的Rootfs和kernel開機
5. 透過Qemu和搭配的工具可以分析Linux kernel的run time 行為

今天只有辦到1, 2和4而已，剩下的請參考之後的文章。

## 目錄

* [測試環境](#lk0_env)
* [安裝Buildroot](#lk0_ins)
    * [下載Buildroot](#lk0_ins_dl)
    * [設定ARM 環境](#lk0_ins_set)
    * [編譯及輸出](#lk0_ins_build)
* [測試](#lk0_test)
* [參考資料](#lk0_ref)
    * [下次準備看的資料](#lk0_ref_data)

<a name="lk0_env"></a>
## 測試環境
因為我已經裝過開發相關的套件，因此如果您是新手可能要自行摸索也許有需要另外安裝的套件如`git`。嘛，練習解讀錯誤訊息也是一種學習。

<pre>
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
</pre>

<a name="lk0_ins"></a>
## 安裝Buildroot
主要分成下面三個步驟

* [下載Buildroot](#lk0_ins_dl)
* [設定ARM 環境](#lk0_ins_set)
* [編譯及輸出](#lk0_ins_build)

<a name="lk0_ins_dl"></a>
### 下載Buildroot
直接看例子，剪下貼上就好

```sh
mkdir buildroot
cd buildroot
git clone git://git.buildroot.net/buildroot
```

<a name="lk0_ins_set"></a>
### 設定ARM 環境
網路上查到大部分都是從`make menuconfig`開始。不過我是很**明確地**要用`Qemu`跑ARM的系統。所以使用下面指令查詢。

```
make list-defconfigs
```

Scott 大大指出可以使用`qemu_arm_vexpress` ，原因是這個平台模擬的CPU是 `Cortex-A9`(ARMv7-A)的平台。[之前](/blog/2016/09/27/linux-kernel-pratice-0-buildroot-setup-with-qemu/)我是用的模擬平台使用的CPU是`ARM926EJ-S`(ARMv5TE)，它的instruction 架構和現在差距太多，所以就轉換到這邊了。


接下來就用`make menuconfig`做細項調整，因為是拿來做分析系統行為，所以調整的重點是增加系統的可觀察度、除錯工具、開發軟體套件等。

開啟或新增下面設定如下：

* Build options
    * build packages with debugging symbol
    * gcc debug level 
        * debug level 3
    * strip command for binaries on target
        * none
    * gcc optimization level
        * optimize for debugging
* Toolchain
    * C library
        * glibc
    * glibc version
        * 2.24
    * GCC compiler Version
        * 4.8.x
    * Build cross gdb for the host
        * TUI support
        * Python support
        * Simulator support
* Target packages
    * Debugging, profiling and benchmark
        * gdb
            * gdbserver
            * full debugger
            * TUI support
        * ltrace
        * strace
        * valgrind 和所有相關的東西
    * Development tools
        * binutils
        * git
        * gperf
        * libtool
        * make
        * pkgconf
        * subversion
        * tree

自行編譯 Kernel 部份下一篇會再說明。

<a name="lk0_ins_build"></a>
### 編譯及輸出
編譯只要下`make`就會幫你下載和編譯開機需要的

1. 套件和一些常用工具，並封裝到`output/image/roofs.ext2`
2. Kernel(預設4.7)，編譯成`zImage`，放在`output/image/zImage`

<a name="lk0_test"></a>
## 測試
接下來也不難，可以參考`board/qemu/arm-vexpress/readme.txt`
簡單來說就是執行下面指令，開機完使用`root`登入不用密碼，使用`poweroff`後再手動離開qemu。

```sh
qemu-system-arm -M vexpress-a9 -smp 1 -m 256 -kernel output/images/zImage -dtb output/images/vexpress-v2p-ca9.dtb -drive file=output/images/rootfs.ext2,if=sd,format=raw -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net user
```

執行畫面如下

```text
$ qemu-system-arm -M vexpress-a9 -smp 1 -m 256 -kernel output/images/zImage -dtb output/images/vexpress-v2p-ca9.dtb -drive file=output/images/rootfs.ext2,if=sd,format=raw -append "console=ttyAMA0,115200 root=/dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net user
Booting Linux on physical CPU 0x0
Initializing cgroup subsys cpuset
Linux version 4.4.2 (user@host) (gcc version 4.8.5 (Buildroot 2016.11-git-00439-g14b2472) ) #1 SMP Sat Oct 29 12:37:50 CST 2016
CPU: ARMv7 Processor [410fc090] revision 0 (ARMv7), cr=10c5387d
...
Initializing random number generator... done.
Starting network: smsc911x 4e000000.ethernet eth0: SMSC911x/921x identified at 0x912a0000, IRQ: 31
udhcpc: started, v1.25.0
udhcpc: sending discover
udhcpc: sending discover
udhcpc: sending select for 10.0.2.15
udhcpc: lease of 10.0.2.15 obtained, lease time 86400
deleting routers
adding dns 10.0.2.3
OK
Starting sshd: OK

Welcome to Buildroot
buildroot login: 

```

<a name="lk0_ref"></a>
## 參考資料
* [The Buildroot user manual](https://buildroot.org/downloads/manual/manual.html)
    * 只有看部份，不過官方文件本來就是應該放在第一位
* [Buildroot and QEMU – the quickest receipe for your own Linux](http://pressreset.net/2013/09/buildroot-and-qemu-the-quickest-receipe-for-your-own-linux/)
    * 東西弄完才看到的文章，入門好文

<a name="lk0_ref_data"></a>
### 下次準備看的資料
* [Qemu and the Kernel](http://www.linux-magazine.com/Online/Features/Qemu-and-the-Kernel)
    * 使用Qemu debug kernel的資料
* [Stackoverflow: Can virtfs/9p be used as root file system?](http://unix.stackexchange.com/questions/90423/can-virtfs-9p-be-used-as-root-file-system)
    * Qemu和Host主機共享資料，甚至直接把rootfs放host讓qemu去讀取的方式
