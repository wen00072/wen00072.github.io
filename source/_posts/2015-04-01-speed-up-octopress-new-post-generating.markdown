---
layout: post
title: "加快Octopress 修改/新增文章發佈前 preview速度"
date: 2015-04-01 00:12:13 +0800
comments: true
categories: Octopress
---
搬了一百多篇文章後，很明顯地發現Octopress產生頁面速度慢到爆炸，有多慢呢？大概下了`rake preview`指令後要幾十秒才能更新。我猜測`rake preview`實作上可能是把`source/_posts`全部都抓起來重新產生頁面造成的問題。

目前能夠找到的方法只能算是workaround，勉強可以號溝。整理如下

1. (optional) `rake newpost['新的title']`
2. `rake isolate['你要修改的檔名']`
    * 其實這個命令單純產生`source/_stash`目錄，然後把你`rake isolate`以外的檔案搬到這邊。換句話說，目前`source/_post`下面只有你`isolate`起來的那一篇。
3.  因為2，`rake preview`速度飛快，你就快樂的修改你的文章，一存檔馬上就可以看到變化。
4. 修改完畢後，要用`rake integrate`剛才搬出去的文章搬回來。
5. 悲劇的地方在這邊，你還是得`rake generate`把所有的文章從markdown轉成HTML，時間一樣漫長。
6. `rake deploy`發佈到Github
7. 記得把更改和新增的檔案commit並push到origin source

總之，就是號溝。如果有更快的工具，支援Markdown和Github page，我會很慎重地考慮再搬一次。

## 參考資料
* [Speed up Octopress generation](https://leafduo.com/articles/2013/04/15/speed-up-octopress-generation/)
