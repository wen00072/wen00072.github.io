---
layout: post
title: "在Banana Pi設定WPA2-PSK無線網路"
date: 2015-12-03 20:50:34 +0800
comments: true
categories: [Banana Pi, Linux, wpa_supplicant]
---
Banana Pi是一套ARMv7為處理器的開發版。一般來說照官方網頁把IMAGE燒到SD卡，外接鍵盤、滑鼠、HDMI螢幕，再通電即可透過GUI設定網路。

由於手上沒有任何外接設備，只有USB轉RS232線和USB WiFi。因此我只能在這樣的設備上設定網路，設定完成後就可以透過ssh server從外面連進去版子了。

照例先描述環境

HOST端
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty

$ minicom --version
minicom version 2.7 (compiled Jan  1 2014)
...
```

設備端，假設你已經將系統燒入到SD卡中

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty
``` 

## 接上TTY
首先你要有一條RS232轉USB的設備，上頭有TX/RX/GND/VCC，VCC在這邊用不到。
<img src="/files/banana_pi/DSC_0026.jpg" title="RS232轉USB線">

版子上面其實已經幫你把腳位標示好了，上面有TX/RX和GND如下：
<img src="/files/banana_pi/DSC_0030.jpg" title="Banana Pi RS232接腳">

剩下的就是把線路接起來。由於版子已經裝上外框，我有用鑷子協助連接線路。
<img src="/files/banana_pi/DSC_0024.jpg" title="接上訊號線">

線路接完後，將USB接上你的HOST，檢查下面幾項

* 下`dmesg`確認HOST找到`/dev/ttyUSBn`（n為0開始的正整數）
* 確認你的終端機（我用minicom）設備指定`/dev/ttyUSBn`（n為0開始的正整數）
* 確認你的終端機（我用minicom）參數為115200 BPS，8N1，軟體硬體流量控制關閉

開啟你的終端機軟體，然後版子通電。當終端機畫面進入提示符號，請輸入帳號密碼。Banana Pi有預設的帳號密碼請自行上網查詢。

## 設定無線網路
Ubuntu 是透過`/etc/network/interface`去設定網路介面。這邊我們可以分成兩個部份討論

### 設定無線網路介面
首先你要下`ifconfig -a`看看你的無線網路介面名稱是什麼。我這邊是`wlan2`，為什麼不是`wlan0`，不要問我。

接下來就是修改`/etc/network/interface`，先貼上我網路參考的部份

```text /etc/network/interface
auto wlan2

allow-hotplug wlan2
iface wlan2 inet dhcp
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

大概解釋一下

* `auto`：`ifup`指令有`-a`參數會把有`auto`的網路介面全部bring up (bring up請自行估狗)
* `allow-hotplug`：當kernel偵測到該網路介面被接上會自動bring up該網路介面，[出處](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html)
* `iface wlan2 inet dhcp`：指定網路介面`wlan2`使用`TCP/IP`，動態分配IP
* `wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf`：找不到最原始出處，網路上就算是[Debian官方文件](https://wiki.debian.org/WiFi/HowToUse)也直接拿來用而已，`man -K wpa-conf`也找不到。不過字面上不太難猜，就是指定wpa 會吃的config檔案路徑。


### 設定無線網路連線
前面有看到設定wpa的config檔案，接下來就來設定吧。我這邊是沒有該config檔，所以要自己新增一個。基本上就是設定SSID，密碼，加密方式，以及說明是否你要連的AP是否沒有broadcast SSID等。這邊我只是[參考這邊](https://coderwall.com/p/v290ta/raspberry-pi-wifi-setup-with-wpa2-psk-aes)，有興趣的人可以自行鑽研。

```text /etc/wpa_supplicant/wpa_supplicant.conf
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
ssid="AP的SSID"
scan_ssid=1 # 如果你的AP 是沒有broadcast SSID就要加這個
psk="你的AP 密碼(passphase)"
proto=RSN
key_mgmt=WPA-PSK
pairwise=CCMP
auth_alg=OPEN
}
```

設定完畢確認連線正常、有安裝sshd後，剩下就透過ssh操作版子了。祝好運！
