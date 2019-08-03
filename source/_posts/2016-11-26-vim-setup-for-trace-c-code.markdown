---
layout: post
title: "給自己剪貼用的vim設定"
date: 2018-02-15 04:03:41 +0800
comments: true
categories: [C, vim, cscope, ctags, vundle, Vim Plugin, Python]
---
分享使用`vim` 的心得，加上使用Vundle plugin管理工具功能配合外部程式碼分享軟體`cscope`和`ctags`來trace `C`語言的程式碼以及編輯`Python`程式碼相關設定。

* 致謝，感謝網友[Scott](http://scottt.tw/)介紹vim register概念，[葉闆](http://yodalee.blogspot.tw/)介紹的tagbar，和Kyle Lin介紹的airline。

## 目錄

* [測試環境](#vtr-env)
* [懶人包](#vtr-pkg)
* [設定.vimrc以及Vundle plugins](#vtr-set)
     *  [事前準備](#vtr-set-prep)
     *  [安裝Vundle](#vtr-set-insvd)
     *  [我安裝的Vundle Plugins](#vtr-set-vdplg)
        * [編輯器相關](#vtr-set-vdplg-ed)
            * [airline](#vtr-set-vdplg-al)
                * [安裝準備](#vtr-set-vdplg-al-pre)
                * [設定airline](#vtr-set-vdplg-al-set)
            * [indentLine](#vtr-set-vdplg-ed-itl)
            * [vim-better-whitespace](#vtr-set-vdplg-vbw)
        * [Trace C語言程式碼相關](#vtr-set-vdplg-tr)
            * [cscope_maps](#vtr-set-vdplg-tr-cm)
            * [SrcExpl](#vtr-set-vdplg-tr-se)
            * [taglist](#vtr-set-vdplg-tr-tl)
            * [nerdtree](#vtr-set-vdplg-tr-nd)
            * [Trinity](#vtr-set-vdplg-tr-tri)
            * [tagbar](#vtr-set-vdplg-tr-tgb)
        * [Markdown 語法支援](#vtr-set-md)
            * [vim-pandoc-syntax](#vtr-set-md-pdoc)
        * [Python開發相關](#vtr-set-py)
            * [準備工作](#vtr-set-py-prepare)
            * [python-mode](#vtr-set-py-pm)
            * [syntastic](#vtr-set-py-sc)
            * [python_match](#vtr-set-py-pt)
            * [indentpython](#vtr-set-py-id)
     *  [和Plugin 無關的設定](#vtr-set-misc)
        * [編輯器和顯示特殊字元相關設定](#vtr-set-misc-ed)
        * [Indent相關設定](#vtr-set-misc-ind)
        * [其他](#vtr-set-misc-msc)
* [參考資料](#vtr-test)

<a name="vtr-env"></a>
## 測試環境

```text
$ lsb_release  -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04.3 LTS
Release:	16.04
Codename:	xenial


$ ctags --version
Exuberant Ctags 5.9~svn20110310, Copyright (C) 1996-2009 Darren Hiebert
...

$ cscope --version
cscope: version 15.8b

$ vim --version
VIM - Vi IMproved 7.4 (2013 Aug 10, compiled Nov 24 2016 16:44:48)
Included patches: 1-1689
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

`Vundle`常用的指令如下，還蠻容易望文生義所以我就不解釋了

* `:PluginList`
* `:PluginInstall`
* `:PluginClean`
* `:PluginUpdate`

安裝方式如下

* 首先你要下載`Vundle`，指令如下
`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`

* 接下來在你的.vimrc加入下面這段，我是從[官方網頁](https://github.com/VundleVim/Vundle.vim)改的，其實只是把他的範例Plugin幹掉並加上分隔線及分隔線內的註解而已

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

注意下面列出的這幾行statements，你要新增或移除Plugin就是改這個地方。這些Plugin將會在後面介紹。剛好我要安裝的Plugin都是在[GitHub](https://github.com)上開發或有mirror。而`Vundle`可以用直接指定Plugin 專案在GitHub相對路徑即可安裝。這些描述也是`Vundle`載入Plugin　的順序，沒寫對順序有可能有相依問題請自行注意。

例如`https://github.com/Yggdroot/indentLine` 就寫成`Yggdroot/indentLine`

```text 我安裝的Plugin
"===============================================================
" Write your plugins here
"===============================================================
Plugin 'Yggdroot/indentLine'
Plugin 'ntpeters/vim-better-whitespace'
Plugin 'vim-airline/vim-airline'
Plugin 'tpope/vim-fugitive'
Plugin 'chazy/cscope_maps'
Plugin 'vim-scripts/taglist.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'wesleyche/SrcExpl'
Plugin 'wesleyche/Trinity'
Plugin 'majutsushi/tagbar'
```

* 確定新增/刪除Plugin後，就可以執行vim/gvim，使用下面命令
    * `:PluginInstall`
    * `:PluginClean`

<a name="vtr-set-vdplg"></a>
### 我安裝的Vundle Plugins
因為安裝方式已經在上面了，這邊就以介紹為主

<a name="vtr-set-vdplg-ed"></a>
#### 編輯器相關

<a name="vtr-set-vdplg-al"></a>
##### airline

<a name="vtr-set-vdplg-al-pre"></a>
###### 安裝準備
先看圖，圖中最下方的那行就是airline，可以顯示一些有用的資訊
<img src=/images/vim_ind11.jpg>

由左到右我們可以看到Vim 模式，Git branch 等資訊。以及一些比較特別的符號，這表示我們需要

* 讓airline取得git資訊
* 讓airline取得特別符號

也就是說，在安裝`airline`前要做一些前置動作如下

* 讓airline取得git資訊
    * 很簡單，安裝`vim-fugitive` plugin即可
* 讓airline取得特別符號
這也不難，就是安裝特殊字型，並且設定GUI時存取這些字型。方式如下

**取得字型**

```text
git clone https://github.com/powerline/fonts
```

**安裝字型**

```text
cd fonts && ./install.sh
```

** .vimrc中指定安裝的字型 **

```text
set guifont=Inconsolata\ for\ Powerline\ 20
```

<a name="vtr-set-vdplg-al-set"></a>
###### 設定airline
把下面的資料放入`.vimrc`即可

```text
let g:airline_powerline_fonts = 1
set laststatus=2
```

<a name="vtr-set-vdplg-ed-itl"></a>
##### indentLine
當Ident為空白增加以下的Indent 對齊參考資線
<img src=/images/vim_ind2.jpg>

**注意此Plugin在Ident為tab同時又加上顯示tab字元時自動失效，目前workaround就是顯示tab字元為`|`，接下來以`.`延伸作為辨別。範例如下：**
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
簡單來說，就是把cscope指令對應到Hot key

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

<a name="vtr-set-vdplg-tr-tgb"></a>
##### tagbar
網友推荐的taglist改良版 plugin，為什麼不換掉taglist呢？因為我喜歡source explorer。除了安裝Plugin外，我也順便設定按下`F7`可以切換，設定如下。

```
"====================================================================
" Tagbar Settings
"====================================================================
" Open and close the tagbar separately 
nmap <F7> :TagbarToggle<CR> 
```

以下是按下`F7` 的畫面，可以注意右邊視窗會更進一步地顯示資料結構的成員名稱
<img src=/images/vim_ind10.jpg>


<a name="vtr-set-md"></a>
### Markdown 語法支援

<a name="vtr-set-md-pdoc"></a>
#### vim-pandoc-syntax 
單純就是讓`vim`可以顯示`Markdown` syntax highlight，範例如下圖:
<img src=/images/vim_ind12.jpg>


<a name="vtr-set-py"></a>
### Python開發相關

<a name="vtr-set-py-prepare"></a>
#### 準備工作
主要是語法檢查套件相關安裝，指令如下

```
sudo apt install -y flake8 python-rope pylint
```

<a name="vtr-set-py-pm"></a>
#### python-mode
[之前](blog/2014/12/06/introduction-to-python-mode-for-vim/)有介紹過，偷懶跳過。也許Python用到一陣子可以上手後可以再分享心得。

<a name="vtr-set-py-sc"></a>
#### syntastic
泛用形語法檢查工具，請參考[Syntax checking hacks for vim](https://github.com/vim-syntastic/syntastic)說明。
目前是我靠他幫忙檢查寫的程式是否符合[PEP8](https://www.python.org/dev/peps/pep-0008/)規範，要注意的是Ubuntu 16.04中`vim` 套件預設只支援`Python 3`，要使用vim 編寫`Python 2`的朋友請自行估狗。我之前是自行編譯`vim`解決的。


語法檢查範例如下圖
<img src=/images/vim_ind13.jpg>

<a name="vtr-set-py-pt"></a>
#### python_match
讓Python 也可以使用`vim`中切換配對的快捷鍵`%`

<a name="vtr-set-py-py"></a>
#### python
提供下列快捷鍵，節錄自Plugin註解:


* `]t`      -- Jump to beginning of block
* `]e`      -- Jump to end of block
* `]v`      -- Select (Visual Line Mode) block
* `]<`      -- Shift block to left
* `]>`      -- Shift block to right
* `]#`      -- Comment selection
* `]u`      -- Uncomment selection
* `]c`      -- Select current/previous class
* `]d`      -- Select current/previous function
* `]<up>`   -- Jump to previous line with the same/lower indentation
* `]<down>` -- Jump to next line with the same/lower indentation

<a name="vtr-set-py-id"></a>
#### indentpython
確保你的程式碼符合[PEP8](https://www.python.org/dev/peps/pep-0008/)的indent規範

<a name="vtr-set-misc"></a>
### 和Plugin 無關的設定
以下都加在`.vimrc中`，建議加到`Vundle`設定結束後以確保可能會用到的Plugin已經啟動

<a name="vtr-set-misc-ed"></a>
#### 編輯器和顯示特殊字元相關設定
* 設定gvim 的配色，請自行找Color scheme
    * `colorscheme koehler`
* 設定gvim 的字型和大小
    * `set guifont=Inconsolata\ for\ Powerline\ 32` 
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
* [vim-airline](https://github.com/vim-airline/vim-airline)
* [vim-fugitive](https://github.com/tpope/vim-fugitive)
* [Powerline fonts](https://github.com/powerline/fonts)
* [Tagbar](https://majutsushi.github.io/tagbar/)
* [indentLine](https://github.com/Yggdroot/indentLine)
* [Vim Better Whitespace Plugin](https://github.com/ntpeters/vim-better-whitespace)
* [cscope maps](https://github.com/chazy/cscope_maps)
* [taglist](https://github.com/vim-scripts/taglist.vim)
* [Nerd Tree](https://github.com/scrooloose/nerdtree)
* [Source Explorer](https://github.com/wesleyche/SrcExpl)
* [Trinity](https://github.com/wesleyche/Trinity)
* [Stackoverflow: Show white space and tab in vim](http://stackoverflow.com/questions/4998582/show-whitespace-characters-in-gvim)
* Python 相關
    * Plugins
        * [Python Mode](https://github.com/python-mode/python-mode)
        * [Python](https://github.com/vim-scripts/python.vim)
        * [Indent Python](https://github.com/vim-scripts/indentpython.vim)
        * [Syntax checking hacks for vim](https://github.com/vim-syntastic/syntastic)
        * [python_match](https://github.com/vim-scripts/python_match.vim)
        * [Stackoverflow: Syntastic and Python-mode together?
](https://stackoverflow.com/questions/19209139/syntastic-and-python-mode-together)
    * `vim`設定相關
        * [Python Wiki: vim](https://wiki.python.org/moin/Vim)
        * [VIM and Python – A Match Made in Heaven](https://realpython.com/vim-and-python-a-match-made-in-heaven/)
* 和plugin無關的vim trace code資料
    * [給程式設計師的Vim入門圖解說明](http://blog.vgod.tw/2009/12/08/vim-cheat-sheet-for-programmers/)
    * [Trace 程式碼之皮投影片上線](http://wen00072.github.io/blog/2014/11/24/the-skin-slides-to-trace-code-on-line/)
* 副本: vim 的register概念
    * [usevim: Vim 101: Registers](http://usevim.com/2012/04/13/registers/)
    * [Stackoverflow: How to make vim paste from (and copy to) system's clipboard?](http://stackoverflow.com/questions/11489428/how-to-make-vim-paste-from-and-copy-to-systems-clipboard)

<a name="vtr-pkg"></a>
## 懶人包

* 安裝相關軟體，Vundle和airline字型

```text
sudo apt-get install -y exuberant-ctags cscope vim-gtk git flake8 python-rope pylint
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
git clone https://github.com/powerline/fonts
cd fonts && ./install.sh
```

* 剪貼下面的文字並存放到 ~/.vimrc

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
" Layouts
Plugin 'Yggdroot/indentLine'
Plugin 'ntpeters/vim-better-whitespace'

" Markdown
Plugin 'vim-pandoc/vim-pandoc-syntax'

" Python related
Plugin 'python-mode/python-mode'
Plugin 'vim-scripts/indentpython.vim'
Plugin 'vim-syntastic/syntastic'
Plugin 'vim-scripts/python_match.vim'
Plugin 'vim-scripts/python.vim'

" Misc tools
Plugin 'kien/ctrlp.vim'
Plugin 'vim-airline/vim-airline'
Plugin 'tpope/vim-fugitive'
Plugin 'Valloric/YouCompleteMe'
Plugin 'chazy/cscope_maps'
Plugin 'vim-scripts/taglist.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'wesleyche/SrcExpl'
Plugin 'wesleyche/Trinity'
Plugin 'majutsushi/tagbar'

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
" Tagbar Settings
"====================================================================
" Open and close the tagbar separately
nmap <F7> :TagbarToggle<CR>

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
" Airline settings
"====================================================================
let g:airline_powerline_fonts = 1
set laststatus=2

"====================================================================
" syntastic settings
"====================================================================
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*

let g:loaded_syntastic_c_checker = 1
let g:loaded_syntastic_cpp_checker = 1
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0

autocmd VimEnter * SyntasticToggleMode " disable syntastic by default

"====================================================================
" pymode settings
"====================================================================
let g:pymode_lint = 1    " Prefer to use syntastic to check lint
let g:pymode_folding = 0 " Unfold all

"====================================================================
" Editor and display Settings
"====================================================================
colorscheme koehler         " Color for gvim

set hlsearch                " Highlight search
set guifont=Inconsolata\ for\ Powerline\ 32 " Font
set cursorline              " Hight background at current cursor line
set nu                      " Display line numbers

" Set background color at colum 80
set colorcolumn=80
highlight ColorColumn guibg=#202020

" Show tabs
set listchars=tab:\|.
set list

" Ensure syntax is on
syntax on

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
" Shared unamed regitered with primary selection
set clipboard+=unnamed

" uft-8 encoding: https://stackoverflow.com/questions/16507777/set-encoding-and-fileencoding-to-utf-8-in-vim
set encoding=utf-8
set fileencoding=utf-8a

"====================================================================
" Python Settings
"====================================================================
au BufNewFile,BufRead *.py
    \ set tabstop=4 |
    \ set softtabstop=4 |
    \ set shiftwidth=4 |
    \ set textwidth=79 |
    \ set expandtab |
    \ set autoindent |
    \ set fileformat=unix
let python_highlight_all=1

"====================================================================
" pandoc Settings
"====================================================================
let g:pandoc#syntax#conceal#use = 0

"====================================================================
" YCM Settings
"====================================================================
let g:ycm_global_ycm_extra_conf = '~/.vim/bundle/YouCompleteMe/.ycm_extra_conf.py'
let g:ycm_show_diagnostics_ui = 0
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

