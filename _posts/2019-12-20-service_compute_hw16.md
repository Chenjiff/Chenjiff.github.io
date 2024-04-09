---
layout: post  
title: "Docker实践"  
subtitle: "服务计算"  
author: "ChenJ"  
header-img: "img/post-bg-imgs/2-systems-analysis-design.jpg"  
header-mask: 0.4  
catalog: true
tags:  服务计算
---

* TOC
{:toc}
---

> 本篇博客是Docker实践的实验报告，也是深入学习Docker技术的实操部分。
>
> 流程参考：服务计算-容器化技术与容器服务课件

###  一、准备docker环境

系统：Centos7

安装具体步骤（参考[官网](https://docs.docker.com/install/linux/docker-ce/centos/)，我选择通过仓库安装）：

1. 安装配置仓库：

   ```
   $ sudo yum install -y yum-utils \
     device-mapper-persistent-data \
     lvm2
   $ yum-config-manager \
       --add-repo \
       https://download.docker.com/linux/centos/docker-ce.repo
   ```

2. 安装docker

   ```
   $ sudo yum install docker-ce docker-ce-cli containerd.io
   //以下版本号自选
   $ yum list docker-ce --showduplicates | sort -r
   ```

3. 启动即可使用

   ```
   $ sudo systemctl start docker
   ```

   查看版本：

   ![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/1.png)

### 二、运行第一个容器

* hello-world:

```
$ docker run hello-world
```

​	![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/2.png)

### 三、Docker基本操作

* bash:

```
$ docker run -it ubuntu bash
```

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/3.png)

* 查看镜像

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/4.png)

* 获得帮助

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/5.png)

* 显示运行中容器

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/6.png)

* 显示所有容器（包含已中止）

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/7.png)

* 继续运行原容器并进入

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/8.png)

### 四、MySQL与容器化

* 拉取 MySQL 镜像

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/9.png)

* 构建并运行docker镜像练习

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/10.png)

* 使用MySQL容器

服务器：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/11.png)

客户端：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/12.png)

数据库文件：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/13.png)

上述位置文件卷即为自动创建的数据卷：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/14.png)

* 创建卷并挂载

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/15.png)

* 客户端链接服务器

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/16.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/17.png)

* 挂载现有数据库

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/18.png)

* 修改容器配置

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/19.png)

* Docker compose 与多容器应用自动化部署

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/20.png)

### 五、Docker网络

* 管理容器网络

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/21.png)

* 备制支持 ifconfig 和 ping 命令的 ubuntu 容器

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/22.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/23.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/24.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/25.png)

* 启动另一个命令窗口，由容器制作镜像

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/26.png)

* 创建自定义网络

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/27.png)

* 在两个窗口创建u1,u2容器网络，并使用以下命令 

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/28.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/29.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/30.png)

### 六、Docker仓库

登陆、tag、上传、下载：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/31.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/32.png)

删除：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/33.png)

运行：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/34.png)

登出：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/35.png)

### 七、容器监控与与日志

* 检查docker的状态

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/40.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/41.png)

* 查看容器内进程

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/42.png)

* 容器详细信息

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/43.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/44.png)

* 容器日志查看

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/45.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/46.png)

### 八、docker图形化管理工具

* Portainer单节点运行

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/47.png)

* 访问管理页面

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/48.png)

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h16/49.png)