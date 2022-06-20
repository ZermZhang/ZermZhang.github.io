---
title: git-rebase-merge说明
tags: ['github','命令','rebase','merge']
date: 2022-06-20 16:15
categories: ['git']
---
> git rebase和git merge是在git使用中常见的命令之二，而且这两个命令的功能较为相似，都是将不同的分支进行分支合并。
> 但是在日常使用中，git merge的使用频率要远高于git rebase，究其原因是git rebase的复杂性导致的。
> 这里简单的介绍一下我个人对git merge和git rebase两者之间的区别和使用过程中需要注意的相关事项。

* 注意，本文中的git状态大多是从下图的基础状态出发：
![git-base-status.png](https://s2.loli.net/2022/06/20/qpvS7xYAdVlbGiL.png)

# 1. 命令介绍
```git
git-merge - Join two or more development histories together
git-rebase - Reapply commits on top of another base tip
```
上述，是git文档中对于merge和rebase两个命令的介绍。


# 2. git merge
显然，当需要处理两个或者是多个分支合并的时候，git merge是一个很自然的选项，但是通过merge处理的分支合并过程中，会增加一个不必要的history join commit.
![git-merge.png](https://s2.loli.net/2022/06/20/r8Gn314UOCbkBgy.png)
如上图，git merge之后，会在现有c1、c2、c3的基础上，新增一个c4节点，该节点是在c2和c3基础上merge之后的内容。

# 3. git rebase
为了防止出现这个无意义的commit而引起不必要的conflict，rebase这时候就成为了一个不错的选项。
rebase的中文名叫做“变基”，（一个很符合github用途的名字:smile:），它的具体作用也和名字很一致：改变当前分支checkout的位置。
如上git-base-status图片中展示的样子，dev是在c1节点处从main分支上checkout出来，通过git rebase可以调整当前分支的checkout位置，从而实现类似merge的功能。

## 3.1 git rebase (on main)
```git
git checkout main
git rebase dev
git merge dev
git branch -d dev
```
1. 先checkout到main分支上
2. 在main分支上对dev进行rebase，表示将main分支checkout的位置从c1调整到dev的节点，即c3上。
此时，main分支的路径变为：c1<-c3<-c2'
3. 可以将dev再merge到main上，但是此时的merge不会影响到main分支的内容
![git-rebase-on-main.png](https://s2.loli.net/2022/06/20/spSLCrK7bfEIRw3.png)

## 3.2 git rebase (on dev)
```git
git rebase main
git checkout main
git merge dev
git branch -d dev
```
1. 直接在dev分支上对main进行rabse，表示将dev分支checkout的位置从c1调整到main的节点，即c2上。
此时，dev分支的路径变为：c1<-c2<-c3'
2. 切换回main分支，再将dev分支merge到main分支上
![git-rebase-on-dev.png](https://s2.loli.net/2022/06/20/GJS9n5sWBPTqhOm.png)

# 4. 备注
1. 因为rebase的作用会改变git的history，所以在使用过程中必须要注意，rebase涉及到的history不能存储在远程，否则会引起本地和远程的conflict
2. git rebase也可以通过参数`-i`来处理过多的commit信息，防止commit log太过杂乱