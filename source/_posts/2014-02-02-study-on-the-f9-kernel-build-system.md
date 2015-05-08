---
layout: post
title: 'F9-Kernel初探 -  Build system'
date: 2014-02-02 13:19
comments: true
categories: [F9, RTOS]
---
## MACRO
### Platform module

- `-D __CHIP__=xxx`
- `-D'INC_PLAT(x)=<platform/__CHIP__/x>'`
- `#include INC_PLAT(gpio.h)`
```
$ find |grep gpio.h
./include/platform/stm32f4/gpio.h

```


## compiler front end options

- `-std=gnu99` 
- `-isystem`
- `-nostdlib` 
- `-ffreestanding`
- `-mcpu=cortex-m4`
- `-mthumb` 
- `-mthumb-interwork`
    - Specify that the code has been generated with interworking between Thumb and ARM code in mind.       
- `-Xassembler -mimplicit-it=thumb -mno-sched-prolog -mno-unaligned-access`
- `-Wdouble-promotion`
- `-fsingle-precision-constant`
- `-fshort-double`  
- `-fno-toplevel-reorder` 
- `-fdata-sections` 
- `-ffunction-sections` 
- `-g3` 
- `-Wall` 
- `-Werror` 
- `-Wundef` 
- `-Wstrict-prototypes` 
- `-Wno-trigraphs` 
- `-fno-strict-aliasing` 
- `-fno-common` 
- `-Werror-implicit-function-declaration`
- `-Wno-format-security` 
- `-fno-delete-null-pointer-checks`
- `-Wdeclaration-after-statement` 
- `-Wno-pointer-sign` 
- `-fno-strict-overflow` 
- `-fconserve-stack`
- `-MMD` 
- `-MF`

## Build directory order

1. platform/stm32f4/
2. board/discoveryf4/
3. platform/
4. kernel/lib/
5. kernel/
6. user/
7. user/lib/l4/platform/
8. user/lib/io/
9. user/apps/

## .config

- Platform related
- Limitation
- Kernel timer
- Flexiable page
- Thread
- KIP
- Kernel debug

## Build binaries
### Root Makefile

- Options
    - BOARD
    - PROJECT
- Decide which files to compiled and linked

### Implicit Rules

- CC
    - mk/generic.mk
- Loader
    - mk/loader.mk


## Questions

- Makefile
    - eval command?
