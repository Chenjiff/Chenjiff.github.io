---
layout: post  
title: "使用Go进行编程"  
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



### 一、开发环境安装

安装过程不是重点，简述一下过程完事。

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



### 二、如何使用Go编程

#### 1. 工作空间和环境变量的配置、包路径

之前已经配置过了，跳过

创建包路径：

```
mkdir -p $GOPATH/src/github.com/user
```

#### 2. 第一个程序

创建包目录：

```
mkdir $GOPATH/src/github.com/user/hello
```

在目录中创建hello.go，并编写程序:

```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, world.\n")
}
```

安装：

```
go install github.com/user/hello
```

执行：

```
$ hello
```

![12](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h3/1.png)

推送到远程仓库：

```
git init
git remote add ...
git add ...
git commit ...
git push ..
```

#### 3. 第一个库

创建一个包目录作为库的目录：

```
mkdir $GOPATH/src/github.com/user/stringutil
```

在该目录中创建名为 `reverse.go` 的文件，内容如下：

```
// stringutil 包含有用于处理字符串的工具函数。
package stringutil

// Reverse 将其实参字符串以符文为单位左右反转。
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

执行命令测试编译：

```
go build
```

修改原hello.go文件：

```
package main

import (
	"fmt"

	"github.com/user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("!oG ,olleH"))
}
```

安装并执行：

```
go install github.com/user/hello
hello
```

![12](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h3/2.png)

#### 4. 测试

在 `$GOPATH/src/github.com/user/stringutil/` 下创建reverse_test.go，其内容如下：

```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

使用 `go test` 运行该测试：

```
$ go test github.com/user/stringutil
ok  	github.com/user/stringutil 0.165s
```

![12](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h3/3.png)

#### 5. 远程包

执行下面命令便可看到demo：

```
$ go get github.com/golang/example/hello
$ $GOPATH/bin/hello
```

![12](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h3/4.png)



### 三、相关项目地址

Github传送门：<https://github.com/Chenjiff/HelloGo>



### 四、实验心得

这次实验主要进行了Go语言的安装配置和简单的编程demo的实现，第一次接触这门语言，和之前的传统语言语法差距比较大，细节还需要多记。