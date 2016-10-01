---
layout: post
title: 'bash變數的字串替換'
date: 2014-06-30 13:53
comments: true
categories: [Linux, Shell Script]
---
有時候會在command line或是script裏面需要極簡單的字串替換。bash有提供這樣的功能。這樣的功能是用以下的方式達成

* 變數=字串
* ${變數名稱/想要找的字串/替換的字串}

直接看範例就知道
```text ${變數取代範例}
$ MY_VAR=test.tar.gz
$ echo ${MY_VAR/.tar.gz/}
test
$ echo ${MY_VAR/.tar.gz/_a}
test_a
```

我自己是偶爾會需要解壓縮tarball，這時候先建立一個目錄再解壓縮會比較妥當。不然可能會變成目前目錄下面散落很多解出來的檔案，這就不是我想要的狀態。因此我的script會這樣做。

```bash extract.sh (沒有檢查錯誤）
#!/bin/bash
FILE=$1                   # Get tarball from command line 
DIR_NAME=${FILE/.tar.gz/}
mkdir $DIR_NAME-pkg
cd $DIR_NAME-pkg
tar -zxvf ../$FILE
```
