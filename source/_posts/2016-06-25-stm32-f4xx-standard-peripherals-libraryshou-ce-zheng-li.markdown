---
layout: post
title: "ARM CM4 Pratice (1): STM32 F4xx Standard Peripherals Library手冊整理"
date: 2016-06-25 14:03:46 +0800
comments: true
categories: [STM32, ARM]
---
## 前言

以前在寫作業的時候，從來沒有想過那些週邊到底怎麼使用，只大概印象和[CMSIS](http://www.arm.com/products/processors/cortex-m/cortex-microcontroller-software-interface-standard.php)有關。後來想應該是要去了解到底這些廠商提供的函式庫在軟體開發中扮演了什麼樣的角色？因此花了時間整理如下。

在使用之前，請自行到[官方網站](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-standard-peripheral-libraries/stsw-stm32065.html)下載STM32F4 DSP and standard peripherals library。

## 目錄

* [簡介](#AST-簡介)
    * [CMSIS和Standard Peripherals Library說明](#AST-CMSIS)
* [Coding rules and conventions](#AST-Rules)
    * [一般命名規則](#AST-Rules-general)
    * [週邊 Register 命名規則](#AST-Rules-reg)
* [參考資料](#AST-Ref)


<a name="AST-簡介"></a>
## 簡介

手冊上提到Standard Peripherals Library (以下簡稱SPL)的特點

* 提供STM32F4 XX的週邊驅動程式和常用的資料結構
* 基於CMSIS開發
* ANSI C開發
* 使用SPL表示週邊設備存取效能和佔用的記憶體空間是由SPL決定，因此有特別限制的話必須要自行最佳化或是實作

SPL包含:

* 週邊設備暫存器的位址mapping，包含這些暫存器的bit
* 所有週邊的控制function以及對應的資料結構
* 範例程式

<a name="AST-CMSIS"></a>
### CMSIS和Standard Peripherals Library說明
整體的架構圖如下(參考手冊資料）

<center><img src="/files/stm32/SPL_Diag.jpg"></img></center>

簡單說明如下

* CMSIS 提供了
    * 統一的暫存器定義、位址定義
    * 協助開發的函數
    * RTOS 介面
    * DSP 相關函數
* 開發版上實體的週邊驅動程式是透過SPL和CMSIS來控制硬體


<a name="AST-Rules"></a>
## Coding rules and conventions

<a name="AST-Rules-general"></a>
### 一般命名規則

* 函數名稱以**大寫週邊簡寫**為prefix如`USART_SendData`
* 文件舉例的API，請自行望文生義
    * `週邊名稱_Init`
    * `週邊名稱_DeInit`
    * `週邊名稱_StructInit`
    * `週邊名稱_Cmd`
    * `週邊名稱_ITConfig`
        * IT: interrupt
    * `週邊名稱_DMAConfig`
    * `週邊名稱_XXXConfig`
        * 週邊設備的XXX設定
    * `週邊名稱_GetFlagStatus`
    * `週邊名稱_ClearFlag`
    * `週邊名稱_ClearITPendingBit`

<a name="AST-Rules-reg"></a>
### 週邊 Register 命名規則

* 週邊 Register 一律包裝在`struct`中，名稱一律為大寫。在 stm32f4xx.h 另外已經宣告了週邊設備位址的結構如：

```c
#define SPI1                  ((SPI_TypeDef *) SPI1_BASE)
```

`SPI_TypeDef`為一個struct，因此你可以使用下面方式直接存取週邊設備暫存器。

```c
SPI1->CR1 = 0x0001;
```

有了大概概念後，接下來就是開幹時間，敬請期待。

<a name="AST-Ref"></a>
## 參考資料
* [STM32F4 DSP and standard peripherals library](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-standard-peripheral-libraries/stsw-stm32065.html)
    * 本篇絕大部分參考解開套件後的stm32f4xx_dsp_stdperiph_lib_um.chm　這個文件檔案
* [STM32CubeF4網頁](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32cube-embedded-software/stm32cubef4.html)
    * 有SPL的相關資料
* [STM32F2 standard peripherals library手冊](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-standard-peripheral-libraries/stsw-stm32062.html)
    * 交叉比對用

