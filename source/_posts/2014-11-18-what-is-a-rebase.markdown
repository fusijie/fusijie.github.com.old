---
layout: post
title: "(译)第三部分：什么是rebase?"
date: 2014-11-18 23:44
comments: true
categories: git
tags: [rebase]
---

###内容提要

* 第一部分：[commit hash是什么？](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)
* 第二部分：[merge是什么？](http://www.fusijie.com/blog/2014/10/15/what-is-a-merge/)
* 第三部分：rebase是什么？

在[第一部分](http://www.fusijie.com/blog/2014/10/08/what-is-a-commit-hash/)中，我们讨论了什么是commit hash，其中一个很重要的特点就是commit无法被修改。hash值是根据存储在commit中的信息生成的，所以修改一个commit或者commit hash，你必须要创建一个全新的commit。我们还讨论了每一个commit存储了它的前一个commit的hash值。我们所没有讨论的是它对我们Git历史的影响。

<!-- more -->

实际上，commit hash是基于他们本身存储的信息生成的，而这些信息其中就包含了前一个commit的hash值，所以想修改你的commit历史基本上是不可能的。每一个commit就像是链条上的一环，紧紧连接着上一环。

![](http://i.imgur.com/cXvBMnk.png)

如果你有如上图一样的一条金属链，在不打断他们的前提下是不可能把前一环和后一环连接起来的。然而，在Git环境中这将会更糟。这样的类比在这里是不靠谱的，因为在一条金属链上你可以焊接一个新环来把前一环和后一环重新连接起来。但是在Git中，你无法做到这一点。

如果你想要在commit历史的中间删除某一个commit，那后一个commit将会指向一个不存在的commit hash。因为你无法在不改变hash的情况下来修改commit，所以你不能简单地生成一个新的commit来引用前一个commit，而后一个commit仍然引用了最原始的commit hash。

如果你改变了一个commit的某个属性，那生成的hash值将不再一样，后一个commit也不会引用到新的commit。结果就是你不得不去修改后一个commit来引用到新的commit hash，这同样会引起commit hash的改变，就这样一路下去直到链条的末尾。

这时候轮到[rebase](http://git-scm.com/docs/git-rebase)上场了。如果你还记得[第二部分](http://www.fusijie.com/blog/2014/10/15/what-is-a-merge/)，当我们将`feature1`分支合并到`master`分支后，有一副图展示了各个commit之间的关系。

![](http://i.imgur.com/S0av3NM.png)

Merge可以很好地工作，但是伴随着所有的fork和横纵交叉的commit关系，Git仓库的图形很快就会失控。下图只是一个我平时工作的Git仓库的小片段。

![](http://i.imgur.com/z28Y4sX.png)

如果你使用一个Git GUI软件，很有可能你也见识过类似的东西。Merge是在不同分支之间移动差异的最简单的方式，因为它避免了破坏commit历史和所引发的蛋疼。然而，一旦你对rebase的工作方式有了比较深刻的理解，你将会从中收益。举个栗子，如果我们在demo仓库中rebase `feature1`分支到`master`分支(译者注：这句话的意思是切换到`feature1`分支，执行`git rebase master`命令)，将会得到一个非常漂亮干净的历史，如图：

![](http://i.imgur.com/pBvTytu.png)

注意到没？现在的历史是一条直线了。Git到底是怎么做到的呢？如果你还记得的话，我们的`Commit 3`和`Commit 4`是共享`Commit 2`作为其共同父节点的，`Commit 3`引用了`Commit 2`作为其前一个commit。现在你也许会疑惑为什么看起来`Commit 3`像是将`Commit 4`作为其前一个commit。

还记得我刚说过的，如果想从中间打断链条，你必须从这个点上开始重现创建其之后的commit，直到结尾。没错，这实际上就是rebase做的事情。

![](http://i.imgur.com/1nPXWq1.png)

仔细看的话，你会发现`Commit 3`，`Commit 5`，`Commit 6`的commit hash已经全部改变了。这3个commit是在`feature 1`分支上提交的。通过将`feature 1`分支rebase操作到`master`分支上，从`master`分支分叉出来的的第一个commit开始，git重写了`feature 1`所有的commit，直到结束。它将分之上的每一个commit之间的差异存储在一个临时文件中，然后开始重写我们的分支历史。而这一次，分支是从`master`，`Commit 4`开始的。

Git给分支上的每一个commit创建了一个新的commit，当然跟着修改的还有commit hash值。当它创建新的commit的时候，第一个commit被改为引用到`master`分支的最新的commit（`Commit 4`），而不是原来的（`Commit 2`）了。这个重新提交你的变更作为新的commit的流程被称为“你的commits在`master`分支上的重播”。

>注意：不要让术语混淆。Rebase到`master`分支不会修改`master`分支本身，它的意思是你的分支commits将会紧跟着`master`分支上最新的commits（译者注：这里的`你的分支`指的是`feature 1`）。

![](http://i.imgur.com/pBvTytu.png)

你会注意到上图中`master`分支仍然指向`Commit 4`，它的commit hash值是没有改变的。如果我们现在切换到`master`分支，然后把`feature 1`分支合并到`master`分支，这将不会产生一个合并commit。这仅仅是一个快进提交，意思就是git将会简单地将指向`master`分支的指针笔直地移到指向`feature 1`分支的指针位置上。

![](http://i.imgur.com/rLdDgw3.png)

如果不把`feature 1`分支合并到`master`分支，我们还有更多的事要做，更多的commit要提交，我们可能会再fork一个仓库。我们的下一个`master`分支的commit将会指向`Commit 4`作为它的父节点，而`feature 1`分支的第一个commit也是指向`Commit 4`作为它的父节点。为了得到一条笔直的提交历史，我们需要再切换到`feature1`分支，然后再次rebase到`master`分支。这种情况很常出现，比如你在github上提交了一个pull request然后它过期了。如果项目的维护者没有合并你 的pull request，而是在这个项目上继续做一些其他工作，那么你的pull request就需要再来一次rebase操作以获取一个干净的git历史。把你做的工作rebase到原仓库分支上才可以让这个pull request能够在合并进去的时候采用简单的快进方式。接受一个pull request只是一个简单的合并。如果在提交pull request之前就rebase了你做的工作，那么这个merge就是一个快进方式的merge，这也能保证原仓库的干净。（译者注：这里的`干净`指的是没有额外的合并信息。）

####危险！！！

这部分内容是对rebase的使用进行一些警告，主要还是在多人协作上需要注意。因为rebase是一种改写commit的操作，所以相对比较危险，作者给出的意见是：

>Undoing a rebase is not easy, and often impossible so you really need to pay attention to what you're doing. The benefits of rebasing are great, but not if you don't know what you're doing.

撤销一个rebase操作不简单，而且经常是不可能的。你必须很注意自己在干什么。rebase让人受益，当时如果你不知道你在干什么的话，别用reabse。

这部分内容不翻译了，有兴趣自己看原文吧。


>英文地址:[http://codetunnel.com/merge-vs-rebase-part-3-what-is-a-rebase/](http://codetunnel.com/merge-vs-rebase-part-3-what-is-a-rebase/)
