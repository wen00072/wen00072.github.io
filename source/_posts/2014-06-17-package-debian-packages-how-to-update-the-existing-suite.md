---
layout: post
title: '[Debian套件打包] 如何更新現有套件並重新打包？'
date: 2014-06-17 15:09
comments: true
categories: Debian
---
Debian New Maintainers' Guide提供了一些更新打包過的套件的建議:
分為

* [修復錯誤的情況](#fix)
* [打包上游更新原始套件](#rel)

<a name="fix"></a>
---
## 修復錯誤的情況
### 需要更改原本套件中的程式碼

* 透過[dquilt](http://wen00072.github.io/blog/2014/06/10/joined-his-patch-package-for-debian-packages)整理成patch
	* dquilt new 描述修正的檔名.patch
  * dquilt add 要修正的程式碼檔名
  * 修改要修正的程式碼檔內部程式碼
  * dquilt refresh產生patch
  * dquilt header -e描述剛才的更改

### 需要更改原本套件中debian/patches的patch

* 透過[dquilt](http://wen00072.github.io/blog/2014/06/10/joined-his-patch-package-for-debian-packages)整理成patch
	* dquilt pop 要修正的patch檔名
  		* dqulit會幫你**un-patch**
  * 修改要修正的patch檔名內部程式碼
  * dquilt refresh產生patch
  * dquilt header -e描述剛才的更改
  * while dquilt push; do dquilt refresh; done
  		* Apply 之前被pop的patch

---
更改完畢並產生完patch後，請更改debian/changelog說明更改項目。可以人肉修正、使用`dch -i`指令或是使用`dch -v 新的版號`協助更改。

如果更新的原因是因為套件管理系統中有bug report，在debian/changelog中需要特別提到`Close: 錯誤追蹤編號`，上傳的時候系統會自動根據該編號關閉相對issue。

更改完debian/changelog後請依照[之前](https://www.debian.org/doc/manuals/maint-guide/build.en.html#completebuild)的建議重新測試套件。測試無誤後，應將changelog中的狀態從UNRELEASED改為unstable或是experimental並且上傳。由於只有更改部份檔案，所以不需要上所有檔案。

---
<a name="rel"></a>
## 打包上游更新原始套件
首先要確認新版和舊版都是最乾淨的情況，也就是說autotools產生的檔案如configure等都不應該存在。接下來使用diff人肉檢查是否有可疑要注意的地方。指令如下：

* `diff -urN 乾淨舊版目錄 乾淨新版目錄`

接下來把舊版的debian目錄放置到新版的原始乾淨目錄中。debian目錄在打包的時候應該會單獨產生一個tarball，檔名為**套件名稱.debian.tar.gz**。做以下的更動

* 照[以前](http://wen00072.github.io/blog/2014/05/28/package-deb-packages-loose-style)格式將上游的tarball以`套件名稱.org.tar.gz`存檔。
* 使用`dch -v 新版本號碼`更新debian/changelog
	* 說明更新上游原始套件，如果有fix bug report的話也要加入`Close: 錯誤追蹤編號`
  * while dquilt push; do dquilt refresh; done
    * Apply 之前被pop的patch
    * 基本上新版有沒有需要apply自己patch?個人覺得不一定。就算需要，行號等資訊會因為程式碼更動等原因而不一定能夠patch上去。當發生問題時手冊有下面建議
      * 如果這個patch上游原始套件已經fix了，請直接使用`dquilt delete`刪除該套件。
      * 使用`dquilt push -f`暴力patch，然後人肉檢查patch退貨的檔案。接下來觀察退貨檔案，自幹新的patch。
      * 重複 while dquilt push; do dquilt refresh; done直到都可以apply為止。
* **上面的動作可以使用`uupdate`幫你簡化部份操作。** 

如果debian下的watch有指定的話，可以使用`uscan`代替`update`，主要的差別是uscan會根據debian/watch去下載上游原始套件而不需要人肉下載。


## 跳過部份

* 從舊的debian格式更新成新的debian格式
* 更上游原始套件的文字檔到UTF-8


---
## 參考資料

* [Debian New Maintainers' Guide: Chapter 8. Updating the package](https://www.debian.org/doc/manuals/maint-guide/update.en.html)
