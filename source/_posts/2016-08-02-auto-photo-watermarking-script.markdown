---
layout: post
title: "Auto photo watermarking script"
date: 2016-08-02 16:53:13 +0800
comments: true
categories: [Shell Script] 
---
因故需要對大量圖片調整解析度並加入浮水印，查了網路找到非常好的[範例文章](http://www.linuxjournal.com/content/easy-watermarking-imagemagick)。這篇[文章](http://www.linuxjournal.com/content/easy-watermarking-imagemagick)裏面也有script，和我的不太一樣，有興趣的人可以去參考。

更改後寫成script如下，它做了：

1. 開了`processed_pictures`目錄，把改過的檔案存到該目錄
2. 將新的圖案解析度寬度轉成1024
3. 將產生的圖片quality設成80%
4. 加入浮水印字串

```bash
#!/bin/sh
TARGET_DIR=processed_pictures
NEW_WIDTH=1024
CAP="Wen Liao"

mkdir -p $TARGET_DIR
for i in $(ls) ; do
    echo convert $i to $TARGET_DIR
    convert -background '#0008' -quality 80 -resize $NEW_WIDTH -fill white -gravity center -size $(identify -format %w $i)x120 caption:"$CAP" $i +swap -gravity south -composite $TARGET_DIR/$i
done
```

要注意

1. 本script假設目前目錄都是圖檔，所以不判斷檔名是否為`.jpg`或是`.png`
2. 無法處理檔名有空白的檔案
3. 你需要自行安裝imagemagick
4. 由於相片landscape 和 portrait的關係，浮水印的位址不一定會和你想的一樣，目前為止我不在意這個問題就是了。附上範例轉出的landscape 和 portrait照片各一張




#### Landscape

<img src="/files/dog.jpg" title="landscape">

#### Portrait
<img src="/files/cat.jpg" title="portrait">

## 參考資料

* [Linux Journal Easy Watermarking with ImageMagick](http://www.linuxjournal.com/content/easy-watermarking-imagemagick)
* [How to automatically watermark or batch watermark photos using ImageMagick](http://www.xoogu.com/2013/how-to-automatically-watermark-or-batch-watermark-photos-using-imagemagick/)
