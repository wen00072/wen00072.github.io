---
layout: post
title: "bash下自動完成gvim --remote-tab"
date: 2017-03-28 21:15:16 +0800
comments: true
categories: [vim]
---
在gvim需要開tab要打`--remote-tab`，打久就開始厭煩想偷懶。後來整理網路上的bash_completion相關資料拼湊一個堪用的版本分享一下。因為我只是想偷懶，所以完全沒有去了解bash_completion的細節。嘛，反正可以組裝需要的功能就好。

## 目錄

* [測試環境](#ac_env)
* [安裝方式](#ac_inst)
* [使用方式](#ac_use)
* [gvim bash_completion script](#ac_gvf)
* [參考資料](#ac_ref)

<a name="ac_env"></a>
## 測試環境

```
$ lsb_release  -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.2 LTS
Release:	16.04
Codename:	xenial
```

<a name="ac_inst"></a>
## 安裝方式

1. 將[下面](#ac_gvf)文字貼到編輯器，存成`/etc/bash_completion.d/gvim`
2. 使用下面指令新增gvim completion

```text 新增gvim completion指令
$ . /etc/bash_completion.d/gvim
``` 

<a name="ac_use"></a>
## 使用方式

* 一般使用，直接gvim `tab` 顯示檔案或目錄

```
$ gvim `tab`
changelog.Debian.gz  copyright            README.emacs         RelNotes/
changelog.gz         NEWS.Debian.gz       README.md            
contrib/             README.Debian        README.source 
```

* 要使用tab時，下gvim -`tab` 就會自動填入`--remote-tab`，接下來再按`tab`即可選擇檔案或目錄

```
$ gvim --remote-tab 
changelog.Debian.gz  copyright            README.emacs         RelNotes/
changelog.gz         NEWS.Debian.gz       README.md            
contrib/             README.Debian        README.source 
```

<a name="ac_gvf"></a>
## gvim bash_completion script

```sh gvim
_gvim()
{
    local cur prev
    COMPREPLY=()
    cur="${COMP_WORDS[COMP_CWORD]}"
    prev="${COMP_WORDS[COMP_CWORD-1]}"
    opts="--remote-tab"

    if [[ ${cur} == -* ]] ; then
        COMPREPLY=( $(compgen -W "${opts}" -- ${cur}) )
        return 0
    fi

    compopt -o default; COMPREPLY=()
    return 0
}
complete -F _gvim gvim
```

<a name="ac_ref"></a>
## 參考資料

* [An introduction to bash completion: part 1](https://debian-administration.org/article/316/An_introduction_to_bash_completion_part_1)
* [An introduction to bash completion: part 2](https://debian-administration.org/article/317/An_introduction_to_bash_completion_part_2)
* [stackoverflow: Getting compgen to include slashes on directories when looking for files](http://stackoverflow.com/questions/12933362/getting-compgen-to-include-slashes-on-directories-when-looking-for-files)


