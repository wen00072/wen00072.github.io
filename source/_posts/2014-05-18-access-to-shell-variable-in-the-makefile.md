---
layout: post
title: 'Makefile中Command敘述的Shell變數存取'
date: 2014-05-18 07:18
comments: true
categories: [makefile]
---
想說寫一個loop自動make變數內的目錄
```makefile 錯誤的Makefile
DIRS = libs src

all:
	for i in ${DIRS}; do make -C $i; done
```
結果跑出來和想像不同
```text make結果
for i in lib src; do make -C ; done
make: option requires an argument -- 'C'
```

原因是$本身在Makefile有特殊意義。要使用$$才能在command敘述中顯示$，因此改進如下

```makefile 正確的Makefile
DIRS = libs src

all:
	for i in ${DIRS}; do make -C $$i; done
```
## 參考資料

* [Stackoverflow: How do I use shell variables in Makefile actions?](http://stackoverflow.com/questions/13774784/how-do-i-use-shell-variables-in-makefile-actions)
