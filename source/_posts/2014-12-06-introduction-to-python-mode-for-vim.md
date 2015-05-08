---
layout: post
title: 'Python mode for vim簡介'
date: 2014-12-06 13:49
comments: true
categories: [vim, python]
---
[Python Mode](https://github.com/klen/python-mode)是一個vim下面關於Python 的plugin，快速整理一下以前看的文件。

## 目錄

* [安裝](#install)
* [使用](#use)
* [.vimrc 設定更動](#vim-settings)
* [Remarks](#remarks)
* [參考資料](#ref)


<a name="install"></a>
## 安裝

## 使用vundle

* 在.vim 加入

```
Bundle 'klen/python-mode'
```

* 在vim下打:BundleInstall
*（不確定是否必要）離開vim 再重新啟動vim
* 執行下面命令

```
:helptags ~/.vim/bundle/python-mode/doc
```

## 使用Pathogen管理下的安裝

* 下載套件

```
$ cd ~/.vim/bundle
$ git clone git://github.com/klen/python-mode.git
```
* 在.vimrc啟動，加入以下內容

```
" Pathogen load
filetype off

call pathogen#infect()
call pathogen#helptags()

filetype plugin indent on
syntax on
```

<a name="use"></a>
## 使用

* 執行
  * `\r`
    * `\`為`<leader>` key的預設值
* 除錯
  * `\b`
    * 設定中斷點，python mode會自動偵測除錯器，我使用pdb。不過設中斷點後執行之後那些除錯命令要在那邊下還不清楚。
* 查詢
  * 游標到要查詢的keyword按`K`
    * 查詢目前statement使用方式
* 查詢PyDoc資料
  * `:PymodeDoc` 你要查詢的東西
    * ex: `:PymodeDoc print`
* 區塊折疊/展開
  * `zo`
    * 展開區塊
  * `zc`
    * 折疊區塊
  * `za`
    * toggle區塊
  * 以上的操作`o`, `c`，`a`換成`O`, `C`, `A`的話表示Apply到目前區塊下面所有的子區塊
* 移動
  * `[[`
    * 移動到游標上方第一層的block，通常是在查詢目前trace到哪個function或是class
  * `]]`
    * 移動到游標下方第一層的block，通常是用於切換下一個function或是class
  * `[m`
    * 移動到游標上方class 第一個method
  * `]m`
    * 移動到游標下方class 第一個method
* 移動到定義(函數、class）
  * `CTRL + C 後按g`
* 區塊處理
  * 先定義處理方式x', x'可代換複製(y)，刪除(d)，選取(v)等
  * `x'aC`
    * 處理整個class
  * `x'iC`
    * inner, 處理目前游標包含的那整個class
  * `x'aM`
    * 處理一個整個method
  * `x'iM`
    * inner, 處理目前游標包含的那整個method


<a name="vim-settings"></a>
## .vimrc 設定更動

* 預設使用Python 2，更改使用Python 3
  * `let g:pymode_python = 'python3'`
    * 注意！這招要確認你的vim已經支援Python3，確認方式
    * `$ vim --version | grep python3` 應該要有+python3，目前Ubuntu 14.04.1沒有看到設定語法檢查工具，放這邊主要是可以檢查是否已經裝了相關套件了。

* let g:pymode_lint_checkers = ['pyflakes', 'pep8', 'mccabe']
  * Refactor，使用rope，我沒使用過rope，故跳過


<a name="remarks"></a>
## Remarks
我安裝了下面兩個套件，用來讓Python Mode檢查語法。不過可以明顯感覺到用了速度變很慢，目前還沒打算去看是否設定有錯誤。

* `sudo apt-get install python-rope`
* `sudo apt-get install pylint`

<a name="ref"></a>
## 參考資料

* [Python Mode說明](https://github.com/klen/python-mode/blob/develop/doc/pymode.txt)
