---
layout: post
title: '筆記: [OpenStack] GIT Commit Good Practice'
date: 2013-09-08 12:00
comments: true
categories: [git, Software Development]
---
[出處：[OpenStack] GIT Commit Good Practice](https://wiki.openstack.org/wiki/GitCommitMessages)

- 大部份的程式碼是被閱讀的次數遠大於更動的次數，因此可讀性很重要。

## 如何結構化地切割變動？

- 原則：每次的commit應該只包含一個邏輯上的變動

### commit時該避免的事

- commit的部份把空白字元更動(如空白從8個換成4個、TAB換成空白字元)和程式變動混在一起
- 把和commit宣稱要改新功能無關的程式碼一起commit進去
- 一次commit 大量的程式碼

## 關於Commit訊息

- 不要假設審查者會瞭解該commit原本要處理的問題，所以不要忘記留下關於你要修掉的問題的描述和資訊，省去審查者自行查找的時間
- 由於git的特性，審查者不一定只在電腦可以連上網路時審查您的commit。因此請將在commit留下足夠的訊息而不是要求別人去連到某個外部連結去參考資料
- 不要自以為程式碼明顯易懂就在commit訊息偷懶省略資料。提供問題描述、解決方式和更動會比較週全
- 請描述更動程式碼的理由和目的
- 如果您的commit訊息需要長篇大論的說明時，也許是個好時機去分拆commit成更細小的commit
- 如果很不幸地必須要commit大量的程式碼，請務必在commit 訊息中提供資訊讓審查者便於審查這大堆頭的程式碼
- commit訊息的第1行非常重要。可以用在email的標題、log viewer的提示等。請想辦法在一行的長度讓閱讀log的人瞭解你commit了什麼資訊
- 如果程式碼有已知錯誤、Todos、或是limitation也記得一併寫在commit訊息中

## OpenStack 使用的commit 訊息格式

- 第1行簡略描述該commit更動的內容，字數小於50字元，結尾不得有句點
- 第2行單純為跳行符號
- 第3行以後開始詳細描述更動的範圍，長度不得超過72字元
    - 描述問題狀況
    - 對應問題的更動描述，以及更動的理由說明
    - 提供更動後的結果範例
    - Limitation，改善空間等
- 軟體管理用的辨別ID如Change-id等
