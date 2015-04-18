---
layout: post
title: 'Linux: 用Live CD跑硬碟裏面的Linux'
date: 2015-01-03 13:05
comments: true
categories: 
---
我用這個救過過兩次grub GG的狀況，每次都要上網回想一下，趁著還記得的情況紀錄一下。


* 用Live CD開機。你的Live CD要和你硬碟的Linux distribution相同。
* 想辦法開終端機
* 輸入以下的~~咒語~~指令，假設你的硬碟root filesystem partition放在/dev/sda2

```bash
sudo mount /dev/sda2 /mnt
sudo mount --bind /dev /mnt/dev
sudo mount --bind /dev/pts /mnt/dev/pts
sudo mount --bind /proc /mnt/proc
sudo mount --bind /sys /mnt/sys
sudo chroot /mnt
```

剩下就是處理grub問題了，請自己估狗。

## **注意！grub是bootloader，沒弄好有可能會讓你的硬碟資料GG。操作有風險，操作前應詳閱相關文件資料。如果你不知道boot loader，grub，root file system，mount，chroot是什麼的話，請找熟悉的碰友幫你處理。**

* 參考資料
	* [How to Repair, Restore, or Reinstall Grub 2 with a Ubuntu Live CD or USB](http://howtoubuntu.org/how-to-repair-restore-reinstall-grub-2-with-a-ubuntu-live-cd)
