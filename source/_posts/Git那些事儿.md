title: Git那些事儿
date: 2016-03-20 12:21:18
tags:
categories:
- tools
---

>Git工具的使用和Git工具的思想是我们无论如何也要掌握的技能，网上非常多的博文讲述了如何使用它，我链接一份官方的中文文档以便查阅

[Git工具使用官方文档](https://git-scm.com/book/zh/v2/)

>参考其中部分谈谈


## 分布式版本控制的由来

版本控制的好处就不多说了，最开始版本控制系统是集中式版本控制系统(CVCS),例如CVC Subversion以及Perforce等 ，他们都有一个集中管理的服务器，如果服务器单点故障，项目的所有记录信息保存在单一位置，一旦丢失，后果相当严重。

<!-- more -->

所以，分布式版本控制出现了，它的精髓就是每个客户端，都会克隆整个项目(快照)，虽然带来的额外的时间和储存开销，但是每个客户端都会对服务器进行完整的备份，即使服务器挂掉了，通过客户端的镜像恢复还是很快。不仅如此，客户端之间还可以交互，这样一来，层次模型式工作，协同工作更是得到了实现。

用两张图来说明差异性
CVCS:

![](http://7xrw2w.com1.z0.glb.clouddn.com/Gitcentralized.png)


Git：

![](http://7xrw2w.com1.z0.glb.clouddn.com/Gitdistributed.png)

## Git 和 其他版本控制系统(svn perforce 等)的巨大差异

- 直接记录快照，而非差异比较

- 几乎所有操作都是本地执行，不用随时连接服务器

- Git保存完整性，通过hash值建立索引


用两张张图来说明差异性
CVCS 如何实现版本更新：

![](http://7xrw2w.com1.z0.glb.clouddn.com/Gitdeltas.png)


Git 如何记录快照

![](http://7xrw2w.com1.z0.glb.clouddn.com/Gitsnapshots.png)

### Git三种状态

  - 1.已提交 commited
    数据已经安全的保存到了 **本地数据库中** ，注意是本地数据库。

  - 2.已修改 modified
    只是简单的修改了，还没有保存到本地数据库中。

  - 3.已暂存 staged
    对一个已经修改文件的当前版本做了标记，使得它包含在下次提交的快照。



### Git三个工作区
  - 1.Git仓库 从其他计算机clone仓库的时候，就是拷贝这里的数据

  - 2.工作目录，从Git仓库的压缩数据库中提取出来放在磁盘上让我们修改使用。

  - 3.暂存区域，一个保存了下次将要提交的文件列表信息，一般放在Git 仓库目录中，有时候说索引。

### Git工作流程
  - 1.工作目录下修改文件。

  - 2.暂存文件，将文件的快照放入暂存区域。

  - 3.提交更新，找到暂存区域的文件，将快照永久性的存储到Git仓库目录。

  ![](http://7xrw2w.com1.z0.glb.clouddn.com/git_areas.png)



## 本地分支管理 master




## 远程分支管理 origin master



>未完待续...
