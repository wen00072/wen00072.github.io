---
layout: post
title: "vim POSIX group regex 使用方式"
date: 2018-08-26 16:22:09 +0800
comments: true
categories: [vim, regex]
---
由於個人需求，需要在`vim`下面使用稍微複雜的字串搜尋取代。故整理這篇以後可以參考。

## 目錄

* [測試環境](#vreg-grp-env)
* [問題描述以及POSIX regex grouping 簡介](#vreg-grp-intro)
* [參考語法](#vreg-grp-syn)
* [範例](#vreg-grp-ex)

<a name="vreg-grp-env"></a>
## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.1 LTS
Release:	18.04
Codename:	bionic

$ vim --version
VIM - Vi IMproved 8.0 (2016 Sep 12, compiled Apr 10 2018 21:31:58)
```

<a name="vreg-grp-intro"></a>
## 問題描述以及POSIX regex grouping 簡介
Regular express 的特色是他可以match不同的pattern，所以用於搜尋和取代是非常的方便，然而當要把**符合條件的字串前面或後面加上字串**就會有一個問題，那就是**符合的字串要怎麼表示？**舉例來說，當我們想要在下面log`[mem....]` 之後放入`test`，要怎麼做到? 這時候我們就可以使用`reguler expression`的`group` 功能了

```
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x0000000000057fff] usable
[    0.000000] BIOS-e820: [mem 0x0000000000058000-0x0000000000058fff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000059000-0x000000000009dfff] usable
```

<a name="vreg-grp-syn"></a>
## 參考語法

* 指定`group`，一組regex可以指定零到多個`group` 
    * `\(match_patter\)`
* 取值
    * `\0`
        * 前項所有的`group`
    * `\1`
        * 第一組`group`
    * `\2`
        * 第二組`group`
    * `\3`
        * 第三組`group`
    * 千秋萬世直到永遠

<a name="vreg-grp-ex"></a>
## 範例
就用上面的訊息當範例吧

```
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x0000000000057fff] usable
[    0.000000] BIOS-e820: [mem 0x0000000000058000-0x0000000000058fff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000059000-0x000000000009dfff] usable
```

### 範例一： 在[mem ...]之後插入test

* 指令: `:%s/\(\[mem.*\]\)/\1 test/g`

```
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x0000000000057fff] test usable
[    0.000000] BIOS-e820: [mem 0x0000000000058000-0x0000000000058fff] test reserved
[    0.000000] BIOS-e820: [mem 0x0000000000059000-0x000000000009dfff] test usable
```

### 範例二： 設定三組`group`，都插入`test`

* 指令: `:%s/\(^\[.*\]\) \(\BIOS-e820:\) \(\[mem.*\]\)/\1 test1 \2 test2 \3 test3 /g`

```
[    0.000000] test1 BIOS-e820: test2 [mem 0x0000000000000000-0x0000000000057fff] test3  usable
[    0.000000] test1 BIOS-e820: test2 [mem 0x0000000000058000-0x0000000000058fff] test3  reserved
[    0.000000] test1 BIOS-e820: test2 [mem 0x0000000000059000-0x000000000009dfff] test3  usable
```

