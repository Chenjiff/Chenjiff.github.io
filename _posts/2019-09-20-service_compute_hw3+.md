---
layout: post  
title: "Go开发环境安装"  
subtitle: "服务计算"  
author: "ChenJF"  
header-img: "img/post-bg-imgs/2-systems-analysis-design.jpg"  
header-mask: 0.4  
catalog: true
tags:  服务计算
---

* TOC
{:toc}
---



### Go开发环境安装

#### 1. vscode安装

centos下的命令：

```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

更新即可：

```
yum check-update
sudo yum install code
```

#### 2. golang安装

先安装epel，直接进行下一步提示找不到可用包

```
yum install epel-release
```

安装golang

```
sudo yum install golang
```

#### 3. 配置环境变量：

创建工作空间文件夹：

```
 mkdir $HOME/gowork
```

我的系统是centos7，在 `~/.bash_profile` 文件中添加:

```
export GOPATH=$HOME/gowork
export PATH=$PATH:$GOPATH/bin
```

执行这些配置:

```
$ source $HOME/.profile
```

检查配置:

```
$ go env
...
GOPATH = ...
...
GOROOT = ...
...
```

#### 4. 安装工具和插件

下载源代码到本地：

```
# 创建文件夹
mkdir $GOPATH/src/golang.org/x/
# 下载源码
go get -d github.com/golang/tools
# copy 
cp $GOPATH/src/github.com/golang/tools $GOPATH/src/golang.org/x/ -rf
```

安装工具包：

```
$ go install golang.org/x/tools/go/buildutil
```

之后退出 vscode，再进入，按提示安装！

#### 5. 安装gotour

执行以下：

```
go get github.com/Go-zh/tour/gotour
gotour
```
