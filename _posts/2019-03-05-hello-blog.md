# HELLO BLOG
---
layout: post
title: "HELLO BLOG"
subtitle: "Hello 2019"
author: "ChenJF"
header-img: "img\post-bg-imgs\1-hello-fight.png"
header-mask: 0.4
tag: 
  -生活
  -小笔记
---

> "Keep on"

##前言
换个地方写博客，新的环境总会带来许多小兴奋！  

这个博客使用的是Github的服务器，利用GitHub Pages + Jekyll就可以快速搭建，很简易方便，并且搭建完访问速度也很快。

页面主题使用的是黄玄大神的模板（[模板的Github地址在这里][1]），基于Jekyll的Clean Blog主题，多了标签等一些功能（我基本照搬 :D）。

还有about和portfolio等等一些页面没有完善，以后再看。

这篇博客也仅仅说明一下博客搭建使用的技术，测试一下..

##正文
正文随便弄一些..

###常用Git命令
####本地版本控制
* 查看工作区和暂存区状态：`git status`
* 查看修改内容：`git diff`（当work tree not clean时）
* 删除文件：手动删除后，`git rm [文件名]`。再commit（和add对应）
* 查看提交历史及相应版本号：`git log`
* 回退版本：`git reset --hard  HEAD^`（HEAD表示指向当前版本的指针，加一个^就对应指向上一个版本）
* 回退或撤销回退：`git reset --hard xxxx`（xxxx为版本号，前几位即可，会自动匹配。若回退后不知道版本号，使用git reflog查看）

####远程仓库控制

* 查看关联远程仓库：`git remote`
*  添加关联远程仓库：`git remote add [远程主机名（平时常写的origin）][远程仓库地址]`

* 推送到远程仓库：`git push [远程主机名] [[本地分支名]:[远程分支名]]`
（省略本地分支名和冒号就是选择当前分支作为要push上去的本地分支，如：git push origin master）
（省略本地分支名就是删除远程分支，因为相当于push了空分支，如：git push origin :branch1）
（省略[本地分支名]:[远程分支名]则视为push同名分支，远程没有会创建）
（加上参数-u就相当于选择默认远程主机和分支，下次可以直接git push或pull）

####分支管理
* 查看分支：`git branch`
* 创建分支：`git branch <name>`
* 切换分支：`git checkout <name>`
* 创建+切换分支：`git checkout -b <name>`
* 合并某分支到当前分支：`git merge <name>`
* 删除分支：`git branch -d <name>`
* ...

还有很多，我一般看这里学习，特别好用：[廖雪峰的Git教程][2]

##后记
后记有话说：没有后记

  [1]: https://github.com/Huxpro/huxpro.github.io
  [2]: https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000