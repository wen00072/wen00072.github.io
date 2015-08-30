---
layout: post
title: "Ubuntu 14.04下使用Doxygen 產生pdf手冊"
date: 2015-08-25 23:13:44 +0800
comments: true
categories: Doxygen
---
一個pdf程式SDK手冊，你會期待有

* 封面
* 版權宣告、版本變動紀錄
* 目錄
* SDK簡介，包含方塊圖
* API使用方式
* 範例

Doxygen是一個open source的文件產生工具，預設是產生HTML。不過它也可以產生很正規的文件，快速紀錄一下，細節還是要自行補完。要達到上面的目的另外前題是你要會估狗查詢使用LaTeX的用法。

1. 安裝doxywizard
2. 安裝doxygen-latex
3. 執行`doxywizard`，透過GUI設定
    * 文件輸出目錄
    * 原始碼目錄
    * 開啟latex輸出
    * (需要塞範例程式) export -> Input -> EXAMPLE_PATH
4. 細項設定完成後，存檔。
5. 到輸出文件的latex目錄中，開啟refman.tex。剪貼
    * `%--- Begin generated contents ---`之前的文字，另存成header.tex
    * `%--- End generated contents ---`之後的文字，另存成end.tex
6. 重新執行`doxywizard 你剛才存的Doxyfile`
    * expert -> LaTeX -> 打開
        * LATEX_HEADER -> 選你剛才存的header.tex
        * LATEX_FOOTER -> 選你剛才存的end.tex
7. 存檔
8. 修改header.tex，可以加入
    * 封面
    * 版權宣告
    * 你產生API文件前的任何東西如簡介，方塊圖，貓的照片等
    * 自訂的header/footer
9. (需要塞範例程式)
    * 在程式碼註解塞入`\example 範例程式檔名`
    * 將`範例程式檔名`存到前面EXAMPLE_PATH的路徑中
10. `doxygen 你剛才存的Doxyfile`
11. 到輸出文件的latex目錄中
    * make，目錄由程式自動產生
12. 開啟refman.pdf，檢查輸出是不是你要的，不是的話回到第八步

