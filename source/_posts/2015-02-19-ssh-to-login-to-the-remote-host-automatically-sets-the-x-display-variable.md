---
layout: post
title: 'ssh登入遠端主機自動設定X Display 變數'
date: 2015-02-19 15:09
comments: true
categories: [Linux]
---

## 前言
---
因為用X用習慣了，常常會連線到另外遠端電腦，然後透過X把該台電腦軟體畫面輸出到本機。這時候的標準流程會是

* ssh登入到遠端
* export DISPLAY=你的本機IP:0.0

每次都要打這些，還要查自己本機IP很煩。所以整理了兩光的script提供以後剪下貼上。

另外有人會問為什麼不直接用`ssh -X`連過去？原因是因為我習慣設完display後，開一個遠端終端機丟在背景然後登出目前的ssh session，這樣的操作在`ssh -X`下從來沒有成功過。我了遠端終端機後就算放在背景，一樣目前終端機會卡住。如果暴力把目前終端機關掉，那麼所有X的session都GG。實在懶得找原因，反正這是個workaround的世界（煙）。

## 背景說明
---
上面操作中，本機需要設定

* X allow TCP
* xhost 加入允許連進來的IP

這兩個部份請自行估狗。

另外本機和遠端主機用相同的Distribution資訊如下，測試端的遠端主機是一台沒有GUI的Ubuntu server。
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty
```

## 使用方式
---
把下面的描述放在`~/.profile`最後面，下次ssh登入後就可以直接開啟X應用程式把畫面丟回本機了。

```bash
# Let's export DISPLAY if we are from ssh remote
if [ "$TERM" = "xterm" -a -n "$SSH_CLIENT" ] ; then
    REMOTE_IP=$(echo $SSH_CLIENT | cut -f 1 -d " ")
    if [ -z "$REMOTE_IP" ] ; then
        exit 0
    fi

    export DISPLAY=$REMOTE_IP:0.0
fi
```
