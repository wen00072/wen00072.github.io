---
layout: post
title: '談談git remote'
date: 2015-03-06 07:15
comments: true
categories: [git]
---
在軟體專案開發常常會需要比對不同的repository。舉例來說，你可能在遠端有一套軟體專案，這個專案是從upstream fork 下來，那麼如果要把upstream 新的功能合併到專案，人肉合併往往是最容易出錯又最沒效率的方式。如果這兩個專案都有git，那麼git remote就是你的救星。針對剛才講的更詳細的use case可以看[Git Hub的範例](https://help.github.com/articles/configuring-a-remote-for-a-fork/)

直接拿Use case範例，不囉唆。
Demo情境

* 有兩個遠端repositories: repo1, repo2
* 將repo1 clone下來到local repository，另外一個透過git remote加入到local repository
* 在local建立一個branch，對應到repo2
* merge repo1到repo2
	* 要注意的是其實以upstream 情況是剛好相反，一般來說repo1，也就是你clone的才是你要開發的專案。選擇這樣的case單純只是吃飽撐著。

詳細行為範例如下

* [建立測試環境](#g2_env)
* [新增remote URL](#g2_rt_url)
* [建立對應remote URL的local branch](#g2_rt_branch)
* [在local合併兩個Romote Branch](#g2_rt_merge)
* [參考資料](#g2_ref)


<a name="g2_env"></a>
## 建立測試環境
產生兩個遠端repositories: repo1, repo2

## repo1
```
$ mkdir remote_repo1
$ cd remote_repo1/
$ git init .
Initialized empty Git repository in /tmp/remote_repo1/.git/
$ echo "repo1" > myfile
$ git add myfile 
$ git commit myfile -m "init"
[master (root-commit) 043d3c6] init
 1 file changed, 1 insertion(+)
 create mode 100644 myfile
```

## repo2
```
$ cd ../
$ mkdir remote_repo2
$ cd remote_repo2/
$ git init .
Initialized empty Git repository in /tmp/remote_repo2/.git/
$ echo "repo2" > myfile
$ git add myfile 
$ git commit myfile -m "init"
[master (root-commit) 5c329bf] init
 1 file changed, 1 insertion(+)
 create mode 100644 myfile
```

## 看一下目錄架構
```
$ cd ../
$ tree remote_repo1 remote_repo2/
remote_repo1
└── myfile
remote_repo2/
└── myfile
```


<a name="g2_rt_url"></a>
## 新增remote URL
就 git clone，沒啥好說

```
$ git clone remote_repo1/ local_repo
Cloning into 'local_repo'...
done.
$ cd local_repo/
$ git remote -v
origin	/tmp/remote_repo1/ (fetch)
origin	/tmp/remote_repo1/ (push)
```


<a name="g2_rt_branch"></a>
## 建立對應remote URL的local branch
我們用了`git remote add 本地如何稱呼remote remote的URL`

```
$ git remote add repo2 /tmp/remote_repo2/
$ git remote -v
origin	/tmp/remote_repo1/ (fetch)
origin	/tmp/remote_repo1/ (push)
repo2	/tmp/remote_repo2/ (fetch)
repo2	/tmp/remote_repo2/ (push)
```

接下來把remote URL的repository 拉下來
```
$ git fetch repo2
warning: no common commits
remote: Counting objects: 3, done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Unpacking objects: 100% (3/3), done.
From /tmp/remote_repo2
 * [new branch]      master     -> repo2/master
```

然後建立一個branch，對應到repo2的master
```
$ git checkout -b local_repo2 repo2/master
Branch local_repo2 set up to track remote branch master from repo2.
Switched to a new branch 'local_repo2'

$ git branch -a
* local_repo2
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/repo2/master

$ git diff master 
diff --git a/myfile b/myfile
index 464d9cd..27d38ae 100644
 a/myfile
+++ b/myfile
@@ -1 +1 @@
-repo1
+repo2
```


<a name="g2_rt_merge"></a>
## 在local合併兩個Romote Branch
剩下非常直覺，就把兩個local branch merge，整理完衝突收工。


```
$ git merge master 
Auto-merging myfile
CONFLICT (add/add): Merge conflict in myfile
Automatic merge failed; fix conflicts and then commit the result.

$ git status 
On branch local_repo2
Your branch is up-to-date with 'repo2/master'.
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both added:      myfile

no changes added to commit (use "git add" and/or "git commit -a")

$ cat myfile 
<<<<<<< HEAD
repo2
=======
repo1
>>>>>>> master
$ echo shut-up > myfile 

$ git add myfile 

$ git commit -m "Overwrite myfile"
[local_repo2 a76c0a6] Overwrite myfile
```

<font color="red"/font>請注意這個case不能夠把更動push 回repo2/master 會噴錯誤如下</font>

```
$ git push repo2 
warning: push.default is unset; its implicit value has changed in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the traditional behavior, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

When push.default is set to 'matching', git will push local branches
to the remote branches that already exist with the same name.

Since Git 2.0, Git defaults to the more conservative 'simple'
behavior, which only pushes the current branch to the corresponding
remote branch that 'git pull' uses to update the current branch.

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

fatal: The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

    git push repo2 HEAD:master

To push to the branch of the same name on the remote, use

    git push repo2 local_repo2

To choose either option permanently, see push.default in 'git help config'.
```

簡單翻譯一下，目前local branch名稱在repo2上面沒有，你要嘛

* 指定push到repo2的master: `git push repo2 HEAD:master`
* 要嘛指定push local branch到repo2 repository上

第一個情況需要bared repository，時間關係有空再談。


<a name="g2_ref"></a>
## 參考資料

* [ProGit: 2.5 Git Basics - Working with Remotes](http://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes)
* [ProGit: 3.5 Git Branching - Remote Branches](http://git-scm.com/book/en/v2/Git-Branching-Remote-Branches)
