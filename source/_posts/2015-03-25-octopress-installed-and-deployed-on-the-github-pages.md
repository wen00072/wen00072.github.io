---
layout: post
title: '安裝octopress並佈署到github pages上面'
date: 2015-03-25 20:40
comments: true
categories: [Octopress, Github pages] 
---
昨天評估使用github.io存放網頁的技術，本來是想直接套用Jekyll的東西，無奈能力不足，Markdown的支援度不如預期。後來想到Octopress應該支援度不差。估狗了一下的確可行，順手紀錄一下安裝方式。

先說結論，Octopress + Github pages的好處有

* 使用git管理你的content
  * 換句話說，你所更動的一切都在Github上面
* Octopress幫你搞定
  * deploy到github page
  * local PC preview，這樣可以改到爽再commit你的content
  * 支援Markdown，簡化寫網頁的effort
  * Open source，喜歡就自己改吧
  * code block有行數，歐耶。


資料有點雜，還是弄一下目錄好了

* [測試環境](#oct-env)
* [預先準備](#oct-prepare)
* [安裝Octopress](#oct-install)
* [設定網頁資訊](#oct-config)
* [撰寫及發表文章](#oct-deploy)
* [Octopress注意事項](#oct-notice)
* [參考資料](#oct-ref)


<a name="oct-env"/></a>
## 測試環境

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty
```


<a name="oct-prepare"/></a>
## 預先準備
#### github 帳號，以及[開通git pages](https://pages.github.com/)
#### 安裝Octopress需要的軟體

```
$ sudo apt-get install ruby1.9.1-dev
$ sudo apt-get install nodejs
$ sudo apt-get install rbenv
$ sudo gem install bundler
$ rbenv rehash
```


<a name="oct-install"/></a>
## 安裝Octopress
基本上就照手冊來，口頭來說就是

* [下載Octopress](##oct-dl)
* [安裝Octopress](##oct-ins)
* [指定你要deploy到Github](##oct-gh)

<a name="oct-dl"/></a>
#### 下載Octopress
官方網址如下

* [Octopress Github](https://github.com/imathis/octopress)

```
$ git clone https://github.com/imathis/octopress
$ cd octopress
$ bundle install
```

`bundle install`會根據Octopress裏面的Gemfile安裝使用Octopress需要的Ruby套件。

<a name="oct-ins"/></a>
#### 安裝Octopress

```
$ rake install
```

<a name="oct-gh"/></a>
#### 指定你要deploy到Github page
跑下面的指令會要求你輸入Github Page的repository如
`git@github.com:wen00072/wen00072.github.io.git`

```
$ rake setup_github_pages
```

`setup_github_pages`很有趣，值得提一下他幫你做了什麼

* 跟你討Github page的URL
* 把你下載Octopress的repository remote名稱從`origin`改成`octopress`
* 把你的Github page的repository設成目前git remote的`origin`
* local新增`source` branch，從master切到該branch
* 設定Octopress要deploy到你的Github pages
* clone你的Github page的repository 到`_deploy`目錄

使用下面的指令，產生資料並上傳到你的Github page，此時直接連Github Page應該就會有網頁畫面了。

```
$ rake generate
$ rake deploy
```

再來我們直接把目前Octopress的Git repository和前面指令產生/修改新的檔案一口氣產生並加到你的Github page中的source branch吧。這個點特別有趣感覺像是做了某種移花接木的技巧，可以仔細想想他為什麼這樣做。想通了應該對於git remote會有點感覺...吧？

提示：你應該還記得前面`rake setup_github_pages`有提到Octopress會將你的Github page repository URL設成目前working directory git remote的orgin吧？

```
$ git add .
$ git commit -m "git add . after rake deploy"
$ git push origin source
```

所以現在你的Github page repository origin會有兩個branch

* master
	* 網頁連過去看到的內容
  * 由rake deploy管控
* source: 從Octopress拉下來的程式碼和你剛才指令新增的檔案
	* 你的content以及config由此更動


<a name="oct-config"/></a>
## 設定網頁資訊
[官方網頁](http://octopress.org/docs/configuring/)有提到如何更改設定。我只做最簡單的更動，就是`_config.yml`

```
diff --git a/_config.yml b/_config.yml
index 4bf56f1..e7de252 100644
 a/_config.yml
+++ b/_config.yml
@@ -3,9 +3,9 @@
 ## -- ##
 
 url: http://wen00072.github.io
-title: My Octopress Blog
-subtitle: A blogging framework for hackers.
-author: Your Name
+title: 國王的耳朵是驢耳朵
+subtitle: 國王的耳朵是驢耳朵
+author: Wen Liao
 simple_search: https://www.google.com/search
 description:
```

如果要讓你的config馬上生效，除了local commit， push到source branch外，不要忘記套用到的Github page。

```
$ rake generate
$ rake deploy
```


<a name="oct-deploy"/></a>
## 撰寫及發表文章
#### 撰寫文章
撰寫文章有兩種，一個是請Octopress幫你生一個範本，指令如下

```
$ rake new_post["你的標題"]
```

然後Octopress會幫你產生到`source/_posts/`下面
來個範例

```
$ rake new_post["First"]
mkdir -p source/_posts
Creating new post: source/_posts/2015-03-25-first.markdown

$ cat source/_posts/2015-03-25-first.markdown

layout: post
title: "First"
date: 2015-03-25 22:10:45 +0800
comments: true
categories: 

```
可以看到產生的檔案會有特定的規範。所以你也可以照樣造句，自己人肉寫一個，這就是第二種方式。不管怎樣，剩下的就是按照Markdown語法編輯檔案了。

#### 發表文章
最開始有講過，使用Octopress的好處是可以改到爽再deploy到Github page，要如何做呢？很簡單

```
$ rake preview
```
接下來就是用瀏覽器連`localhost:4000`，如果不滿意就停止，開始改你的markdown檔案，再`rake preview`。改到爽為止，決定發佈就用下面的指令上傳到Github page上面。

```
$ rake deploy
```

上傳完畢後不要忘記local的markdown檔案並沒有commit並且更新。所以你可以做

```
$ git add source/_posts/
$ git commit -m "Hello world"
$ git push origin source
```

要注意`git add source/_posts/`第一次可以這樣做，接下來建議還是針對你的markdown檔案一個一個確認比較妥當。

這邊是我的[示範網頁](http://wen00072.github.io/)。


<a name="oct-notice"/></a>
## 注意事項
嘗試Octopress發現和以前使用的Markdown有一些要注意的地方

* 第一個item (我用*) 前面一定要空一行，不然系統無法辨識
* HTML 的anchor tag (我拿來做目錄跳躍)一定要`<a name="ref"></a>`，以前的`<a name="ref"/>`會讓Octopress的排版亂到讓你想殺人。
* 強烈建議修改`sass/custom/_layout.scss`，把`//$indented-lists`改成`indented-lists`，不然list的bullet對齊會很難看。
* 如果直接`rake preview`發現網頁沒更新，請用`rake generate`檢查是否你的markdown有錯誤的語法。
* preview的時候你的markdown更新到你可以看到網頁改變時間有點長，我這邊大概要十幾秒到二十幾秒的時間才會更動。
* 不要一次丟太多markdown進去，因為目前`rake generate`會噴錯誤，但是不會跟你說在parse哪個markdown檔案出了問題。

<a name="oct-ref"/></a>
## 參考資料
* [李嘉玲的技術筆記: Github Page + Octopress](http://zerodie.github.io/blog/2012/01/19/octopress-github-pages/)
* [Octopress: Octopress Setup](http://octopress.org/docs/setup/)
* [Octopress: Deploying to Github Pages](http://octopress.org/docs/deploying/github/)
* [Octopress: Configuring Octopress](http://octopress.org/docs/configuring/)
* [Octopress: Blogging Basics](http://octopress.org/docs/blogging/)
* [Markdown list does not indent using octopress](http://stackoverflow.com/questions/24794024/markdown-list-does-not-indent-using-octopress)
* [Some Octopress Rake Tips](http://robdodson.me/some-octopress-rake-tips/)
	* 沒試過，小心服用
