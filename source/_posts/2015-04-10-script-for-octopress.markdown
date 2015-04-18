---
layout: post
title: "加快Octopress 編輯文章的script"
date: 2015-04-10 08:30:41 +0800
comments: true
categories: [Octopress]
---

[之前](http://wen00072.github.io/blog/2015/04/01/speed-up-octopress-new-post-generating)有提到Octopress在文章夠多的時候產生網頁很慢，有號溝的方法。每次要用的時候都要打一狗票。**偷懶是組裝工的美德**，所以寫了一個script，喜歡的人自己拿去用吧。

這個script主要是

* 例行檢查
* isolate、preview、generate文件

```sh
#!/bin/bash
# Check parameter
if [ "$#" = "0" ] ; then
    echo "usage: $(basename $0) [octopress markdown flie]"
    exit
fi

# Check file
if [ ! -f "$1" ] ; then
    echo "No such file: $1"
    exit
fi

# Let's create and process markdown
rake isolate[$1]
rake generate
rake preview
rake integrate
rake generate
```
