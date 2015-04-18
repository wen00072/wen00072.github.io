---
layout: post
title: '如何trace bash script中的function'
date: 2015-01-22 22:35
comments: true
categories: 
---
在組裝的過程中，常常會遇到用source去載入一個shell script，裏面通常是一組的環境變數。然而如果專案大到某個程度，這個script就會塞入很多複雜的function，例如某專案的build/envsetup.sh。這時候要去trace用肉眼去看實在是太過殘酷，最近為了這個問題想到了一招很簡單的方式，那就是：

```
$ bash -x
```

沒錯就這麼簡單，直接來看例子吧。首先我們有一個script裏面有function。

```bash demo.sh
#!/bin/bash
function whatsoever()
{
    echo "My code works, I don't know why."
}
```

接下來就是剛才講的操作

```
$ bash -x
+ '[' -z '\s-\v\$ ' ']'
+ shopt -s checkwinsize
... # 中間發生超多事，有興趣自行研究。發現好玩的再跟我說。

$ cd /tmp
+ cd /tmp
$ . demo.sh
+ . demo.sh
$ whatsoever 
+ whatsoever
+ echo 'My code works, I don'\''t know why.'
My code works, I don't know why.
```

打完收工，謝謝收看。

## 更新

網友Scott Tasi大大說這個方式和下面的方式相同：
```
set -x
```
我就學天線寶寶再重複一下實驗

```
$ set -x
$ . demo.sh
+ . demo.sh
$ whatsoever 
+ whatsoever
+ echo 'My code works, I don'\''t know why.'
My code works, I don't know why.
$ set +x
+ set +x
$ whatsoever 
My code works, I don't know why.
```
結論就是，用`set -x`，`set +x`就可以了。不過`bash -x`跑出來的訊息還真的嚇了我一跳，原來載入一個bash有那麼東的東西要做。這就是所謂的微言大意嗎？
