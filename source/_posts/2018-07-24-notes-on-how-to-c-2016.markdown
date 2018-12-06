---
layout: post
title: "Notes on How to C 2016"
date: 2018-07-24 20:43:52 +0800
comments: true
categories: [C, C99]
---
最近看了[How to C 2016](https://matt.sh/howto-c)，反正很多記不起來乾脆寫起來以後剪貼。

## GNU Flags

* `-std=c99`

### Warning flags

`-Wall -Wextra -Werror -Wshadow -Wno-missing-field-initializers -Wstrict-overflow -fno-strict-aliasing`

* 延伸：[Stackoverflow: Useful gcc flag for C](https://stackoverflow.com/questions/3375697/useful-gcc-flags-for-c)

### Links

* [stdint.h](http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/stdint.h.html) 
    * `intmax_t`，...
    * `intptr_t`
        * 根據platform int size 調整
* [stdtype.h](http://pubs.opengroup.org/onlinepubs/7908799/xsh/stddef.h.html)
    * `ptr_diff_t`

## 其他

* `size_t`和`ssize_t`差別：`s`->signed，當有錯誤時會為`-1`
* `#pragma once` 可用來取代Header guard


### 名詞

* [LTO](https://gcc.gnu.org/onlinedocs/gccint/LTO-Overview.html)
