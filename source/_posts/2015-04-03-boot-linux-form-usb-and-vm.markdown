---
layout: post
title: "透過 Virtualbox 安裝Linux到USB Disk上"
date: 2015-04-03 19:21:01 +0800
comments: true
categories: [Linux, USB, Virtualbox]
---

想說裝個Ubuntu 到USB 3.0 Disk上，隨手紀錄一下

## 目錄
* [事先準備](#usb_prepare)
* [測試環境](#usb_env)
* [使用Virtualbox 安裝Linux 到 USB disk](#usb_install)
* [使用Virtualbox 從USB Disk 開機](#usb_boot)
* [安裝後設定](#usb_setup)
* [參考資料](#usb_ref)

<a name="usb_prepare"></a>
## 事先準備

* USB 3.0 Disk，寫入速度愈快愈好。另外你要看你喜歡的Linux Distribution決定USB空間大小。我是隨便買個299號稱寫入速度20MB的16G USB Disk。
* 安裝VirtualBox
* 你喜歡的 Linux Distribution iso檔案
* USB 插槽要2.0，很遺憾3.0目前為止Virtual Box不支援。不用鐵齒，我在這邊浪費三小時生命。

<a name="usb_env"></a>
## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty

$ vboxmanage --version
4.3.26r98988
```

<a name="usb_install"></a>
## 使用Virtualbox 安裝Linux 到 USB disk

* Virtualbox 新增machine，只要把光碟設成ISO檔。不用新增硬碟。
* 開機Virtualbox machine，出現GUI後，到Virtualbox machine　的 Device -> USB 挑你要安裝的USB volume。
* 開始安裝。因為USB disk有寫入次數限制，所以我不開swap。剩下就是下一步下一步，除非你玩Arch Linux或是Ubuntu server。順帶一題，我安裝Ubuntu server 到USB失敗kerker。

<a name="usb_boot"></a>
## 使用Virtualbox 從USB Disk 開機

這邊有幾個要注意的地方

* 你要有USB 的對應partition 的device node權限。精確的來說，你要在`disk`的group中。請用下面的指令
    * `sudo usermod -G -a disk $USER`
    * <font color="red">血的教訓，`usermod -G`一定要加`-a`，不然你group就會只有`disk`，如果剛好你的帳號是唯一在`sudo`group的話，就只好拿rescue disk開機救回了。</font>
    * 目前我只能logout session重新login才能更新group。
* 你要讓virtualbox 可以直接用USB disk對應的partition 開機。請用下面的指令。
    * `VBoxManage internalcommands createrawvmdk -filename /path/to/file.vmdk -rawdisk /dev/sdx`

接下來就是開另外一個新的Virtualbox machine，指定該檔案。打完收工！

<a name="usb_setup"></a>
## 安裝後設定

主要是要減少寫入Disk的機會，目前找到的方式有

* 關掉瀏覽器的cache
* mount disk option加入noatime
* 安裝zram-config
* 把/tmp改成tmpfs


<a name="usb_ref"></a>
## 參考資料
* [stackexchange: How can I extend the life of my SD card?](http://raspberrypi.stackexchange.com/questions/169/how-can-i-extend-the-life-of-my-sd-card)
* [Virtualbox manual: 9.9.1.1. Access to entire physical hard disk](https://www.virtualbox.org/manual/ch09.html#rawdisk)
