---
layout: post
title: '[Debian套件打包] pbuilder 中如何安裝自己打包的套件'
date: 2014-11-20 16:20
comments: true
categories: [pbuilder]
---
標題有點難下。直接講為什麼會有這樣的需求好了，[在這邊](http://wen00072-blog.logdown.com/posts/212087-package-debian-packages-discussion-on-pbuilder)介紹了使用pbuilder驗證單獨的套件。但是如果你要打包的套件需要先安裝另外一個自行先打包的套件就GG了。原因是因為pbuilder不知道去那邊找相依的套件以致於無法編譯你的套件。

舉例來說，假設你現在要打包一個套件是顯示圖片的程式，我們把他叫`pkg_image`。而這個`pkg_image`套件使用了`lib_img`這套函式庫開發。不巧的是`lib_img`並不在debian官方套件中，所以你必須要先打包`lib_img`、`lib_img-dev`和`lib_dbg`這三個套件。接下來你要確定`lib_img-dev`套件安裝到你的系統中，你才能夠開始打包`pkg_img`。

這樣的狀況在pbuilder會有前面講到的套件相依問題。後來在Debian文件的[PbuilderTricks](https://wiki.debian.org/PbuilderTricks)中找到解法，整理如下。

* [複習 & 事先準備](#prepare)
* [修改pbuilder config檔案](#conf)
* [套用並測試](#test)
* [參考資料](#ref)

<a name="prepare"></a>

## 複習 ＆ 事先準備
首先我們依照Ubuntu的文件[PbuilderHowto](https://wiki.ubuntu.com/PbuilderHowto)中的建議，安裝下面的套件。這部份應該和前面有重複。
```
sudo apt-get install pbuilder debootstrap devscripts
```

其中devscripts提供打包debian套件所需要的script，如前面提到的debuild。而debootstrap則是可以讓你在系統中安裝並執行另外一個debian Linux。在使用時可以指定要裝debian的版本，至於debootstrap支援那些版本？你可以用下面的方式查詢： `ls /usr/share/debootstrap/scripts`

如果你準備打包並上傳到debian官方套件管理庫，那麼你還要安裝debian 的GPG key，指令如下：
```
sudo apt-get install debian-archive-keyring
```

接下來我們要建立一個待測Debian的基本image。我使用

```
sudo pbuilder create --distribution sid \
                     --mirror http://ftp.tw.debian.org/debian \
                     --debootstrapopts \ 
                     "--keyring=/usr/share/keyrings/debian-archive-keyring.gpg --variant=buildd"
```

這個指令表示

* 我要針對下建立基於debian sid的base.tgz檔案: `--distribution sid`
* 指定mirror site: `--mirror http://ftp.tw.debian.org/debian`
* 傳遞下面的參數給debootstrap: `--debootstrapopts`
  * 使用debian archive的gpg key 來驗證下載的套件`--keyring=/usr/share/keyrings/debian-archive-keyring.gpg`
  * 除了基本的套件外，請幫我額外安裝編譯軟體需要的相關套件 (--variant=buildd)
  

<a name="conf"></a>

## 修改pbuilder config檔案
建立了base.tgz後。還要更改pbuilder的設定以及加入hook。請編輯/etc/pbuilderrc，或是你執行的pbuilder身份home目錄下的~/.pbuilderrc。主要是描述你的local repository放在那邊，以及hook放在那邊。local repository就是你自己打包的套件要放的地方，以前面的例子，就是要放lib_imag.deb和lib_imag-dev.deb到該目錄。而hook可以比喻成/etc/rc.d內的東西，基本上就是在該目錄寫一些script，當pbuilder啟動後就會依順序呼叫。

```text /etc/pbuilder.c
OTHERMIRROR="deb [trusted=yes] file:///path/to/the/dir/deps ./"
BINDMOUNTS="/path/to/the/dir/deps"
HOOKDIR="/path/to/hook/dir"
EXTRAPACKAGES="apt-utils"
```

請確認內容中的每個路徑都存在，還有[trusted=yes]有被加入。沒加這個在pbuilder安裝自己local套件會因為安全條件不滿足而無法安裝。

寫完設定檔後，記得要告訴pbuilder你已經更新了設定。指令如下

```
sudo pbuilder --update --override-config --distribution sid
```

接下來就是在HOOKDIR指定的目錄中填寫hook的內容，debian trick網頁是使用`D05deps`，我也是照抄。

```text D05deps
#!/bin/sh
(cd /path/to/the/dir/deps; apt-ftparchive packages . > Packages)
apt-get update
```

這個hook就是切換到你存放自己打包deb套件的目錄，然後產生出apt看得懂的套件列表到Packages檔案，最後就是更新apt。所以每次該目錄有新的deb套件時，都可以確保pbuilder一定可以使用。

<a name="test"></a>

## 套用並測試
完全和前面一樣，pbuilder --build 一個dsc檔。以前面的例子，我們會依照下面的順序打包

* `pbuilder --build lib_img.dsc`打包`lib_img`，`lib_img-dev`，`lib_img-dbg`這三個套件。
* 把`lib_img`，`lib_img-dev`，`lib_img-dbg`搬到你給pbuilder吃的local repository目錄
* `pbuilder --build pkg_image.dsc`，pbuilder會自動從local repository中找出並安裝相依套件。

<a name="ref"></a>

## 參考資料

* [PbuilderTricks](https://wiki.debian.org/PbuilderTricks)
* [(Ubuntu) PbuilderHowto](https://wiki.ubuntu.com/PbuilderHowto)
  * 後面的提到dput之後我就沒往下看了，感覺似乎和Debian的稍微不同？
* [pbuilder User's Manual](http://pbuilder.alioth.debian.org/)
* [Debian: Bootstrap](https://wiki.debian.org/Debootstrap)
