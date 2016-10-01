---
layout: post
title: '[Debian套件打包] debian目錄初探 (2)'
date: 2014-06-11 04:32
comments: true
categories: [Debian]
---
在[[Debian套件打包] debian目錄初探 (1)](http://wen00072.github.io/blog/2014/06/10/package-debian-packages-study-on-the-debian-directory)我整理了debian目錄下面主要的幾個檔案。主要是參考[Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)的第五章。

dh_make產生的範本除了產生control, copyright, changelog以及rules這個四檔案以外，還會產生deb的工具(debhelper)使用的一些設定檔範本。這些設定檔範本一般來說副檔名為ex。有些設定檔範本的檔名prefix會是你的原始套件名稱。舉例來說，我們要產生的套件為testautotools，那debian下面可以看到

* testautotools.cron.d.ex  
* testautotools.default.ex  
* testautotools.doc-base.EX
這些檔案。

另外請注意dh_make沒有產生**所有**debhelper會用到的設定檔，如果真的需要這些檔案，使用者必須自己寫一個出來。

要使用這些設定檔，要做的是：

* 如果設定檔範本副檔名為ex或是EX，把檔名中.ex或是.EX刪除。
* 如果設定檔範本prefix是你的原始套件名稱，請確認該名稱和最後release的套件名稱一致。
* 更動設定範例檔。
* 幹掉沒在用的設定範例檔。
* 視情況更動control和rules檔。

## 設定檔範本說明

* `README.Debian`: 和原始套件無關但是和Debian套件有關的可以放這邊，沒用到就幹掉。
* `compat`: debhelper相容性，手冊建議設成9。
* `conffiles`: 處理升級套件時config file(如/etc下的設定檔)衝突的問題。有興趣可以參考[這邊](https://wiki.debian.org/DpkgConffileHandling)和[這邊](https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html#s-configdetails/)。
* `dirs`: 在Makefile內make install不會建立但是又需要的目錄。基本上這有點邏輯矛盾，通常應該是要回頭看Makefile。這個算是一個workaround吧？
* `docs`: 其他要工具安裝的文件如README或是自訂的文件檔案。
* `emacsen-*`: emacs相關檔案，跳過。
* `install`: 原本套件的build system沒有安裝的檔案，但是產生的deb套件需要加料安裝的就放在這邊。
* `{package.,source/}lintian-overrides`: 看不懂，跳過
* `menu`: 如果是GUI的應用程式，可以設定選單位置。詳細語法在[這邊](https://www.debian.org/doc/packaging-manuals/menu-policy/)
* `NEWS`、`TODO`: 工具會自動安裝該檔案。
* `postinst`, `preinst`, `postrm`, `prerm`: dpkg安裝、更新、移除套件時會呼叫的scripts。一般建議菜鳥不要亂動，很容易GG的。如果真的非要更動的話，請確認測試過您套件install,upgrade, remove, purge等操作。細節說明在[這邊](https://www.debian.org/doc/debian-policy/ch-maintainerscripts.html)。
* `watch`: uscan這個程式會根據watch的資訊看upstream是否有新的release。
* `source/format`: 顯示原始套件是原裝的(native)或是需要透過[quilt](http://wen00072.github.io/blog/2014/06/08/study-on-the-quilt)管理patch。詳細說明可以man dpkg-source
* `source/local-options`:提供選項讓協助管理產生patch，例如請工具忽略autotool產生的configure.sub檔案。
* ` patches/*`: 透過[quilt](http://wen00072.github.io/blog/2014/06/08/study-on-the-quilt)產生的patch放在這邊。
* `manpage.*`: 原始套件應該要付manpage，如果沒有dh_make會生一些範本 。
	* `manpage.1.ex`: 使用manpage標準語法，1的部份需要根據您的套件性質更動。如一般API就需要改成3。詳細可以man man找section查詢數字代表的section。
  * dh_make也提供其他格式的manpage如xml和SGML，有興趣的請自行查詢[手冊](https://www.debian.org/doc/manuals/maint-guide/dother.en.html)。
* `套件名稱.manpages`: 告訴工具要安裝原始套件中的哪些manpage。
* `套件名稱.symbols`: 不建議菜鳥把包函式庫，一般來說函式庫應該要包含這個檔案。
* `套件名稱.cron.*`: 套件定期需要執行的事。
* `套件名稱.examples`: 如果需要工具安裝範例請寫在這邊。
* `套件名稱.init`: 開機需要自動從/etc/init.d啟動的套件的啟動script。
* `套件名稱.default`: 從/etc/init.d啟動的套件的啟動script會參考的變數放在這邊。
* `套件名稱.info`: info 指令會顯示出來的文件說明
* `套件名稱.link`: 在套件build system要額外建立symbolic link可在這邊指定。不過目前想不通什麼時機會用到。
* `套件名稱.doc-base`: 描述套件文件要放在系統的位置。範例如下：

```text testautotools.doc.EX
Document: testautotools
Title: Debian testautotools Manual
Author: <insert document author here>
Abstract: This manual describes what testautotools is
 and how it can be used to
 manage online manuals on Debian systems.
Section: unknown

Format: debiandoc-sgml
Files: /usr/share/doc/testautotools/testautotools.sgml.gz

Format: postscript
Files: /usr/share/doc/testautotools/testautotools.ps.gz

Format: text
Files: /usr/share/doc/testautotools/testautotools.text.gz

Format: HTML
Index: /usr/share/doc/testautotools/html/index.html
Files: /usr/share/doc/testautotools/html/*.html
```

---
## 參考資料

* [Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)
