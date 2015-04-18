---
layout: post
title: "Add categories list plugin to Octopress"
date: 2015-03-31 01:54:10 +0800
comments: true
categories: Octopress 
---
安裝方式如下

* 下載[Plugin](https://github.com/ctdk/octopress-category-list)
  * 因為檔案和Octopress 的路徑重疊，懶的處理可能的git問題，所以我直接拉zip。
* 修改`_config.yml`，把`custom/asides/category_cloud.html, custom/asides/category_list.html, custom/asides/top_category_list.html`加入到`default_asides:`這段裏面。
  * 最後頁面右邊會出現相對應的Category資訊
* 建立上方連結
  * `rake newpage['categories']`
  * 修改`soource/categories/index.md`，範例[在這邊](https://github.com/wen00072/wen00072.github.io/blob/source/source/categories/index.md)
  * 更改`source/_includes/custom/navigation.html`
    * 加入`<li><a href="{{ root_url }}/categories">Categories</a></li>`
  * `rake generate`
    * 不確定這個是否可以省略
  * `rake preview`
    * 看看有沒有錯誤
  * `rake deploy`
  * 記得把更改和新增的檔案commit並push到origin source

## 參考資料

* [Octopress Top Categories Plugin](http://time.to.pullthepl.ug/blog/2012/8/20/octopress-top-categories-plugin/)
