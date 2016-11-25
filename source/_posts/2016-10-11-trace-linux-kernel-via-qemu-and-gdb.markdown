---
layout: post
title: "(過期) Linux Kernel Pratice 0.5: 使用gdb 加 Qemu trace Linux kernel runtime 行為"
date: 2016-10-11 09:46:25 +0800
comments: true
categories: [過期]
---
### <font color="red">感謝Scott 大大糾正，選錯平台，這篇使用了ARMv4指令集的測試平台。請大家忽略，正確的版本將會之後更新!</font>

## 前言
還是感謝[Scott](http://scottt.tw/)大大和其他網友的幫忙。這次最重要的心得就是不要在睡眠不足的時候做實驗，浪費的時間拿去睡覺反而比較實在。

## 目錄

* [測試環境](#de_lk0_5_env)
* [Linux kernel環境設定](#de_lk0_5_kr)
* [測試](#de_lk0_5_test)
    * [Qemu](#de_lk0_5_test_qemu)
    * [gdb](#de_lk0_5_test_gdb)
* [參考資料](#de_lk0_5_ref)
* [補充](#de_lk0_5_add)

<a name="de_lk0_5_env"></a>
## 測試環境

### Host
* Host OS

```sh
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty
```

* Qemu

```sh
$ qemu-system-arm --version
QEMU emulator version 2.0.0 (Debian 2.0.0+dfsg-2ubuntu1.27), Copyright (c) 2003-2008 Fabrice Bellard
```

* gdb

```text
$ arm-none-eabi-gdb --version
GNU gdb (GNU Tools for ARM Embedded Processors) 7.10.1.20160923-cvs
Copyright (C) 2015 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl
...
```

* buildroot 版本
    * commit hash: `14b24726a81b719b35fee70c8ba8be2d682a7313`

### Target

* Linux kernerl 版本
    * 4.4.2
* 模擬平台
    * Versatile

<a name="de_lk0_5_kr"></a>
## Linux kernel環境設定
我目前只打開加入debug資訊的選項。接下來重編，編譯的方式請參考[這邊](/blog/2016/10/01/linux-kernel-pratice-0-buildroot-2-customized-kernel/)

* Kernel hacking -> Compile-time checks and compiler options ->
    * Compile the kernel with debug info

<a name="de_lk0_5_test"></a>
## 測試

依照下面兩個步驟執行

1. 執行qemu，並且加入支援gdb以及開始馬上freeze CPU的參數
2. 執行gdb，載入symbol並聯到qemu除錯


<a name="de_lk0_5_test_qemu"></a>
### Qemu
基本上就是原本的指令加入兩個選項

* `-S`
    * qemu一開始立即Freeze CPU
* `-s`
    * `-gdb tcp::1234`的縮寫，也就是說gdb可以透過port 1234和連到Qemu除錯

假設你在buildroot最上層，就可以使用下面指令執行qemu 並使用gdb 除錯

```text
qemu-system-arm -M versatilepb \
                -kernel /tmp/kernel/arch/arm/boot/zImage \
                -drive file=output/images/rootfs.ext2,if=scsi,format=raw \
                -append "root=/dev/sda console=ttyAMA0,115200" \
                -serial stdio -net nic,model=rtl8139 -net user -s -S
```

<a name="de_lk0_5_test_gdb"></a>
### gdb

這邊有點瑣碎，先講一下步驟

1. 載入Linux kernel symbol
    * 假設你在kernel最上層目錄有三種方式載入
        * 直接`arm-none-eabi-gdb ./vmlinux`
        * `arm-none-eabi-gdb -ex "file ./vmlinux"`
        * 進入gdb後打`file ./vmlinux`指令
2. 連上qemu
    * 一樣兩種方式
        * `arm-none-eabi-gdb -ex "target remote :1234"`
        * 進入gdb後打`target remote :1234`指令
3. 設定breakpoint等你要觀察的資訊
    * 如`b printk`
4. 告訴qemu開始執行
    * `continue`


1和2可以一起使用如下

```text 懶人包
arm-none-eabi-gdb -ex "file ./vmlinux"  -ex "target remote :1234"
```

現在看一下操作範例

```text
$ arm-none-eabi-gdb ./vmlinux  -ex "target remote :1234" 
GNU gdb (GNU Tools for ARM Embedded Processors) 7.10.1.20160923-cvs
...
Reading symbols from ./vmlinux...done.
Remote debugging using :1234
__vectors_start () at arch/arm/kernel/entry-armv.S:1210
1210		W(b)	vector_rst
(gdb) b printk
Breakpoint 1 at 0xc0069df4: file kernel/printk/printk.c, line 1900.
(gdb) c
Continuing.

Breakpoint 1, printk (fmt=0x0 <__vectors_start>) at kernel/printk/printk.c:1900
1900	{
(gdb) bt
#0  printk (fmt=0x0 <__vectors_start>) at kernel/printk/printk.c:1900
#1  0xc039d8c4 in start_kernel () at init/main.c:508
#2  0x00008048 in ?? ()
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
```

另外如果有興趣使用Linux kernel提供的指令，可以使用下面的方式在啟動gdb時載入
，只要把下面的描述加到你的`~/.gdbinit`即可

```
add-auto-load-safe-path /tmp/kernel/scripts/gdb/vmlinux-gdb.py
```

那麼你就可以使用Linux kernel提供的gdb script，詳細的設定和指令說明在[這邊](https://www.kernel.org/doc/Documentation/gdb-kernel-debugging.txt)。

<a name="de_lk0_5_ref"></a>
## 參考資料

* [Linux Magazine: Qemu and the Kernel](http://www.linux-magazine.com/Online/Features/Qemu-and-the-Kernel)
* [Linux kernel document: Debugging kernel and modules via gdb](https://www.kernel.org/doc/Documentation/gdb-kernel-debugging.txt)
* [stackoverflow: How to debug the Linux kernel with GDB and QEMU?](http://stackoverflow.com/questions/11408041/how-to-debug-the-linux-kernel-with-gdb-and-qemu)


<a name="de_lk0_5_add"></a>
## 補充
當初犯了蠢事載入不正確的kernel image導致一堆不必要的除錯。不過多學到一個gdb Python script除錯指令，當Python script發生exception時可以用下面的指令印出Python錯誤call stack

* gdb內：`set python print-stack full`
* 啟動gdb時加入參數： `-ex "set python print-stack full"`
