---
layout: post
title: "再談 git remote"
date: 2016-01-24 20:30:32 +0800
comments: true
categories: 
---

* 更新：
    * Apr/09/2016：感謝網友柏瑀的告知，修正白字。

git remote 是一個常常被忽略的東西，這次就來看看這個指令和對應的觀念吧。

一樣，先講測試環境，避免無法reproduce的問題。

```text
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty

$ git --version
git version 2.7.0
```

本次介紹目錄如下

* [背景說明](#gr_gb)
* [git remote指令介紹](#gr_cmd)
    * [查詢remote資訊](#gr_cmd_status)
    * [新增remote](#gr_cmd_add)
    * [拉remote repository](#gr_cmd_fetch)
    * [remote的branch操作](#gr_cmd_push)
    * [刪除remote](#gr_cmd_rm)

<a name="gr_gb"></a>
## 背景說明
先來問問男人`git remote`是什麼吧？

```
GIT-REMOTE(1)                                     Git Manual                                    GIT-REMOTE(1)

NAME
       git-remote - Manage set of tracked repositories

```

白話來說，`git remote`是用來管理<font color="red">多個</font>tracked repositories。（中文難翻，意思是你要追蹤的git repository）。在大部分的情況，`git clone`下來後只需要和原本clone的遠端互動的話，你只有一個remote，那麼你機乎不會感受到remote的運作，我天生駑鈍，剛開始只是覺得奇怪怎麼有時候後會有個`origin`跑出來而已。

如果夠幸運（還是不幸？）你不太會需要遇到需要處理remote的情況。不過在github上面，倒是蠻有機會要用到remote，情境如下。

* 你在github上面有帳號，要fork別人的project `A`，fork來的稱為`A+`
* 你在`A+`開發，同時`A`也在開發
* `A+`有需要和`A`同步，也許是`A`有新功能，或是你要回饋程式碼總不能回饋和`A`差異太大的程式碼

`A+`和`A`同步，是remote常用的情境。常用到github直接給[懶人包](https://help.github.com/articles/configuring-a-remote-for-a-fork/)。大概描述如下：

* `A+`加一個remote，URL為`A`的URL。白話來說，就是`A+`要track repository `A`。
    * remote必須給個名字，github的[懶人包](https://help.github.com/articles/configuring-a-remote-for-a-fork/)上給的名字叫upstream。
* 既然加了一個tracked repository，你就可以做
    * 拉remote程式碼下來
    * 切換到remote下面的branch
    * **merge remote 的branch**

<a name="gr_cmd"></a>
## git remote指令介紹
在介紹指令之前，先建立示範git repository 以便下面的說明。情境如下

1. 建立新的git repository，稱為`repo_1`
2. clone `repo_1`到`repo_2`
3. clone `repo_2`到`repo_3`

```
$ git init repo_1
Initialized empty Git repository in /tmp/test_remote/repo_1/.git/

$ cd repo_1/
/repo_1$ touch test_file && git add test_file && git commit -a -m "init for test"
[master (root-commit) 6847c3c] init for test
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test_file

$ cd ../

$ git clone --bare repo_1 repo_2 
Cloning into bare repository 'repo_2'...
done.

$ git clone repo_2/ repo_3
Cloning into 'repo_3'...
done.
```
關於repo_2的--bare部份，一言難盡。長話短說，不這樣幹repo_3不能push到repo_2，有興趣可以看[這邊](http://stackoverflow.com/questions/2816369/git-push-error-remote-rejected-master-master-branch-is-currently-checked)。

如果你的git repository是clone過來的，一定會有一個預設remote。沒意外的會叫`origin` <del>(什麼？你的remote叫`aosp`? 呃太複雜不討論)</del>，接下來你可以對於remote 操作，分別說明如下：

* [查詢remote資訊](#gr_cmd_status)
* [新增remote](#gr_cmd_add)
* [拉remote repository](#gr_cmd_fetch)
* [remote的branch操作](#gr_cmd_push)
* [刪除remote](#gr_cmd_rm)

<a name="gr_cmd_status"></a>
## 查詢remote資訊

* `git remote`
    * 顯示remote名稱
* `git remote -v`
    * 顯示remote名稱及remote URL
* `git remote show remote名稱`
    * 顯示`remote名稱`的詳細資訊

直接看範例

```
$ cd /tmp/test_remote/repo_3

$ git remote
origin

$ git remote -v
origin	/tmp/test_remote/repo_2/ (fetch)
origin	/tmp/test_remote/repo_2/ (push)

$ git remote show origin 
* remote origin
  Fetch URL: /tmp/test_remote/repo_2/
  Push  URL: /tmp/test_remote/repo_2/
  HEAD branch: master
  Remote branch:
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

<a name="gr_cmd_add"></a>
## 新增remote
* `git add remote名稱 remote_repository_URL`

範例如下

```
$ git remote add upstream /tmp/test_remote/repo_1/

$ git remote -v
origin	/tmp/test_remote/repo_2/ (fetch)
origin	/tmp/test_remote/repo_2/ (push)
upstream	/tmp/test_remote/repo_1/ (fetch)
upstream	/tmp/test_remote/repo_1/ (push)
```

新增完畢後，請記得`remote_名稱`、`remote_名稱 remote_branch_名稱`或`remote_名稱/remote_branch_名稱`是你要操作remote的參數。

另外你還可以加入多個remote來場大亂鬥。例如你從`A` fork `A+`，也許有人fork 出來的`A++`有你想要的功能，就可以再把`A++`加入你`A` repository的remote。

<a name="gr_cmd_fetch"></a>
## 拉remote repository
* `git fetch remote_名稱`
* `git pull remote_名稱/remote_branch_名稱`

我們先在`repo_1`開一個branch稱為`br1`
```
$ cd /tmp/test_remote/repo_1/

$ git checkout -b br1
Switched to a new branch 'br1'

$ touch test_on_branch && git add test_on_branch && git commit test_on_branch -m "test"
[br1 cc7222f] test
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test_on_branch
```

接下來就是範例時間
```
$ git fetch upstream 
From /tmp/test_remote/repo_1
 * [new branch]      br1        -> upstream/br1
 * [new branch]      master     -> upstream/master

$ ls
test_file

$ git pull upstream br1
From /tmp/test_remote/repo_1
 * branch            br1        -> FETCH_HEAD
Updating 6847c3c..cc7222f
Fast-forward
 test_on_branch | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test_on_branch

$ ls
test_file  test_on_branch

$ git status 
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean
```
注意最後的指令，因為我們pull了`upstream/br1`，所以local多了一個commit。因此訊息中有說你目前repository 比`origin/master`(你clone的remote/branch)多一個commit。你可以push回`origin/master`。

<a name="gr_cmd_push"></a>
## remote的branch操作
原則就是命令中有參數會是`remote_名稱`、`remote_名稱 remote_branch_名稱`或`remote_名稱/remote_branch_名稱`。


* 切換
```
$ git checkout upstream/br1 
Note: checking out 'upstream/br1'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at bd026e3... test
```
中間的一堆英文簡單來說是因為git checkout沒特別參數，git只是把HEAD指到命令中的hash而已。如果你要同時在local開一個branch，可以這樣幹：

```
$ git checkout --track  upstream/br1 
Branch br1 set up to track remote branch br1 from upstream.
Switched to a new branch 'br1'
```

* merge

```
# 先在upstream/br1建立新的commit
$ cd /tmp/test_remote/repo_1/

$ touch test1_on_branch && git add test1_on_branch && git commit test1_on_branch -m "test1"
[br1 d82612a] test1
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test1_on_branch

# 切回repo_3
cd /tmp/test_remote/repo_3

# 先fetch
$ git fetch upstream 
remote: Counting objects: 2, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 2 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (2/2), done.
From /tmp/test_remote/repo_1
   bd026e3..d82612a  br1        -> upstream/br1

# 再merge，事實上和git pull upstream br1 一樣效果（默）
$ git merge upstream/br1 
Updating bd026e3..d82612a
Fast-forward
 test1_on_branch | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test1_on_branch
```

<a name="gr_cmd_rm"></a>
## 刪除remote
* git remote remove remote名稱

範例如下

```
$ git remote remove upstream 

$ git remote -v
origin	/tmp/test_remote/repo_2 (fetch)
origin	/tmp/test_remote/repo_2 (push)
```


