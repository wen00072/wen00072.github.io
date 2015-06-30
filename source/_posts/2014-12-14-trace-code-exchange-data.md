---
layout: post
title: 'Trace code 交流會資料整理'
date: 2014-12-14 20:37
comments: true
categories: presentation
---
## Wen Liao

* [C語言在Linux下組裝經驗分享]http://wen00072.github.io/blog/2013/11/29/c-language-assembler-under-linux-sharing)

現場只有live demo，vim的精華在這邊有整理投影片。
<iframe src="//www.slideshare.net/slideshow/embed_code/41960022" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/zzz00072/trace-41960022" title="Trace 程式碼之皮" target="_blank">Trace 程式碼之皮</a> </strong> from <strong><a href="//www.slideshare.net/zzz00072" target="_blank">Wen Liao</a></strong> </div>

## Landice

* 介紹使用逆向工程觀察Android 應用程式行為。
	* 使用[apktools](https://code.google.com/p/android-apktool/)解開APP。
  * 使用[smali](https://code.google.com/p/smali/)反組譯dex byte code。

## Villar

* ftrace使用方式
	* 要自己編支援ftrace的Linux kernel
  * 使用Qemu載入支援ftrace的Linux kernel
  * 在Guest OS上
		* mount debugfs
		* 操作ftrace，設定要觀看的行為
		* 告訴kernel開始trace，以及結束trace
		* 觀看結果

<iframe src="//www.slideshare.net/slideshow/embed_code/42682768" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/vh21/trace-kernel-code-tips" title="Trace kernel code tips" target="_blank">Trace kernel code tips</a> </strong> from <strong><a href="//www.slideshare.net/vh21" target="_blank">Viller Hsiao</a></strong> </div>


## Zack/StarNight

* [介紹Conque-GDB](https://github.com/vim-scripts/Conque-GDB)
* [介紹cgdb](http://tech.mozilla.com.tw/posts/3826/cgdb-%E6%9B%B4%E5%A5%BD%E7%94%A8%E7%9A%84-gdb)

<iframe src="//www.slideshare.net/slideshow/embed_code/42683739" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/chienhungpan/debug-c-c-programs-more-comfortably-20141214-trace-code-meetup" title="Debug c c++ programs more comfortably @ 2014.12.14 trace code meetup" target="_blank">Debug c c++ programs more comfortably @ 2014.12.14 trace code meetup</a> </strong> from <strong><a href="//www.slideshare.net/chienhungpan" target="_blank">Chien-Hung Pan</a></strong> </div>
