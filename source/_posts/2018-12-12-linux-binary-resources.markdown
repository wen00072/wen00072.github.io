---
layout: post
title: "Linux binary 縮寫和連結"
date: 2018-12-12 21:48:22 +0800
comments: true
categories: [binutil, ELF]
---
整理目前看過的資料縮寫以及網路連結，可能會更新。

## ELF header

* `hdr`: header
* `Ehdr`: ELF header
* `EI`: ELF ident
* `ET`: ELF type
* `EM`: ELF machine

## Program header

* `Phdr`: Program header
* `PT`: Program header type
* `PF`: Prgram header flag

## Section header

* `Shdr`: Section header
* `SHN`: Section header index
* `sh_`: Section header
* `SHT`: Section header type
* `SHF`: Section header flag

## Symbol table

* `st`: Symbol table
* `STT`: Symbol table type
* `STV`: Symbol table visibility

## Dynamic section

* `DT`: Dynamic section type

## Note section

* `NT`: Note type

## Auxiliary vector

* `AT`: auxiliary vector type

## 資源

* [man 5 elf](http://man7.org/linux/man-pages/man5/elf.5.html)
* [man ld](http://man7.org/linux/man-pages/man1/ld.1.html)
* [man readelf](http://man7.org/linux/man-pages/man1/readelf.1.html)
* [man objdump](http://man7.org/linux/man-pages/man1/objdump.1.html)
* [man ld.so](http://man7.org/linux/man-pages/man8/ld.so.8.html)

