---
layout: post
title: "(译)第一部分：什么是commit hash?"
date: 2014-10-08 22:25
comments: true
categories: git
tags: [commit hash]
---

####内容提要

* 第一部分：commit hash是什么？
* 第二部分：[merge是什么？]()
* 第三部分：[rebase是什么？]()

最近一段时间我在学习如何使用[Git](http://git-scm.com/)，碰到的一个难点：如何区别[merge](http://git-scm.com/docs/git-merge)和[rebase](http://git-scm.com/docs/git-rebase)？大部分人都能理解merge的概念，但是对于rebase就不是很清楚了。在这三篇博文中我将尽可能用最简单的方式来解释它们的异同。不过在此之前，我们需要先了解一下什么是commit hash。

<!-- more -->

如果你看过自己的commit历史，那么对于下面的内容肯定不会陌生：

```
	commit a9ca2c9f4e1e0061075aa47cbb97201a43b0f66f 
	Author: Alex Ford 
	Date: Mon Sep 8 6:49:17 2014

	Initial commit.
```

你也许会认为这个由字母和数字组成的长长的字符串是一个单独commit的唯一的ID。虽然你是对的，但是你可能不知道它是一个[SHA-1](http://en.wikipedia.org/wiki/SHA-1)生成的哈希码，用于表示一个git commit对象。如果不去深入理解git [commit object](http://git-scm.com/book/en/Git-Internals-Git-Objects#Commit-Objects)，那顶多就只知道这是一个基于它所表示的信息直接生成的一个很大的加密字符串。改变一个commit hash的唯一方式就是改变commit的细节，本质上来说，其实是生成了一个全新的commit对象。

除了一些明显的信息，比如commit的作者，时间，存储的数据，commit通常还包含了在它之前的一个commit的hash，这正是你的commit历史产生的原因。每一个commit都知道紧跟它之前的commit hash。

![](http://i.imgur.com/mljhFlh.png)

上图可以看到我的[SourceTree](https://www.atlassian.com/software/sourcetree/overview)窗口，打开了一个我创建的demo仓库。我做了3次commit。SourceTree相当智能，它可以读取仓库中的每一个commit，然后用图形的方式展现出commit历史。可以看到，`Commit 2`直接引用了`Commit 1`，而`Commit 1`直接引用了`Commit 0`。需要注意的是，在这里，我直接使用Commit字样作为commit描述，为的是尽可能简单地谈论这个话题。实际上，每一个commit信息都应该正确地描述它们所做的改变。

因为我的demo仓库master分支上只包含了这3个commit，所以SourceTree的图形从头到尾就是一条简单直线。好，现在我们就做点稍微复杂一点的事情，为了一个新的功能，我们需要创建一个独立的分支。

![](http://i.imgur.com/S5o9qWL.png)

从上图可以看到我创建了一个叫`feature1`的分支，但是图形仍然是一样的。这是因为在创建完新的分支后，我并没有进行任何新的commit。分支实际上只是指向一个特殊commit的指针。现在，`master`和`feature1`都指向了同一个commit。好了，我们往`feature1`分支添加一个新的commit

![](http://i.imgur.com/qjIWl7F.png)

可以看到，我们的`feature1`分支移动了它的指针来指向一个新的commit，`Commit 3`。你可以看到，我们的图形*仍然*是一条简单的直线。这是因为到目前为止，仅有4个commit，而每一个commit都是引用了紧跟它的前一个commit。如果我现在将`feature1`合并到`master`，只会发生一件事，就是`master`分支会直接跳到和`feature1`指向的相同commit，也就是`Commit 3`。这个叫做*fast-forward*合并（快进合并），因为它只是简单地将`master`分支的移动到指向最新的commit。

OK，当我们兴高采烈地在`feature1`上赶工，突然老板一个电话说一个新的Bug需要被马上解决，这是重中之重。这需要暂停`feature1`上的工作，然后马上在`master`分支上修复bug并提交。这个时候，我们不得不切换到`master`分支，然后进行一个commit。如果bug很大，可能得考虑是否要创建另一个分支，然后在这个分支上进行多个commit，现在假装bug很小，只要一个commit就能搞定。

![](http://i.imgur.com/8MFZLBi.png)

好了，现在看起来就有点不一样了，请注意上图的图形，`feature1`分支上的`Commit 3`在自己封闭的路径上了。原因很简单，`Commit 4`和`Commit 3`有相同的祖先。还记得commit是如何存储紧跟它之前的commit吗？当切换到`master`分支的时候，我们将会返回到`Commit 2`，因为`Commit 3`只由`feature1`分支指针引用。而`master`分支指针仍然指向`Commit 2`。因为我们的修复commit（`Commit 4`）将`Commit 2`视为它的前一个commit。

现在的图形告诉我们，`Commit 4`和`Commit 3`同时都引用了`Commit 2`作为他的前一个commit。在这种情况下，可以将`Commit 2`视为`Commit 3`和`Commit 4`共有的祖先。现在我们的修复已经被提交了，所以可以回到`feature1`分支继续工作了。

![](http://i.imgur.com/kxDIgKl.png)

现在我在`feature1`分支上创建了2个新的commit，`Commit 5`和`Commit 6`。新功能已经完成了，也是时候合并`feature1`分支到`master`分支中了。这时，我们可以选择merge `feature1`分支到`master`分支，也可以reabse `feature1`分支到`master`分支，让我们在[第二部分]()中探究一下merge是什么？


>原文地址:[http://codetunnel.com/merge-vs-rebase-part-1-what-is-a-commit-hash/](http://codetunnel.com/merge-vs-rebase-part-1-what-is-a-commit-hash/)






