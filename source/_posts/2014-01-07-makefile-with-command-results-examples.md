---
layout: post
title: 'Makefile結合Command Line輸出結果範例'
date: 2014-01-07 02:49
comments: true
categories: [makefile]
---
測試大檔案支援順手玩了一下Makefile撈command line 結果，整理如下

- `$(shell command)`
    - 跑command，回傳command 輸出畫面，類似command line的`$(command)`或是\`command\`
- `$(warning)`
    - 顯示警告訊息，這邊是單純拿來顯示
- ifeq字串比較記得不要亂加`"`

範例如下
```makefile Makefile example
# Warning! Not sure if there is any compatilbe problem, and surely 
# not worked on cross compilation!
IS_64=$(shell file /bin/ls|grep -o 64-bit)

ifneq ($(IS_64),64-bit)
$(warning "Set -D_FILE_OFFSET_BITS=64")
CFLAGS+=-D_FILE_OFFSET_BITS=64
endif
```
  
