---
layout: post
title: ' ProGit: Customizing Git - Git Configuration導讀'
date: 2015-03-08 22:12
comments: true
categories: [git]
---
## 目錄

* [測試環境](#g3_conf_env)
* [簡易使用方式](#g3_conf_use)
* [git config參考檔案以及順序](#g3_conf_file)
* [git config 選項節錄](#g3_conf_opt)
	* [指定使用外部比較命令](#g3_conf_ext)
* [參考資料](#g3_conf_ref)


<a name="g3_conf_env"></a>
## 測試環境
```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty

$ git --version
git version 2.3.0

$ git remote -v
origin	https://github.com/pcman-bbs/pcmanx.git (fetch)
origin	https://github.com/pcman-bbs/pcmanx.git (push)
```


<a name="g3_conf_use"></a>
## 簡易使用方式
最簡單的使用方式

* 新增/更改選項
	* git config 型態 名稱 你要設定的值
		* 常見的型態有`--system`或是`--global`
* 讀取選項
	* git config --get 名稱
  
值得注意的是，git config 的名稱一般來說會使用`.`作為分類的參考如
`git config test.user.name.first Liao`
`git config test.user.name.last Wen`
而在設定檔中，會用`[]`把第一個`.`的前面的字串放在前面，然後最後一個`.`後面的字串和你要設定的值變成
* `最後一個.後面的字串=你要設定的值`

你應該要問，那中間的呢？好問題，中間的就放在`[]`裏面囉。

很模糊？看例子比較快。

```
$ git config test.user.name.first Liao
$ git config test.user.name.last Wen
$ cat .git/config
...
[test "user.name"]
	first = Liao
	last = Wen
```


<a name="g3_conf_file"></a>
## git config參考檔案以及順序
要注意的是/etc/gitconfig不一定存在，我的是沒有。所以生一個測試驗證用。

* `/etc/gitconfig`
	* 由`git config --system`產生
* `~/.gitconfig`
	* 由`git config --global`產生
* `你的local repostiory/.git/config`
	* 在`你的local repostiory`下面執行`git config`產生

檔案讀取順序是由上往下，全部都有資料的話。最後回傳值順序是由下而上。
要練習驗證，所以來看一下。

用strace看`git config --get user.name`開檔案的順序。
```
$ strace -e open git config --get user.name
...
open("/etc/gitconfig", O_RDONLY)        = 3
open("/home/user/.gitconfig", O_RDONLY)  = 3
open(".git/config", O_RDONLY)           = 3
...
Wen.Liao
+++ exited with 0 +++
```

可以看到`/etc/gitcnfig`沒有user.name
```
$ cat /etc/gitconfig  | grep name
```

`~/.gitconfig`下有Wen.Liao的字串
```
$ cat ~/.gitconfig 
[user]
	name = Wen.Liao
...
```

同樣的，目前repository下面的config是沒有user.name資料
```
cat .git/config  | grep name
```

接下來改一下user.name
```
$ strace -e open git config user.name qqq
...
open("/etc/gitconfig", O_RDONLY)        = 3
open("/home/user/.gitconfig", O_RDONLY)  = 3
open(".git/config", O_RDONLY)           = 3
...
+++ exited with 0 +++
```

可以看到目前目錄下`.git/config`新增了user.name
```
$ cat .git/config 
...
[user]
	name = qqq
```

驗證是否`.git/config`的值真的會比`~/.gitconfig`優先
```
$ git config --get user.name
qqq
```


<a name="g3_conf_opt"></a>
## git config 選項節錄
Git config 選項有分為server端和client端，大部分選項和client有關，文件中節錄部份，我再節錄我有興趣的部份。想要知道全部的選項可以問男人。`man git config`。再次提醒，如果是你要套用到全部git repository請用`git config --global`，不然這些選項只會在你目前git repository生效。

* `core.editor`
	* 你要commit 時候會用的編輯器
* `commit.template`
  * 這個比較重要，如果你要套用commit template，就用這個。把你自己或是團隊的commit template引入吧。我自己會用# 為template加入註解。
* `color.ui`
	* 為你的UI文字上色。可以設成`true` 或是`false`


<a name="g3_conf_ext"></a>
## 指定使用外部比較命令
git可以指令使用外部比較命令，首先要搞清楚git 會送什麼參數給外部命令，所以我們先寫個script印出參數，並且指定外部diff的時候使用這個script。

```
$ cat ~/bin/git_diff.sh 
#!/bin/sh
echo $@

$ git config --global diff.external ~/bin/git_diff.sh

$ git diff HEAD^
src/core/caret.cpp /tmp/6nZQoO_caret.cpp 99fdca2d1d2b90e6452555572d3b0c3cf3ae75b0 100644 src/core/caret.cpp 15b662547eb5a853f02a6fc086e07feed2e45d4f 100644
```

這些參數的順序是

1 更動的檔案名稱
2 將被更動前的檔案內容另存新檔路徑
3 被更動前的檔案blob hash
4 被更動前檔案權限
5 將被更動後的檔案內容另存新檔路徑
6 被更動後的檔案blob hash
7 被更動後檔案權限

對我們來說，有興趣的是2和5，所以我們可以將命令更動為
```
$ cat ~/bin/git_diff.sh 
#!/bin/sh -x
echo $@
meld $2 $5
```

meld是一個GUI程式的比對工具。接下來就來測試吧

```
$ git diff HEAD^
+ echo src/core/caret.cpp /tmp/0M6Fgi_caret.cpp 99fdca2d1d2b90e6452555572d3b0c3cf3ae75b0 100644 src/core/caret.cpp 15b662547eb5a853f02a6fc086e07feed2e45d4f 100644
src/core/caret.cpp /tmp/0M6Fgi_caret.cpp 99fdca2d1d2b90e6452555572d3b0c3cf3ae75b0 100644 src/core/caret.cpp 15b662547eb5a853f02a6fc086e07feed2e45d4f 100644
+ meld /tmp/0M6Fgi_caret.cpp src/core/caret.cpp
```

GUI畫面如下:
![](https://www.dropbox.com/s/h68sor0kwkwtewg/meld_git.png?raw=1)

手冊上面還有提到merge tools 的設定，本人不感興趣。跳過。


<a name="g3_conf_ref"></a>
## 參考資料

* [ProGit: 8.1 Customizing Git - Git Configuration](http://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration#_git_config)
