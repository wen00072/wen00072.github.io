---
layout: post
title: '雜記: What a C programmer should know about memory'
date: 2015-08-08 15:30
comments: true
categories: 
---
出處: [What a C programmer should know about memory](http://marek.vavrusa.com/c/memory/2015/02/20/memory/)

這是筆記，不是導讀。單純放我覺得特別的東西和感想。不會自我要求可讀性和文章架構，請自行斟酌。

## Understanding virtual memory - the plot thickens

* The virtual memory allocator (VMA) may give you a memory it doesn't have, all in a vain hope that you're not going to use it. Just like banks today. 幹原來銀行是虛擬記憶體

* malloc給valid pointer不要太高興，等你要開始用的時候搞不好OS給個OOM說人肉鹹鹹。簡單來說就是一張支票，能不能拿來開等到兌現才知道（煙）

## Understanding stack allocation

原來C99的variable length array的運作是因為stack frame的特性，反正你要多少stack在橋的時候順便加一加。malloa一樣的原則。又學到了

## When to bother with a custom allocator
* 今日英文(?): GP allocator, General purpose allocator，你每天用的malloc是也。什麼，沒用malloc過啊，還真是幸福（煙）。

### Slab allocator
* 有的時候程式會allocte並使用多個不連續的記憶體區塊，如樹狀的資料結構。這時候對於系統來說有幾個問題，一是fragment、二是因為不連續，無法使用cache增快效能。

作者posix_memalign()可解決這類的問題。目前看範例，看不懂，感覺上和malloc很大的記憶體空間，自己管理差不多。先跳過，有大大路過可以留言解惑一下。

## Demand paging explained

* Linux系統提供一系列的記憶體管理API
    * 分配，釋放
    * 記憶體管理API
	    * mlock，禁止被swapped out
    * madvise，提供管道告訴系統page管理方式如
  	    * MADV_RANDOM，期待記憶體page讀取行為是隨機的。
    
* laze loading：allocate記憶體先給位址。等到process要存取的時候OS就會發現存取到沒摸過的記憶體，於是就產生page fault，這時候才去處理page分配的問題。

* 每次page fault 就像kernel發動白金之星，然後就有一個白痴驚訝地發現自己樓梯怎麼走都走不完，但是他完全無法感受/了解發生了什麼事。

## Fun with flags memory mapping

* long page_size = sysconf(_SC_PAGESIZE);
    * 取得系統的page size。

## Fixed memory mappings
投降輸一半，不懂Fixed memory mappings是啥、以及什麼時候需要這樣的東西。

## File-backed memory maps
透過mmap，可以將檔案內容mmap到記憶體中，如此一來可以加快存取速度。配合參數，多個process可以共用同一塊記憶體。不過衍生出來有幾個問題

1. 如何寫回檔案
    * man msync
    * 其他方式：fsync/pwrite
2. 如何調整檔案/記憶體長度
    * man ftruncate 

### Copy-on-write
有些情況是一個process要吃別的process已經map到記憶體的內容，而不要把自己改過的資料放回原本的記憶體。也就是說最終會有兩塊記憶體(兩份資料)。當然每次都複製有點多餘，因此系統使用了Copy-on-write機制。要怎麼做呢？就是在mmap使用MAP_PRIVATE參數即可。

### mmap不是萬能丹，極端情況下連續page fault可能會比原本的開檔獨資料還慢。


---
# 延伸參考資料，我都沒看
* [What every programmer should know about memory](http://www.akkadia.org/drepper/cpumemory.pdf)
* [Memeory FAQ](http://landley.net/writing/memory-faq.txt)
* [An Investigation Into the Theoretical Foundations and Implementation of the Linux Virtual Memory Manager](https://www.kernel.org/doc/gorman/pdf/thesis.pdf)
* [Understanding the Linux Virtual Memory Manager](https://www.kernel.org/doc/gorman/pdf/understand.pdf) 
	* 爆炸長，講Linux的VM管理
* [Understanding Memory](http://www.ualberta.ca/CNS/RESEARCH/LinuxClusters/mem.html)
* [Anatomy of a Program in Memory](http://duartes.org/gustavo/blog/post/anatomy-of-a-program-in-memory/)
