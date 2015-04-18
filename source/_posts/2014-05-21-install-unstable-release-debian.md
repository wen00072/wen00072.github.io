---
layout: post
title: '安裝unstable release Debian方式'
date: 2014-05-21 11:20
comments: true
categories: 
---
想安裝Unstable release Debian，跑去官方網站看，發現ISO檔只有stable和testing兩種。網路搜尋後發現原來unstable沒有ISO檔[*](#ref1)，後來決定從已經安裝的Debian系統切換過去。

測試環境：Debian “wheezy” 7.5

如果您的系統是stable release的話，第一步要把release從stable切換到testing。切換方式為

* 把`/etc/apt/sources.list`中的`wheezy`更換成`jessie`
* `sudo apt-get update`
* `sudo apt-get dist-upgrade`
* `sudo reboot`

如果您已經是testing或是從stable切換到testing的話，也是做類似的事情。

* 把`/etc/apt/sources.list`中的下面描述
```
deb http://ftp.debian.org/debian jessie main
deb-src http://ftp.debian.org/debian jessie main
```
換成
```
deb http://ftp.debian.org/debian sid main
deb-src http://ftp.debian.org/debian sid main
```

**請注意URL不一定會相同，但是`jessie main`以及`sid main`會相同**
至於為何不全部換呢？請看[這邊](#src-list)解釋。

然後執行
* `sudo apt-get update`
* `sudo apt-get dist-upgrade`
* `sudo reboot`
* 重開後從終端機執行`lsb_release -a`結果應為
```
Distributor ID:	Debian
Description:	Debian GNU/Linux unstable (sid)
Release:	unstable
Codename:	sid
```

<a name="src-list"></a>
## sid 的sources.list分析
`/etc/apt/sources.list`存放APT套件管理系統存取遠端套件資訊。裏面的格式為

* `type url 目錄名稱 元件` 
	* 範例`deb http://ftp.debian.org/debian jessie main contrib non-free`

根據[這邊](https://wiki.debian.org/RepositoryFormat#A.22Release.22_files)的描述，遠端資料是放在主機URL的`dists`下面([範例](http://ftp.debian.org/debian/dists/))。我們瀏覽該URL下面的的dists目錄，就會發現下面的目錄和`目錄名稱`相同。

回到主題：為何從testing換成unstable只有改一個repository？原因是其他的repository並沒有unstable版本。確認如下：

* 在`URL/dists`中([範例](http://ftp.debian.org/debian/dists/))，你會發現`sid`只有一個目錄。不像`jessie`或是`wheezy`會有幾個用這個Code name開頭的目錄。
* Security套件團隊沒有處理unstable release ([出處](https://www.debian.org/releases/sid/)) 


## 參考資料
<a name="ref1"></a>

* [The unstable distribution ("sid")](https://www.debian.org/releases/sid/)
* [DebianUnstable - Debian Wiki](https://wiki.debian.org/DebianUnstable)
* [The Debian Administrator's Handbook: Chapter 6. Maintenance and Updates: The APT Tools](http://debian-handbook.info/browse/wheezy/apt.html)
* [Debian 無痛起步法 ( 線上最新版 )](http://people.debian.org.tw/~moto/debian/DebianLessPain/Debian-Install-Guide.html)
  * 2002年
