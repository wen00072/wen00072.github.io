---
layout: post
title: 'Linux下面快速搜尋PDF的方式'
date: 2013-10-17 07:04
comments: true
categories: [Linux Utilities]
---
有時候手上一堆PDF檔，不知道要查的資料在那個檔案，可以用下面的指令找出

- `find -name "*.pdf" -exec pdfgrep {} \;`
- 注意pdfgrep需要額外安裝
    - `sudo apt-get install pdfgrep`
