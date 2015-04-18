---
layout: post
title: "Modify footer in Octopress"
date: 2015-03-30 00:23:43 +0800
comments: true
categories: Octopress 
---
步驟如下：

* 更改`source/_includes/custom/footer.html`
* `rake generate`
  * 不確定這個是否可以省略
* `rake preview`
  * 看看有沒有錯誤
* `rake deploy`
* 記得把更改的檔案commit並push到orgin source

目前的頁面最後已經加上了CC-BY-NC說明，就是將`source/_includes/custom/footer.html`內加上相關HTML tag的。

## 參考資料

* [Octopress: Theming & Customization](http://octopress.org/docs/theme/template/)



