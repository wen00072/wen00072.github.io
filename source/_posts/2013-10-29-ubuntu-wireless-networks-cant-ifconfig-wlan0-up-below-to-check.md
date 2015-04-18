---
layout: post
title: 'Ubuntu 12.04下面無線網路無法ifconfig wlan0 up的檢查'
date: 2013-10-29 00:48
comments: true
categories: [Linux, wireless]
---
## 錯誤訊息
```
$ifconfig wlan0 up
SIOCSIFFLAGS: Operation not possible due to RF-kill
```

## 解決方式

- 列出RF的block狀態，如果是硬體關掉請自行打開RF開關
    - `rfkill list all`

```
$ rfkill list all
8: phy1: Wireless LAN
	Soft blocked: yes
	Hard blocked: yes
9: hci0: Bluetooth
	Soft blocked: yes
	Hard blocked: no
```

- 從軟體上面打開WIFI
    - `rfkill unblock wifi`
