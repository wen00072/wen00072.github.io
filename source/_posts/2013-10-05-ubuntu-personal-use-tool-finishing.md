---
layout: post
title: '[Ubuntu 12.04] 安裝完系統後新增工具整理'
date: 2013-10-05 00:11
comments: true
categories: [ubuntu, Linux_Utilities]
---
安裝完作業系統後還需要裝一些平常使用的工具，順手整理一下

## 目錄

- [懶人包](#懶人包)
- [工具分類及簡單說明](#工具分類及簡單說明)
     - [系統程式開發套件](#系統程式開發套件)
     - [程式開發輔助工具](#程式開發輔助工具)
     - [一般工具](#一般工具)

<a name="懶人包"></a>
## 懶人包
```
sudo apt-get install build-essential
sudo apt-get install ccache
sudo apt-get install bison 
sudo apt-get install flex
sudo apt-get install automake
sudo apt-get install libtools
sudo apt-get install intltool
sudo apt-get install libgtk2.0-dev
sudo apt-get install libncurses5-dev

sudo apt-get install git-svn
sudo apt-get install tig
sudo apt-get install meld
sudo apt-get install cscope
sudo apt-get install vim-gtk
sudo apt-get install ttf-inconsolata
sudo apt-get install geany
sudo apt-get install joe
sudo apt-get install doxygen-gui
sudo apt-get install exuberant-ctags
sudo apt-get install manpages-dev
sudo apt-get install manpages-posix-dev
sudo apt-get install minicom

sudo apt-get install tree
sudo apt-get install terminator
sudo apt-get install ttf-mscorefonts-installer 
sudo apt-get install ack-grep
sudo apt-get install pandoc
sudo apt-get install gnome-system-tools
sudo apt-get install ghex
sudo apt-get install mc
sudo apt-get install dict
sudo apt-get install wireshark
sudo apt-get install mtr
sudo apt-get install pdfgrep
```
<a name="工具分類及簡單說明"></a>
## 工具分類及簡單說明
<a name="系統程式開發套件"></a>
### 系統程式開發套件 (不解釋)

- build-essential
- ccache
- bison 
- flex
- automake
- libtools
- intltool
- libgtk2.0-dev
- libncurses5-dev

<a name="程式開發輔助工具"></a>

### 程式開發輔助工具

- git-svn
    - git, svn, git-svn三個願望一次滿足
- tig
    - 文字介面的git管理工具
- meld
    - GUI介面的檔案比較工具
- cscope
    - 追蹤程式碼工具，可以查詢函數被誰呼叫，宣告等功能。請配合vim服用
- exuberant-ctags
    - 追蹤程式碼工具，可以切入函數呼叫等功能。請配合vim服用
- vim-gtk
    - vim和gvim兩個願望一次滿足
- ttf-inconsolata
    - 給開發程式時的GUI編輯器使用，因為不希望因為粗體斜體等效果干擾字元對齊
- geany
    - 簡單好用的GUI編輯
- joe
    - 文字介面的WordStar式介面編輯器
- doxygen-gui
    - 文件產生器和他的Front UI
- ghex
    - GUI 16進位檔案檢視工具
- manpages-dev
    - 更多男人
- manpages-posix-dev
    - POSIX男人，查phread用
- minicom
    - Serial port終端機工具

<a name="一般工具"></a>
### 一般工具

- tree
    - 文字介面使用樹狀結構顯示檔案
- terminator
    - 增強版gnome-terminal，可以同一個畫面任意新增分割的終端
- ttf-mscorefonts-installer 
    - 主要是要Wingdings字型，有一堆有趣的向量ICON可以用
- ack-grep
    - 學長推荐的加強版grep，還不熟
- pandoc
    - 吃markdown語法產生其他文件如pdf
- gnome-system-tools
    - 需要GUI使用者管理所以安裝
- mc
    - 文字介面的norton commander介面式檔案管理工具，除了複製搬移外、還可以解壓縮檔案
- dict
    - 文字介面查英文單字用
- wireshark
    - 分析封包的工具
- mtr
    - 加強版的ping + tracert工具
- pdfgrep
    - grep pdf檔案的工具
