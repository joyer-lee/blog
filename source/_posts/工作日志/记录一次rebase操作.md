---
title: 记录一次rebase操作
tags: []
id: '152'
categories:
  - - git
date: 2021-04-28 10:20:14
---



## git 合并方法介绍

### 方法一：git merge

把两个父节点及其提交记录包含起来生成一个新的节点

### 方法二：git rebase

复制被rebase的提交记录到当前分支

## 记录一次rebase操作

> 我要进行合并啦，记录下问题

### 步骤一：合并之前分支更新到最新

dev分支 ![](../images/uploads/sites/2/2021/04/QQ图片20210423152110.png) add-account-view分支 ![](../images/uploads/sites/2/2021/04/QQ图片20210423152236-1.png)

### 步骤二：login rebase dev

![](../images/uploads/sites/2/2021/04/QQ图片20210423152606.png) **遇到冲突**

### 步骤三：解决冲突后，git add，然后 rebase --continue

![](../images/uploads/sites/2/2021/04/QQ图片20210423152920.png) 若依然存在冲突，则**重复步骤三**，冲突解决完毕后，结果如下： ![](../images/uploads/sites/2/2021/04/QQ图片20210423153056.png)

> _错误提示_ 当前add-account-view的状态为，需要拉取39个提交，以及推送67个提交， 按理来说我们正确的合并了dev分支，并解决了相应冲突。 那么问题为什么会出现呢？以及怎么解决这个问题呢？

#### 若根据git状态拉取，则需继续解决冲突，此时冲突跟之前遇到的一样

![](../images/uploads/sites/2/2021/04/I1VLL_8Q5YAQM@44EO.png)

#### 继续解决重复冲突，完毕后如下

![](../images/uploads/sites/2/2021/04/image-20210423154134815.png)

#### 回到dev分支，继续rebase

依然存在同样冲突： ![](../images/uploads/sites/2/2021/04/I1VLL_8Q5YAQM@44EO-2.png) _继续解决冲突，完毕，依然需要pull，重新解决冲突，此时问题跟在add-account-view分支上问题一模一样。_

> 通过在网上查找资料，参考别人操作： 得知，正确步骤为： ![](../images/uploads/sites/2/2021/04/ICXI9IHJVCLZE6OA0SG8.png)

### 步骤四 直接执行 git push --force

![](../images/uploads/sites/2/2021/04/QQ图片20210423154746.png)

### 步骤五 成功rebase

### 原理分析

![](../images/uploads/sites/2/2021/04/QQ图片20210423155042.png)

### 个人理解

1.  rebase之后，相对远程分支而言，本地基底发生了变化，引发了需要pull一下的提示
2.  如果确保没问题是可以强制 push的

#### 举例

相当于变基之后本地提交是： C1，C2'，C3',C4,C5;（C2‘ C3’为目标合并分支的提交记录） 远程提交时： C1，C4，C5

#### 分析

**本地提交的前三个提交跟远程的前三个提交不一致，就会提示拉取代码，此时拉取代码会把C4，C5再拉取一下** 如果拉取就相当于跟我新合并的代码又冲突，还得解决一次，而且**git pull默认是merge**合并 **git pull之后，就相当于又merge合并了一次：** 把提交记录变成了： C1，C4，C5，C2',C3' ,C4,C5,可能后边还有其他的合并提交 同名的后一个只是副本这种，还会造成其他的合并提交 ![](../images/uploads/sites/2/2021/04/QQ图片20210423160822.png) 先强推下account分支，然后dev再rebase account分支，不然如果同时有其他同事也在提交，那就会被覆盖掉。