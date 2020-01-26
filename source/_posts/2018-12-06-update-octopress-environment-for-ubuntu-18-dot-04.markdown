---
layout: post
title: "Ubuntu 16.04更新到Ubuntu 18.04後Octopress 環境變動"
date: 2018-12-06 22:24:01 +0800
comments: true
categories: [Octopress]
---

本來想要寫一些東西，結果發現`rake`在Ubuntu 無法執行，只好先處理了。

主要的問題是更新後`Ruby`版本從16.04使用的`2.3`升級成`2.5`了。以下是我紀錄過的測試指令，必須承認這是網路上的東西剪貼，我不想知道後面的原理，後果自行負責。寫這篇文章另一個目的是確定上傳到網路上後可以正常發佈才證明真的解決問題了。

## 預安裝套件
```
sudo apt install -y gcc libcurl4-openssl-dev libxml2-dev
```

## Ruby 相關更新，完全不知道做啥

```
sudo gem install bundler
bundle install
sudo gem install rake
```

## 更新Octopress Gemfile
由於更新後`rake`版本也從`10.5.0`變成`12.3.1`，所以一跑`rake`就會出現版本不合的錯誤，因此我把`Gemfile` `rake`的版本檢查改成`12`，diff 檔案如下

```diff
diff --git a/Gemfile b/Gemfile
index 153dd3d..9f5048b 100644
--- a/Gemfile
+++ b/Gemfile
@@ -1,7 +1,7 @@
 source "https://rubygems.org"
 
 group :development do
-  gem 'rake', '~> 10.0'
+  gem 'rake', '~> 12.0'
   gem 'jekyll', '~> 2.0'
   gem 'octopress-hooks', '~> 2.2'
   gem 'octopress-date-format', '~> 2.0'
```

## 參考資料

* [[Ubuntu 18.04] Error installing on step 'bundle install' (fix in comments)](https://github.com/cliffe/SecGen/issues/113)
* [Rake aborted! no such file to load --bundler/setup Rails 3.1](https://stackoverflow.com/questions/7483515/rake-aborted-no-such-file-to-load-bundler-setup-rails-3-1)
