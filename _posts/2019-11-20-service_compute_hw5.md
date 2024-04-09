---
layout: post  
title: "Go开发命令行程序"  
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

### 一、开发实践要求

使用 golang 开发 [开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html) 中的 **selpg**

提示：

1. 请按文档 **使用 selpg** 章节要求测试你的程序
2. 请使用 pflag 替代 goflag 以满足 Unix 命令行规范， 参考：[Golang之使用Flag和Pflag](https://o-my-chenjian.com/2017/09/20/Using-Flag-And-Pflag-With-Golang/)
3. golang 文件读写、读环境变量，请自己查 os 包
4. “-dXXX” 实现，请自己查 `os/exec` 库，例如案例 [Command](https://godoc.org/os/exec#example-Command)，管理子进程的标准输入和输出通常使用 `io.Pipe`，具体案例见 [Pipe](https://godoc.org/io#Pipe)

### 二、设计说明

本次开发的是一个命令行程序，总体的流程设计如下：

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/lc.png)

在流程中重点要解决的几个问题：

**1. 参数提取**

本次开发的是一个命令行程序，程序的外部输入就是输入的命令及其参数，那么首先要考虑如何获得输入的命令内容并进行解析、验证，在示例c代码中是通过main函数直接获得字符串类型的输入，然后对它进行拆分，从头逐步解析同时验证。在Go中，main函数并没有传入参数，而是可以通过os包或者flag（pflag）包就可以进行快速方便地提取参数，本次开发我选择pflag，使用IntP，Parse等函数。

**2. 错误处理**

在程序执行流程中会产生许多错误，例如：参数使用不当、文件读取异常等，程序需要在出错时返回有用的信息并结束。因此，可以在出错流程中适时输出出错原因或正确用法，然后使用os.Exit(int)退出，在函数中使用不同的错误码来识别不同错误。但是文件处理时不可以直接使用os.Exit(int)退出，此时文件流还未关闭，此时可以使用panic函数配合defer来确保文件流的关闭（panic函数会在处理完错误后执行defer的代码），所以本代码**统一使用panic处理错误**。

**3.输入和输出处理**

* 来源

 输入有两种来源，标准输入和指定文件，我们只需要一视同仁地转换为io.Reader接口就可以，前者通过fmt.Stdin（类型为*File）获取，后者通过os.open()获取（返回类型同样为\*File）。

输出目的类似，os.Stdout和cmd.StdinPipe()，后者是使用管道进行数据流的转移，使用了os/exec，同样都把他们转换为os.Writer接口。

* 方式

  输入有两种方式：按行和按\\f分割符，bufio包中有很多处理流的方法可以帮助我们节省很多精力，bufio.Reader中，ReadLine()方法可以按行读取，ReadSlice('\f')可以按\\f分割符读取。

* 流的关闭

输入源的关闭上文已说过使用defer file.close()就可以确保关闭。

输出源的关闭主要考虑cmd中管道流的关闭。但是我们看官方Go doc中StdinPipe()的说明：

*The pipe will be closed automatically after Wait sees the command exit. A caller need only call Close to force the pipe to close sooner.*

也就是说pipe会自动关闭，所以可以不考虑此输出源的关闭。

### 三、代码实现

**以下为重点实现：**

命令的存储结构体：

```go
type scom struct {
	st, ed int //start,end page.No
	isl bool //true for line based, or \f based
	lnum int //line number. Used when isl == true
	dest string //Output destination. Empty string means Stdout
	fn string //Input source. Empty string means Stdin
}
```

main函数的流程：

```Go
var sc scom
parseArgs(&sc)
//Show the print information read just now
printCom(&sc)
processInput(&sc)
```

参数提取：

```go
st := pflag.IntP("start", "s", -1, "The to-print start page.No")
//...
pflag.Usage = usage
pflag.Parse()
```

输入源获取，重点：使用defer处理文件流关闭：

```go
var reader io.Reader
var file os.File
if sc.fn != "" {
	_, errEx:= os.Stat(sc.fn)
    //...
	file, err := os.Open(sc.fn)
	//...
	reader = file
} else {
	file := os.Stdin
	reader = file
}
defer file.Close()
```

输出到bytes.Buffer

```go
for {
	bs, err := rd.ReadSlice('\f')
	if cp >= sc.st && cp <= sc.ed {
		buf.Write(bs)
		buf.WriteByte('\f')
	}
	cp++
	if err == io.EOF {
		cp--
		break
	}
}
```

通过exec获得“-dXXX”的输出流：

```go
cmd := exec.Command("lp", "-d"+dest)
fout, err := cmd.StdinPipe()
if err != nil {
    //...
}
cmd.Stdout = os.Stdout
cmd.Stderr = os.Stderr
errr := cmd.Run()
if errr != nil {
    //...
}
```



### 四、测试结果

测试文件如下，共有11行，包含4个分页符（vim中ctrl+L插入显示^L）：

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/txt.png)

1.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/1.png)

2.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/2.png)

3.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/3.png)

3.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/3.png)

4.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/4.png)

5.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/5.png)

6.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/6.png)

7.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/7.png)

8.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/8.png)

9.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/9.png)

10.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/10.png)

11.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/11.png)

12.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/12.png)

13.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/13.png)

14.

![5](https://raw.githubusercontent.com/Chenjiff/Chenjiff.github.io/master/img/in-post/SC/h5/14.png)

### 五、代码地址

分享链接： [http://www.go-online.org.cn:8080/share/bnvltku76kvk304rcmtg?secret=false]( http://www.go-online.org.cn:8080/share/bnvltku76kvk304rcmtg?secret=false)

### 六、心得体会

此次作业写了一个命令行程序，算是第一个go程序，之前也没有写过命令行程序，收获满满，对go语言的使用、命令行程序的编写原理有了很多新的认识。在编写过程中有遇到一些困难，但是基本靠自己的能力可以解决，过程不容易，结果很美好。

