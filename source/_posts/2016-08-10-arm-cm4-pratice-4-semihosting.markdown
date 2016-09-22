---
layout: post
title: "ARM CM4 Pratice (4): semihosting"
date: 2016-08-10 12:51:46 +0800
comments: true
categories: [ARM, STM32, semihosting]
---
Semihosting 是一種讓開發版透過除錯介面執行主機上的服務。透過semihosting，使用者可以快速驗證功能。舉例來說，在USART還沒搞定前先把訊息透過semihosting印到主機上觀看程式的行為。

本次練習使用`openocd`提供主機上的服務。以下是這次的練習

## 目錄

* [測試環境](#stm-4-env)
* [使用Semihosting](#stm-4-semi)
    * [開發版](#stm-4-semi-target)
    * [主機 (openocd)](#stm-4-semi-host)
* [實驗：Echo](#stm-4-test)
    * [程式](#stm-4-test-code)
        * [主程式](#stm-4-test-code-main)
        * [Semihosting 程式](#stm-4-test-code-semi)
    * [Makefile](#stm-4-test-makefile)
    * [執行結果](#stm-4-test-result)
* [參考資料](#stm-4-ref)

<a name="stm-4-env"></a>
## 測試環境

```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.4 LTS
Release:	14.04
Codename:	trusty

$ arm-none-eabi-gcc --version
arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.4.1 20160609 (release) [ARM/embedded-5-branch revision 237715]
...

$ openocd --version
Open On-Chip Debugger 0.10.0-dev-00250-g9c37747 (2016-04-07-22:20)
...
``` 

* SPL版本： STM32F4xx_DSP_StdPeriph_Lib_V1.6.1
* 開發板： STM32F4 Dicovery, [STM32F429-Disco](http://www.st.com/content/st_com/en/products/evaluation-tools/product-evaluation-tools/mcu-eval-tools/stm32-mcu-eval-tools/stm32-mcu-discovery-kits/32f429idiscovery.html)

<a name="stm-4-semi"></a>
## 使用Semihosting


<a name="stm-4-semi-target"></a>
### 開發版
ARM官方網站有說，在ARMv6-M 和 ARMv7-M (Cortex M4是ARMv7E-M)架構中，使用break point指令並帶入參數`0xab`即可。詳細的使用範例會在[後面](#stm-4-test)說明。

以下是ARM規範的Semihosting服務

* [angel_SWIreason_EnterSVC (0x17)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/BEIDFAEH.html)
* [angel_SWIreason_ReportException (0x18)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/BEIGEDFE.html)
* [SYS_CLOSE (0x02)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baccdcef.html)
* [SYS_CLOCK (0x10)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdcfje.html)
* [SYS_ELAPSED (0x30)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdgdcc.html)
* [SYS_ERRNO (0x13)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdhfgc.html)
* [SYS_FLEN (0x0C)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdgdic.html)
* [SYS_GET_CMDLINE (0x15)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baccbcbe.html)
* [SYS_HEAPINFO (0x16)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacbefaa.html)
* [SYS_ISERROR (0x08)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baccjafh.html)
* [SYS_ISTTY (0x09)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacbhcee.html)
* [SYS_OPEN (0x01)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdagge.html)
* [SYS_READ (0x06)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baceahbd.html)
* [SYS_READC (0x07)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baccbaba.html)
* [SYS_REMOVE (0x0E)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacbffjf.html)
* [SYS_RENAME (0x0F)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdjebc.html)
* [SYS_SEEK (0x0A)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdfgcg.html)
* [SYS_SYSTEM (0x12)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdijcb.html)
* [SYS_TICKFREQ (0x31)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacchdbe.html)
* [SYS_TIME (0x11)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacebdhg.html)
* [SYS_TMPNAM (0x0D)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Baccafja.html)
* [SYS_WRITE (0x05)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacbedji.html)
* [SYS_WRITEC (0x03)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacbejbe.html)
* [SYS_WRITE0 (0x04)](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/Bacdhdcd.html)

<a name="stm-4-semi-host"></a>
### 主機 (openocd)
其實非常簡單，就是啟動openocd的時候要把semihosting打開。單獨指令如下

```
arm semihosting enable
```

<a name="stm-4-test"></a>
## 實驗：Echo
完整可以編譯的程式碼在[這邊](https://github.com/zzz0072/STM32F429-Discovery-Disco-Pratice/tree/master/labs/2_semihosting)

這個實驗行為和[前一個實驗](http://wen00072.github.io/blog/2016/08/05/arm-cm4-pratice-3-lab-usart/)很類似，就是透過semihosting的read服務主機端輸入資料，然後再透過semihosting的write服務印到螢幕上。

<a name="stm-4-test-code"></a>
### 程式


<a name="stm-4-test-code-main"></a>
#### 主程式
非常簡單，唯一要說明的是在微處理器中，所有的東西都要自己來，也就是說下面的`memset`、`strlen`都是自己寫的。

```c semihosting.c
#include "my_utils.h"

#define GREETING_STR  "Hello World\n\r"
#define PROMPT_STR    "\r> "
#define LINE_MAX_CHAR (64)

int main(int argc, char **argv)
{
    char line[LINE_MAX_CHAR];

    /* Greeting */
    host_write(STDOUT, GREETING_STR, strlen(GREETING_STR));
    host_write(STDOUT, PROMPT_STR, strlen(PROMPT_STR));
    while(1) {
        memset(line, 0x00, LINE_MAX_CHAR);
        host_read(STDIN, line, LINE_MAX_CHAR);
        host_write(STDOUT, line, strlen(line));

        /* Show prompt */
        host_write(STDOUT, PROMPT_STR, strlen(PROMPT_STR));
    }

    return 0;
}
```

<a name="stm-4-test-code-semi"></a>
#### semihosting 程式

首先我們從手冊中知道read/write的semihosting對應代號，<font color="red">用hardcode是不道德的</font>，所以我們使用有意義的文字代替。

```c
enum HOST_SYSCALL {
    HOSTCALL_WRITE       = 0x05,
    HOSTCALL_READ        = 0x06,
};
```

接下來我們發現read/write參數的型態有整數(file descriptor)
和位址這兩種，而手冊中說明參數傳遞的方式是將參數全部放在一個連續的記憶體空間，再將該記憶體空間位址存放在`R1`暫存器中。也就是說我們需要

1. 連續的空間
2. 空間中的參數可能有多個，而且它們的型態可能都不同

要達到這樣的方式，以前上課看來的方式就是做一個`union`。每次使用時建立陣列、指令參數型態和值，接下來把該陣列的位址傳給semihosting就可以了。

以下是這次用的`union`


```c
/* Semihost system call parameters */
union param_t
{
    int   pdInt;
    void *pdPtr;
};

typedef union param_t param;
```

接下來是實際的呼叫semihosting服務。就是前面提到下break point指令。你可能會問這邊怎麼沒有參數傳遞？傻孩子，這就是ABI存在的目的了。
 
```c
static int host_call(enum HOST_SYSCALL action, void *arg) __attribute__ ((naked));
static int host_call(enum HOST_SYSCALL action, void *arg)
{
    int result;
    __asm__( \
      "bkpt 0xAB\n"\
      "nop\n" \
      "bx lr\n"\
        :"=r" (result) ::\
    );
    return result;
}
```

剩下就簡單了，設定好參數呼叫semihosting的介面收工。
```c
size_t host_read(int fd, void *buf, size_t count)
{
    param semi_param[3] = {
        { .pdInt = fd },
        { .pdPtr = buf },
        { .pdInt = count }
    };

    return host_call(HOSTCALL_READ, semi_param);
}

size_t host_write(int fd, const void *buf, size_t count)
{
    param semi_param[3] = {
        { .pdInt = fd },
        { .pdPtr = (void *) buf },
        { .pdInt = count }
    };

    return host_call(HOSTCALL_WRITE, semi_param);
}
```

<a name="stm-4-test-makefile"></a>
### Makefile
最主要的是要透過openocd提供semihosting服務。如果想要在gdb測試時打開可以參考[這邊](http://electronics.stackexchange.com/questions/149387/how-do-i-print-debug-messages-to-gdb-console-with-stm32-discovery-board-using-gd)的資料。

Makefile相關描述如下：

```Makefile
run: $(OUT_DIR)/$(TARGET).bin
	openocd -f interface/stlink-v2.cfg  \
            -f target/stm32f4x.cfg      \
            -c "init"                   \
            -c "reset init"             \
            -c "arm semihosting enable" \
            -c "reset run"
```

<a name="stm-4-test-result"></a>
### 執行結果

```text
$ make run
openocd -f interface/stlink-v2.cfg  \
            -f target/stm32f4x.cfg      \
            -c "init"                   \
            -c "reset init"             \
            -c "arm semihosting enable" \
            -c "reset run"
Open On-Chip Debugger 0.10.0-dev-00250-g9c37747 (2016-04-07-22:20)
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
adapter speed: 2000 kHz
adapter_nsrst_delay: 100
none separate
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : Unable to match requested speed 2000 kHz, using 1800 kHz
Info : clock speed 1800 kHz
Info : STLINK v2 JTAG v17 API v2 SWIM v0 VID 0x0483 PID 0x3748
Info : using stlink api v2
Info : Target voltage: 2.883392
Info : stm32f4x.cpu: hardware has 6 breakpoints, 4 watchpoints
adapter speed: 2000 kHz
stm32f4x.cpu: target state: halted
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x080005c4 msp: 0x20030000
adapter speed: 8000 kHz
semihosting is enabled
adapter speed: 2000 kHz
Hello World
> My test
My test
> !!!!
!!!!
> 
```


<a name="stm-4-ref"></a>
## 參考資料

* [ARM Compiler toolchain Developing Software for ARM Processors: Semihosting](http://infocenter.arm.com/help/topic/com.arm.doc.dui0471g/CHDJHHDI.html)
* [OpenOCD User's Guide: Architecture and Core Commands](http://openocd.org/doc/html/Architecture-and-Core-Commands.html)
* [Semihosting on ARM with GCC and OpenOCD](http://bgamari.github.io/posts/2014-10-31-semihosting.html)
    * 這個很有趣，可以用Semihosting直接使用glibc的stdio，不過做實驗就是要吃原味，要又痛又爽才是最高境界，所以這個我就沒去嘗試了
