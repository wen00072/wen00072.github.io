---
layout: post
title: 'GStreamer Application Development Manual 導讀 第一章到第十章 投影片上線'
date: 2014-05-16 21:26
comments: true
categories: [gstreamer]
---
<iframe src="http://www.slideshare.net/slideshow/embed_code/34371927" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px 1px 0; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px">  </div> 

跑手冊Hello world範例程式如果是自行編譯Gstreamer的環境的話，要注意

* 最少要再編譯Gstreamer-plugin-base
* 編譯plugin時，執行`autogen.sh`或是`configure`要看一下log，找出哪些沒有被編譯的plugin還有錯誤訊息，否則Hello workd會編得過但是因為沒有相對的plugin造成無法執行。
