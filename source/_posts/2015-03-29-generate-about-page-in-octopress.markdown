---
layout: post
title: "Octopress 產生 About頁面"
date: 2015-03-29 23:13:07 +0800
comments: true
categories: [Octopress]
---

順序如下

* 切到Octopress根目錄
* `rake newpage['about']`
* Octopress會幫你產生`source/about/index.md`
* 開始修改`source/about/index.md`
* 修改`source/_includes/custom/navigation.html`
  * [範例](https://github.com/wen00072/wen00072.github.io/commit/9a0edf3ceadbeecfd26059523943d52d840654d4)
* `rake generate`
  * 不確定這個是否可以省略
* `rake preview`
  * 看看有沒有錯誤
* `rake deploy`
* 記得把更改和新增的檔案commit並push到origin source

## 參考資料

* [Add 'About' Page in Octopress](http://gangmax.me/blog/2012/05/04/add-about-page-in-octopress/)
  * 這邊資料無法直接套用，不過當初估狗到這個至少讓我知道大概的方向，該給的Credit還是要給。
  * 2015/Mar/31更新，後來發現直接`rake newpage['頁面名稱']`，Octopress就會幫你產生`source/頁面名稱/index.md`以及相關的metadata。
