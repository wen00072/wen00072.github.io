---
layout: post
title: '打包Debian 套件時加入自己的Patch方式'
date: 2014-06-10 03:24
comments: true
categories: Debian
---
在打包Debian套件，有時候會需要從更改原本套件，也就是說需要在打包時自行apply patch。而Debian套件可以透過[quilt](http://wen00072.github.io/blog/2014/06/08/study-on-the-quilt)協助管理patch。[Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)建議方式如下。

* 確認有安裝quilt
* 修改`~/.bashrc`加入：
```bash ~/.bashrc
alias dquilt="quilt --quiltrc=${HOME}/.quiltrc-dpkg"
complete -F _quilt_completion $_quilt_complete_opt dquilt
```
* 新增`~/.quiltrc-dpkg`
```text ~/.quiltrc-dpkg
d=. ; while [ ! -d $d/debian -a $(readlink -e $d) != / ]; do d=$d/..; done

if [ -d $d/debian ] && [ -z $QUILT_PATCHES ]; then
	## if in Debian packaging tree with unset $QUILT_PATCHES
    QUILT_PATCHES="debian/patches"
    QUILT_PATCH_OPTS="--reject-format=unified"
    QUILT_DIFF_ARGS="-p ab --no-timestamps --no-index --color=auto"
    QUILT_REFRESH_ARGS="-p ab --no-timestamps --no-index"
    QUILT_COLORS="diff_hdr=1;32:diff_add=1;34:diff_rem=1;31:diff_hunk=1;33:diff_ctx=35:diff_cctx=33"
    if ! [ -d $d/debian/patches ]; then mkdir $d/debian/patches; fi
fi
```
* 打包Debian產生patch時使用`dquilt`而不是quilt

---
## `~/.quiltrc-dpkg`說明
不要被這堆符號嚇到（先承認我一開使有被嚇到）。這邊可以看到為什麼要把quilt打包成quilt:

* `QUILT_PATCHES`：產生的patch會放在**debian/patches**目錄而不是預設的patches目錄
* `QUILT_PATCH_OPTS`：讓quilt呼叫patch執行檔apply patch時被退貨要使用unified格式
* `QUILT_REFRESH_ARGS`和`QUILT_DIFF_ARGS`：讓quilt呼叫diff執行檔指定
	* `-p ab`：使用`a/file b/file`的diff格式而不是`dir.orig/file dir/file`的表示方法
  * 產生patch檔案內不包含timestamp及index的資訊
* `QUILT_COLORS`：不用解釋吧



---
## 參考資料

* [Debian New Maintainers' Guide](https://www.debian.org/doc/manuals/maint-guide/)
