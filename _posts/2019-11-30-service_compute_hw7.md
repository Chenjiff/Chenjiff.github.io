---
layout: post  
title: "Go开发agenda程序"  
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

### 一、项目描述

项目名：agenda

说明：使用cobra开发的命令行程序，可以实现用户注册、登陆功能



### 二、业务设计

**用户注册**

1. 注册新用户时，用户需设置一个唯一的用户名和一个密码。另外，还需登记邮箱及电话信息。
2. 如果注册时提供的用户名已由其他用户使用，应反馈一个适当的出错信息；成功注册后，亦应反馈一个成功注册的信息。

**用户登录**

1. 用户使用用户名和密码登录 Agenda 系统。
2. 用户名和密码同时正确则登录成功并反馈一个成功登录的信息。否则，登录失败并反馈一个失败登录的信息。

### 三、测试结果

注册：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h7/1.png)

注册失败（用户名已被注册）：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h7/2.png)

登陆失败（密码错误）：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h7/3.png)

登陆成功：

![1](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h7/4.png)



### 四、项目地址

分享链接： [http://www.go-online.org.cn:8080/share/bo0bth676kvk304rcn10?secret=false](http://www.go-online.org.cn:8080/share/bo0bth676kvk304rcn10?secret=false)

