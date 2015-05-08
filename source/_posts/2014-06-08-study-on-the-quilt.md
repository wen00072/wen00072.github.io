---
layout: post
title: 'quilt初探 -  使用quilt產生和管理patch'
date: 2014-06-08 07:33
comments: true
categories: [quilt, patch]
---
傳統的組裝的情況，常常會需要處理patch，然而當patch數量資料多到某種程度，加上時間推移原資料更動，套用patch會讓人生不如死。而quilt是Linux下面處理多個patch檔案的管理軟體。因為組裝軟體和打包Linux 套件會使用到，所以花點時間看一下整理一下。目前感覺有點像是先用git format-patch產生一些patch檔，另外一邊用git am 去merge這些patch檔。然後merge完可以用git reset HEAD^幾次來unmerge。

## 目錄

* [透過quilt產生patch](#usage)
	* [quilt產生的檔案](#files)
* [透過quilt管理patch](#apply)
* [~/.quiltrc](#rc)
* [參考資料](#ref)

quilt的格式是`quilt 命令`，**而它處理的對象是stack**，也就是說你可以使用push, pop。沒有特別指定的話，command處理的對象是stack top。

---
<a name="usage"></a>
## 透過quilt產生patch

* `quilt new 產生的patch檔案名稱`
* `quilt add 要監督那個檔案`
* 更動檔案。
* `quilt refresh`產生patch

重複前面動作會再產生新的patch，置於stack的top。可以透過`quilt series`觀察stack的狀態。另外有兩點值得注意：

* 假設都是重複更動同一份檔案，`quilt refresh`就是單純和最原始的檔案diff。舉例來說，下了`quilt add`後修改了三行再執行`quilt refresh`產生patch，之後再修改另外五行後執行`quilt refresh`原本的patch檔案會有第一次改的那三行再加上後來改的那五行。
* 一個檔案a，經過`quilt new` ,`quilt add`, `quilt refresh`後，再用quilt產生另外一份檔案的patch後。要再修改檔案a會需要經過`quilt new` ,`quilt add`, `quilt refresh`產生另外全新的patch檔案。
* 可以使用`quilt header -e`去編輯patch檔的詳細說明。另外也可以用`quilt header -e patch名稱`指定編輯特定patch名稱的詳細說明。

quilt會將產生的patch放在目前目錄下的`patches`目錄，這部份可以透過更改`QUILT_PATCHES`的變數更改名稱。

---
<a name="files"></a>
### quilt產生的檔案
`quilt new 要產生的patch檔案`被執行後會產生

* pactches目錄
	* 紀錄patch 順序的series檔案
* .pc目錄
	* applied-patches，和series相同
  * `要產生的patch檔案名稱`的目錄

`quilt add 要監督那個檔案`被執行後會產生
* .pc目錄/要產生的patch檔案名稱的目錄/監督檔案路徑/監督檔案
	* ex: ./pc/fix_err_no_27113/libs/liba.c
  
`quilt refresh`產生patch

* .pc目錄/要產生的patch檔案名稱的目錄/.timestamp
* patches/要產生的patch檔案名稱

**我們可以觀察發現`quilt add`後quilt會將未修改的原始檔複製一份到.pc目錄/要產生的patch檔案名稱的目錄/監督檔案路徑/監督檔案。之後`quilt refresh`就是單純的diff產生結果**

---
<a name="apply"></a>
## 透過quilt管理patch
apply patch並管理的步驟是。

* 取得未apply patch的原始套件目錄。
* 將前面產生出來的patches目錄複製到未apply patch的原始的套件目錄當中。
* 透過quilt 操作並管理patch

假設你有了patches目錄，你可以做以下操作

* 查詢
  * `quilt series`查詢有哪些patch
  * `quilt applied`查詢已經applied那些patch
  * `quilt unapplied`查詢還沒apply那些patch
  * `quilt next`查詢下一個要apply的patch
  * `quilt next`查詢前一個已經applied的patch
  * `quilt header`顯示目前stack top的註解說明資訊及要patch的檔案
  * `quilt header patch名稱`顯示目前stack中patch名稱的註解說明資訊及要patch的檔案
* 套用/還原patch
  * `quilt push`依照stack反順序apply patch
    * `quilt push -a`可以一口氣apply patch
    * `quilt push patch名稱`
  * `quilt pop`undo 一次已經applied的patch。
    * `quilt pop -a`可以一口氣apply patch

舉例來說，假設patches目錄stack有`01_fix`, `02_fix`和`03_fix`。將該目錄複製到未patch的目錄中，下兩次`quilt push`會分別patch `01_fix`, `02_fix`。之後再`quilt pop`會變成只有apply `01_fix`而已。

---
<a name="rc"></a>
## ~/.quiltrc
使用者透過`~/.quiltrc`自訂quilt的行為，列舉如下

* QUILT_PATCHES：指定產生的patch存放的目錄，預設為patches目錄
* QUILT_PATCH_OPTS：讓quilt呼叫patch執行檔apply patch時傳遞的參數
* QUILT_REFRESH_ARGS和QUILT_DIFF_ARGS：讓quilt呼叫diff執行檔時傳遞的參數
* QUILT_COLORS：不用解釋吧

---
<a name="ref"></a>
## 參考資料

* quilt手冊： /usr/share/doc/quilt/quilt.pdf.gz
* [How to use quilt to manage patches in Debian packages (推)](http://raphaelhertzog.com/2012/08/08/how-to-use-quilt-to-manage-patches-in-debian-packages/)
* [UsingQuilt](https://wiki.debian.org/UsingQuilt)
* [Wikipedia: Quilt](http://en.wikipedia.org/wiki/Quilt_%28software%29)
* [OpenWrt Wiki: Working with patches](http://wiki.openwrt.org/doc/devel/patches)  
