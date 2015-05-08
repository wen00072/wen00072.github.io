---
layout: post
title: 'Ubuntu 14.04 下面的VirtualBox存取USB設備'
date: 2015-02-26 00:36
comments: true
categories: [VirtualBox]
---
照慣例，列一下測試環境

* Host OS
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty
```

* Guest OS
同上。

* VirtualBox 版本: Oracle VM VirtualBox Manager 4.3.22

---
主要有兩件事要做，就是

* [將使用者加入Virtual Box 群組](#vbgrp)
* [安裝延伸套件（Optional）](#vbext)

---
<a name="vbgrp"></a>
## 將使用者加入Virtual Box 群組
把使用者加入`vboxusers`群組
```
$ sudo usermod -a -G vboxusers $USER
```

Double check是否已經在`vboxusers` group 指令
```
$ groups $USER
```

---
<a name="vbext"></a>
## 安裝延伸套件（Optional）

* 去[VirtualBox官方網站](https://www.virtualbox.org/wiki/Downloads)下載Oracle VM VirtualBox Extension Pack。

* 安裝

```
$ sudo virtualbox Oracle_VM_VirtualBox_Extension_Pack-4.3.22-98236.vbox-extpack
```

* 從主選單中要使用USB VM的Settings->USB選單中，勾選`Enable USB 2.0 (EHCI) Controller`，請確認VM是Powered off狀態。
* 驗證:
	* Start VM
  * VM視窗中選Devices -> USB -> 挑你想要使用的USB
  * Guest OS開完機，Linux的話用可以使用lsusb驗證

---
## 參考資料

* [How to add users to vboxusers](http://askubuntu.com/questions/377778/how-to-add-users-to-vboxusers)
