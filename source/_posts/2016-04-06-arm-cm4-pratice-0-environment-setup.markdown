---
layout: post
title: "ARM CM4 Pratice (0): Environment Setup"
date: 2016-04-06 22:15:15 +0800
comments: true
categories: [ARM, STM32F4]
---
很久以前買了[STM32F4 Disco的開發版](http://www.st.com/web/catalog/tools/FM116/SC959/SS1532/PF259090)（[繁體中文資訊](http://wiki.csie.ncku.edu.tw/embedded/STM32F429)），最近開始想要學習ARM架構和硬體控制等技術。在設定目標，規劃進度前，一定要先把編譯和燒錄環境架設起來，整理如下：


## 目錄

* [安裝作業系統環境](#cm4-0-env)
* [安裝步驟](#cm4-0-steps)
    * [安裝Toolchain](#cm4-0-steps-tl)
    * [手動安裝燒錄和除錯工具](#cm4-0-steps-pkg-install)
        * [安裝需要套件](#cm4-0-steps-pkg-install-prepare)
        * [手動安裝st-link](#cm4-0-steps-pkg-install-stlink)
        * [手動安裝OpenOCD](#cm4-0-steps-pkg-install-openocd)
* [驗證燒錄](#cm4-0-test)
    * [事前準備](#cm4-0-test-prepare)
    * [使用st-flash 開發版燒錄](#cm4-0-test-stflash)
    * [使用openocd 開發版燒錄](#cm4-0-test-openocd)
* [參考資料](#cm4-0-ref)

<a name="cm4-0-env"></a>
## 安裝作業系統環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.4 LTS
Release:	14.04
Codename:	trusty
```

<a name="cm4-0-steps"></a>
## 安裝步驟
要開發非本機的平台，我們會需要安裝

* Toolchain：提供編譯，函數，分析binary等功能
* 燒錄軟體
* 除錯工具

st-link和openocd同時有燒錄和開發版軟體除錯的功能，因此我們兩個都安裝。

<a name="cm4-0-steps-tl"></a>
### 安裝Toolchain

網路上找到[GCC ARM Embedded (ARM R和M系列使用 )](https://launchpad.net/gcc-arm-embedded)的PPA。簡單找一下網路，**<font color="red">沒有直接證據說這邊的PPA是由ARM官方維護</font>**。不過在[mbed(ARM 成立的IoT作業系統)網站裏面](http://yottadocs.mbed.com/#installing-on-linux)的確有提到這個PPA，暫時當作間接證據。**<font color="red">目前我還沒有驗證這個Toolchain編譯出來的binary是否可以在開發版上正常動作</font>**。

```text 安裝Toolchain指令
sudo add-apt-repository ppa:team-gcc-arm-embedded/ppa
sudo apt-get update
sudo apt-get install gcc-arm-embedded
```
<a name="cm4-0-steps-pkg-install"></a>
### 手動安裝燒錄和除錯工具
<a name="cm4-0-steps-pkg-install-prepare"></a>
#### 安裝需要套件

```text 系統安裝設定STM32F4環境相關套件 
sudo apt-get install git
sudo apt-get install libtool
sudo apt-get install automake
sudo apt-get install autoconf
sudo apt-get install pkg-config
sudo apt-get install libusb-1.0-0-dev
sudo apt-get install libftdi-dev   # 當openocd要支援MPSSE mode才需要裝，這是啥我不知道。
sudo apt-get install libhidapi-dev # 當openocd要支援CMSIS-DAP才會用到，這是啥我不知道。
```

我自己的習慣是非必要不會放在系統中，**接下來的指令都是預設`st-link`和`openocd`裝到`$HOME/bin`中**，Ubuntu在登入時會自動將`~/bin`加入`PATH`中。

如果您沒有~/bin的話，請使用下面指令建立。
```
mkdir ~/bin/
. ~/.profile # 強迫系統自動將`~/bin`加入`PATH`中
```

<a name="cm4-0-steps-pkg-install-stlink"></a>
#### 手動安裝st-link
簡單講一下，就是下載、編譯、安裝軟體。特別要注意的是，為了要讓你的PC在使用Mini USB連到開發版後能夠讓Ubuntu的`udev`服務可以順利地偵測到開發版，需要加入相關設定並重新啟動`udev`服務。

下載Source code
```text 下載Source code  
git clone https://github.com/texane/stlink
```

編譯stlink
```text 編譯
cd stlink/
./autogen.sh 
./configure 
make
cp st-* ~/bin
```

設定並重新起動udev
```text 設定並重新起動udev
sudo cp 49-stlinkv* /etc/udev/rules.d/ -rv
sudo service udev restart
```

驗證一下是否可以偵測到開發版，請確定測試前開發版已經和您的電腦透過Mini USB連線，以及有設定`udev`並且重新啟動。

```text   
$ st-probe 
Found 1 stlink programmers
30303030303030303030303100
	 flash: 2097152 (pagesize: 16384)
	  sram: 262144
	chipid: 0x0419
	 descr: F42x and F43x device
```

<a name="cm4-0-steps-pkg-install-openocd"></a>
#### 手動安裝OpenOCD

下載Source code
```text 下載Source code
git clone git://git.code.sf.net/p/openocd/code openocd-code
```

編譯
```text 編譯
cd openocd-code/
./bootstrap 
./configure # script會自動偵測系統是否有相依開發套件
make
```

安裝
```text 安裝
cp src/openocd ~/bin
cp tcl/board/stm32f429discovery.cfg ~/.openocd/openocd.cfg
cp tcl/* ~/.openocd
```

驗證一下是否可以偵測到開發版，請確定測試前開發版已經和您的電腦透過Mini USB連線。

```text
$ openocd 
Open On-Chip Debugger 0.10.0-dev-00250-g9c37747 (2016-04-07-22:20)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
srst_only separate srst_nogate srst_open_drain connect_deassert_srst
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : clock speed 1800 kHz
Info : STLINK v2 JTAG v17 API v2 SWIM v0 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 2.862887
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
```

<a name="cm4-0-test"></a>
## 驗證燒錄
<a name="cm4-0-test-parepare"></a>
### 事前準備
下載jserv整理的STM 軔體
```
git clone https://github.com/jserv/stm32f429-demos
```

將source中的hex轉成binary
```
cd stm32f429-demos/
make
```

使用下載提供的Makefile燒錄，裏面會先用openocd燒錄，失敗的話自動切到st-flash燒錄，非常建議讀一下Makefile。
```
make flash-demo檔案
```

目前套件內蒐集的demo檔案

* flash-GamesInJava
* flash-STM32F429I-DISCOVERY_Demo_V1.0.1
* flash-Paint
* flash-TouchGFX-demo2014
* flash-STM32CubeDemo_STM32F429I-Discovery  

<a name="cm4-0-test-stflash"></a>
### 使用st-flash 開發版燒錄

* `st-flash write 軔體binary檔案名稱 0x8000000`

為什麼是`0x800000`呢？可以查一下[手冊](http://www.st.com/web/en/resource/technical/document/datasheet/DM00071990.pdf)，上面有說這塊記憶體是分配給flash memory。

<a name="cm4-0-test-openocd"></a>
#### 使用openocd 開發版燒錄
從[GitHub: jserv/stm32f429-demos](https://github.com/jserv/stm32f429-demos)裏面的Makefile抄來的。指令比較長，不過這是因為openocd可以把一系列的命令連發的關係

```
openocd -f interface/stlink-v2.cfg  -f target/stm32f4x.cfg  -c "init"  -c "reset init"  -c "stm32f2x unlock 0"  -c "flash probe 0"  -c "flash info 0"  -c "flash write_image erase 軔體binary檔案名稱 0x8000000"  -c "reset run" -c shutdown
```

使用比較好看的排版
```
openocd -f interface/stlink-v2.cfg  \
        -f target/stm32f4x.cfg      \
        -c "init"                   \
        -c "reset init"             \
        -c "stm32f2x unlock 0"      \
        -c "flash probe 0"          \
        -c "flash info 0"           \
        -c "flash write_image erase 軔體binary檔案名稱 0x8000000" \
        -c "reset run" -c shutdown
``` 

#### 參數說明

* `-f`
    * 指定config檔案。你可能會想問說interface目錄在那邊，還記得[前面](cm4-0-steps-pkg-install-openocd)的動作嘛？
    * 可以指定
        * interface: 除錯使用的介面
        * target: 執行的平台CPU如STM32F4 
        * board: 開發版
* `-c`
    * 執行openocd指令

#### 指令說明
* `-f interface/stlink-v2.cfg`
* `-f target/stm32f4x.cfg`
* `-c "init"`
    * 結束config stage，開始進入run stage ([出處](http://openocd.org/doc/html/Daemon-Configuration.html#Entering-the-Run-Stage))
* `-c "reset init"`
    * reset 開發版，重新開機([出處](http://openocd.org/doc/html/General-Commands.html#Target-State-handling))後進入init狀態。
* `-c "stm32f2x unlock 0"`
    * 將開發版的flash解鎖([出處](http://openocd.org/doc/html/Flash-Commands.html#Flash-Configuration-Commands))
* `-c "flash probe 0"`
    * 偵測開發版的flash([出處](http://openocd.org/doc/html/Flash-Commands.html#Flash-Configuration-Commands))
* `-c "flash info 0"`
    * 顯示開發版的flash資訊([出處](http://openocd.org/doc/html/Flash-Commands.html#Flash-Configuration-Commands))
* `-c "flash write_image erase 軔體binary檔案名稱 0x8000000"`
    * 將檔案寫入flash([出處](http://openocd.org/doc/html/Flash-Commands.html#Flash-Configuration-Commands))
* `-c "reset run" -c shutdown`
    * 正常重新開機([出處](http://openocd.org/doc/html/General-Commands.html#Target-State-handling))

<a name="cm4-0-ref"></a>
## 參考資料

* [yotta Documentation: Install on Linux](http://yottadocs.mbed.com/#installing-on-linux)
* [ARM Embedded PPA網站](https://launchpad.net/gcc-arm-embedded)
* [GNU ARM Embedded Toolchain](https://launchpad.net/~team-gcc-arm-embedded/+archive/ubuntu/ppa)
* [GitHub: jserv/stm32f429-demos](https://github.com/jserv/stm32f429-demos)
* [OpenOCD手冊](http://openocd.org/doc/html/)
