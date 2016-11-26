---
layout: post
title: "Trace C 程式碼的vim設定"
date: 2016-11-26 04:03:41 +0800
comments: true
categories: [C, vim, cscope, ctags, vundle, Vim Plugin]
---
分享使用`vim` 的心得，加上使用Vundle plugin管理工具功能配合外部程式碼分享軟體`cscope`和`ctags`來trace C語言的程式碼。

## 目錄

* [測試環境](#vtr-env)
* [設定.vimrc以及Vundle plugins](#vtr-set)
     *  [事前準備](#vtr-set-prep)
     *  [安裝Vundle](#vtr-set-insvd)
     *  [我安裝的Vundle Plugins](#vtr-set-vdplg)
        * [編輯器相關](#vtr-set-vdplg-ed)
            * [indentLine](#vtr-set-vdplg-ed-itl)
            * [vim-better-whitespace](#vtr-set-vdplg-vbw)
        * [Trace 程式碼相關](#vtr-set-vdplg-tr)
            * [cscope_maps](#vtr-set-vdplg-tr-cm)
            * [SrcExpl](#vtr-set-vdplg-tr-se)
            * [taglist](#vtr-set-vdplg-tr-tl)
            * [nerdtree](#vtr-set-vdplg-tr-nd)
            * [Trinity](#vtr-set-vdplg-tr-tri)
     *  [和Plugin 無關的設定](#vtr-set-misc)
        * [編輯器和顯示特殊字元相關設定](#vtr-set-misc-ed)
        * [Indent相關設定](#vtr-set-misc-ind)
        * [其他](#vtr-set-misc-msc)
* [參考資料](#vtr-test)
* [懶人包](#vtr-pkg)

<a name="vtr-env"></a>
## 測試環境

```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.5 LTS
Release:	14.04
Codename:	trusty

$ ctags --version
Exuberant Ctags 5.9~svn20110310, Copyright (C) 1996-2009 Darren Hiebert
...

$ cscope --version
cscope: version 15.8a

$ vim --version
VIM - Vi IMproved 7.4 (2013 Aug 10, compiled Jan  2 2014 19:39:47)
...
```

<a name="vtr-set"></a>
## 設定.vimrc以及Vundle plugins

<a name="vtr-set-prep"></a>
### 事前準備
您需要確認

* vim版本為7.4以上
* 安裝ctags和cscope，指令如下
`sudo apt-get install exuberant-ctags cscope`

<a name="vtr-set-insvd"></a>
### 安裝Vundle
`Vundle`是vim plugin 管理工具，他可以透過URL, github, 以及local FS等方式安裝甚至更新Plugin。類似的工具還有不少，我只是挑看到的第一個而已。

`Vundle`常用的指令如下，還蠻容易望文生義所以我就不解試了

* `:PluginList`
* `:PluginInstall`
* `:PluginClean`
* `:PluginUpdate`

安裝方式如下

* 首先你要下載`Vundle`，指令如下
`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`

* 接下來在你的.vimrc加入下面這段，我是從[官方網頁](https://github.com/VundleVim/Vundle.vim)改的，其實只是把他的範例Plugin幹掉並加上分隔線

```text .vimrc 要加的部份
"====================================================================
" Start vundle
"====================================================================
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

"===============================================================
" Write your plugins here
"===============================================================
Plugin 'Yggdroot/indentLine'
Plugin 'ntpeters/vim-better-whitespace'
Plugin 'chazy/cscope_maps'
Plugin 'vim-scripts/taglist.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'wesleyche/SrcExpl'
Plugin 'wesleyche/Trinity'

"====================================================================
" Run vundle
"====================================================================
" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

注意上面的這幾行statement，你要新增或移除Plugin就是改這個地方。這些Plugin將會在後面介紹。剛好我要安裝的Plugin都是在[GitHub](https://github.com)上開發或有mirror。而`Vundle`可以用直接指定Plugin 專案在GitHub相對路徑即可安裝。這些描述也是`Vundle`載入Plugin　的順序，沒寫對順序有可能有相依問題請自行注意。

例如`https://github.com/Yggdroot/indentLine` 就寫成`Yggdroot/indentLine`

```text 我安裝的Plugin
"===============================================================
" Write your plugins here
"===============================================================
Plugin 'Yggdroot/indentLine'
Plugin 'ntpeters/vim-better-whitespace'
Plugin 'chazy/cscope_maps'
Plugin 'vim-scripts/taglist.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'wesleyche/SrcExpl'
Plugin 'wesleyche/Trinity'
```

* 確定新增/刪除Plugin後，就可以執行vim/gvim，使用下面命令
    * `:PluginInstall`
    * `:PluginClean`

<a name="vtr-set-vdplg"></a>
### 我安裝的Vundle Plugins
因為安裝方式已經在上面了，這邊就以介紹為主

<a name="vtr-set-vdplg-ed"></a>
#### 編輯器相關

<a name="vtr-set-vdplg-ed-itl"></a>
##### indentLine
當Ident為空白增加以下的Indent 對齊參考資線
<img src=/images/vim_ind2.jpg>

**注意此Plugin在Ident為tab同時又顯示tab時自動失效**
<img src=/images/vim_ind1.jpg>

<a name="vtr-set-vdplg-vbw"></a>
##### vim-better-whitespace
將[trailing space](http://stackoverflow.com/questions/22273233/what-is-meant-by-
trailing-space-and-whats-the-difference-between-it-and-a-blank)顯示成明顯的紅色
<img src=/images/vim_ind3.jpg>

<a name="vtr-set-vdplg-tr"></a>
#### Trace 程式碼相關

<a name="vtr-set-vdplg-tr-cm"></a>
##### cscope_maps
簡單來說，把cscope指令對應到Hot key

先列出find部份的指令
```text
find : Query for a pattern            (Usage: find c|d|e|f|g|i|s|t name)
       c: Find functions calling this function
       d: Find functions called by this function
       e: Find this egrep pattern
       f: Find this file
       g: Find this definition
       i: Find files #including this file
       s: Find this C symbol
       t: Find this text string   
```

他的使用方法也很簡單，就是先把游標移動到你要查的statement，再按`ctrl` + `\` + `c|d|e|f|g|i|s|t 其中一個`。

舉例來說，我在下圖中把游標移動到`core_sys_select`函數後按下`ctrl` + `\` + `c`的結果如下
<img src=/images/vim_ind4.jpg>


<a name="vtr-set-vdplg-tr-se"></a>
##### SrcExpl
當啟動時，您的游標在那個敘述，Source explorer 會切割視窗，印出該敘述的定義。舉例來說，當我游標在138行的`free_poll_entry`的話，顯示的畫面如下。

<img src=/images/vim_ind5.jpg>

<a name="vtr-set-vdplg-tr-tl"></a>
##### taglist
列出目前檔案所有symbol並且可以選擇symbol切換到該symbol在檔案中的位置
<img src=/images/vim_ind6.jpg>

<a name="vtr-set-vdplg-tr-nd"></a>
##### nerdtree
以樹狀顯示目前檔案所在目錄結構，看圖就知道
<img src=/images/vim_ind7.jpg>

<a name="vtr-set-vdplg-tr-tri"></a>
##### Trinity
看完以上三個，你可能會覺得奇怪好像沒提到怎麼啟動。這就是`Trinity`大顯身手的地方了。你安裝Trinity後，再Vundle後面加上下面的敘述就可以有

* `F8` : 同時打開或關閉`nerdtree`, `Source explorer`, 以及`tag list`
* `F9` : 打開或關閉`Source explorer`
* `F10`: 打開或關閉`tag list`
* `F11`: 打開或關閉`nerdtree`

```text .vimrc 的Trinity設定
"====================================================================
" Trinity Settings
"====================================================================
" Open and close all the three plugins on the same time 
nmap <F8>  :TrinityToggleAll<CR> 

" Open and close the Source Explorer separately 
nmap <F9>  :TrinityToggleSourceExplorer<CR> 

" Open and close the Taglist separately 
nmap <F10> :TrinityToggleTagList<CR> 

" Open and close the NERD Tree separately 
nmap <F11> :TrinityToggleNERDTree<CR> 
```

以下是按下`F8` 的畫面
<img src=/images/vim_ind8.jpg>

<a name="vtr-set-misc"></a>
### 和Plugin 無關的設定
以下都加在`.vimrc中`，建議加到`Vundle`設定結束後以確保可能會用到的Plugin已經啟動

<a name="vtr-set-misc-ed"></a>
#### 編輯器和顯示特殊字元相關設定
* 設定gvim 的配色，請自行找Color scheme
    * `colorscheme koehler`
* 設定gvim 的字型和大小
    * `set guifont=inconsolata\ 20`
* 將找到的字串設成高亮度
    * `set hlsearch`
* 游標在的該行背景高亮度
    * `set cursorline`
* 顯示行號
    * `set nu`
* 第八十字元地方顯示高亮度區塊（這是連續兩個描述)
    * `set colorcolumn=80`
    * `highlight ColorColumn guibg=#202020`
* 顯示tab （這是連續兩個描述)
    * `set listchars=tab:»\ `
        * **注意`\`後面有一個空白**
    * `set list`

效果如下圖
<img src=/images/vim_ind9.jpg>

<a name="vtr-set-misc-ind"></a>
#### Indent相關設定

* Tab相關設定
    * `set ts=4`
        * tab space 為4個字元
    * `set expandtab`
        * 不使用tab，用空白字元代替
    * `set shiftwidth=4`
        * Auto indent的移動字元數量
* visual 模式下一次移動一個indent
    * `vnoremap < <gv`
        * 往左移動一個indent
    * `vnoremap > >gv`
        * 往右移動一個indent

<a name="vtr-set-misc-msc"></a>
#### 其他
* `set clipboard+=unnamed`
    * PRIMARY selection的register `"*`包含vim的unnamed register。白話講就是其他的APP如gedit中滑鼠選字後可以用`"*p`貼到vim，同樣的`"*y6y`的結果可以貼在其他的APP如gedit上。這部份有vim `register`副本，建議到參考資料的副本區一讀。再次感謝[Scott](http://scottt.tw/)大大。

<a name="vtr-ref"></a>
## 參考資料

* [Vundle Github](https://github.com/VundleVim/Vundle.vim)
* [indentLine](https://github.com/Yggdroot/indentLine)
* [Vim Better Whitespace Plugin](https://github.com/ntpeters/vim-better-whitespace)
* [cscope maps](https://github.com/chazy/cscope_maps)
* [taglist](https://github.com/vim-scripts/taglist.vim)
* [Nerd Tree](https://github.com/scrooloose/nerdtree)
* [Source Explorer](https://github.com/wesleyche/SrcExpl)
* [Trinity](https://github.com/wesleyche/Trinity)
* [Stackoverflow: Show white space and tab in vim](http://stackoverflow.com/questions/4998582/show-whitespace-characters-in-gvim)
* 和plugin無關的vim trace code資料
    * [給程式設計師的Vim入門圖解說明](http://blog.vgod.tw/2009/12/08/vim-cheat-sheet-for-programmers/)
    * [Trace 程式碼之皮投影片上線](http://wen00072.github.io/blog/2014/11/24/the-skin-slides-to-trace-code-on-line/)
* 副本: vim 的register概念
    * [usevim: Vim 101: Registers](http://usevim.com/2012/04/13/registers/)
    * [Stackoverflow: How to make vim paste from (and copy to) system's clipboard?](http://stackoverflow.com/questions/11489428/how-to-make-vim-paste-from-and-copy-to-systems-clipboard)

<a name="vtr-pkg"></a>
## 懶人包

* 安裝相關軟體和Vundle

```text
sudo apt-get install exuberant-ctags cscope vim-gtk git
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

* 剪貼下面的文字並放到 ~/.vimrc

```text .vimrc
"====================================================================
" Start vundle
"====================================================================
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

"===============================================================
" Write your plugins here
"===============================================================
Plugin 'Yggdroot/indentLine'
Plugin 'ntpeters/vim-better-whitespace'
Plugin 'chazy/cscope_maps'
Plugin 'vim-scripts/taglist.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'wesleyche/SrcExpl'
Plugin 'wesleyche/Trinity'

"====================================================================
" Run vundle
"====================================================================
" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

"====================================================================
" Trinity Settings
"====================================================================
" Open and close all the three plugins on the same time 
nmap <F8>  :TrinityToggleAll<CR> 

" Open and close the Source Explorer separately 
nmap <F9>  :TrinityToggleSourceExplorer<CR> 

" Open and close the Taglist separately 
nmap <F10> :TrinityToggleTagList<CR> 

" Open and close the NERD Tree separately 
nmap <F11> :TrinityToggleNERDTree<CR> 

"====================================================================
" Editor and display Settings
"====================================================================
colorscheme koehler         " Color for gvim

set hlsearch                " Highlight search
set guifont=inconsolata\ 20 " Font 
set cursorline              " Hight background at current cursor line
set nu                      " Display line numbers

" Set background color at colum 80
set colorcolumn=80
highlight ColorColumn guibg=#202020

" Show tabs
set listchars=tab:»\ 
set list

"====================================================================
" Indent Settings
"====================================================================
" Tabs
set ts=4
set expandtab
set shiftwidth=4

" visual indent shift
vnoremap < <gv
vnoremap > >gv

"====================================================================
" MISC Settings
"====================================================================
" Shared with primary selection
set clipboard+=unnamed
```

* gvim -> `:PluginInstall` 安裝Plugin重新開啟收工

### 其他

* cscope產生database供vim使用
    * `cscope -bqkR` 
        * k表示使用kernel mode，不把/usr/include之類的加入資料庫。Cross compile也不會使用host 的header file，所以請自行斟酌。其他參數請自己問男人。
* ctags產生database供vim使用
    * `ctags -R`
* 要在vim使用到ctags和cscope的話，請記得**vim一定要開在database同一層目錄!**

舉例來說，你在/tmp/linux-stable目錄下了上面兩個指令。要開啟檔案請在/tmp/linux-stable目錄中指定對應路徑。範例如下

```text
user@host:/tmp/linux-stable$ gvim fs/select.c
```

