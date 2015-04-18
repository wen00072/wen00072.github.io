---
layout: post
title: 'FreeRTOS 筆記'
date: 2013-10-07 13:26
comments: true
categories: [FreeRTOS]
---
[出處:The Architecture of Open Source Applications (Volume 2): FreeRTOS](http://www.aosabook.org/en/freertos.html)
## 目錄

- [簡介](#簡介)
- [架構](#架構)
- [Task Scheduling](#Task Scheduling)
- [Tasks](#Tasks)
    - [Create Task](#xTaskCreate)
- [Lists](#Lists)

<a name="簡介"></a>
## 簡介

- [FreeRTOS](www.freertos.org/‎)是基於GPLv2授權的開源自由RTOS
- 支援多種平台架構

<a name="架構"></a>
## 架構

- FreeRTOS元件有三個
    - Tasks：使用者行撰寫含有優先權的C語言程式碼，為RTOS執行的單位
    - Communication：提供Tasks和中斷之間溝通的管道
    - 硬體包裝層：提供介面分隔操作實際硬體和RTOS行為
- 軟體階層
    - User task和ISR
    - 硬體抽象化後的程式碼
    - 硬體相關程式碼
        - 放置於: FreeRTOS/Source/portable/平台
            - port.c/ portmacro.h
    - 硬體
- FreeRTOS/Demo/[平台]/FreeRTOSConfig.h存放硬體相關設定如Clock和stack size等
- FreeRTOS透過巨集來將硬體無關的程式碼對應到硬體相關的實作


<a name="Task Scheduling"></a>
## Task Scheduling

- FreeRTOS使用陣列list來管理task Prority
    - `static xList pxReadyTasksLists[ configMAX_PRIORITIES ];`
- 每次tick中斷起來就叫一次`vTaskSwitchContext()`
- `vTaskSwitchContext()`
    - 根據優先權從ready list由高到低挑出一個list
        - 如果高優先權已經沒有task，再從下一個優先權list挑
    - 從該list中挑出上一次跑過的task下一個task
    - 開始執行task

<a name="Tasks"></a>
## Tasks

- TCB: Task control block，為FreeRTOS的重要資料結構，提供
    - Stack資訊
    - List資訊
    - Event資訊
    - 優先權
    - ...
- Task狀態：每一個狀態就是對應到一個task list
    - Running
    - Ready
    - Suspend
    - Block

<a name="xTaskCreate"></a>
### Create Task流程

- 產生並設定TCB
- 設定stack及硬體context switch 會用到的暫存器
- Disable 中斷避免有人更動TCB資訊
- 把TCB加入List中、系統初始化還會Init list
    - Ready list
    - Suspend list
    - Kill list
    - Delay list
- 處理完list後enable中斷
   

<a name="Lists"></a>
## Lists

- FreeRTOS list使用doule 環狀link
- list node會指向一個TCB
- list 依`xItemValue`順序排列
- list 會指向container，該container提供
    - list node個數
    - 起始node
    - 結束node
    - 指到目前系統處理的node
