# **Golang有关问题描述**
## **1 浅谈 GOPATH 和 GOROOT 区别**
## **2 Golang的命令行常用命令**
## **3 Golang 的版本管理方式(Go mod)**
## **4 参考**

## **1 浅谈 GOPATH 和 GOROOT 的区别**
1. GOROOT 是一个环境变量，不需要用户设置，一般下载完 Go 包就由自动配置好了，一般就是 Go 包的下载目录 <br>
2. GOPATH 也是一个变量，需要用户指定。GOPATH 是指代用户当前的工作空间，UNIX 下默认是 home/go, Windows 下默认是 C:\UserName\go，用户可以自己指定，但是不要与 Golang 的安装目录相同<br>
用户可以自己配置多个 GOPATH 目录（但是在 go get 下载包时，会自动下载到第一个 GOPTAH 目录），每个 GOPATH 包都必须有相同的目录结构， /src, /pkg, /bin <br>
GOPATH/src 目录包含源代码。
以下是一个 GOPATH 目录的布局：
```
/home/user/go/
    src/
        foo/
            bar/               (go code in package bar)
                x.go
            quux/              (go code in package main)
                y.go
    bin/
        quux                   (installed command)
    pkg/
        linux_amd64/
            foo/
                bar.a          (installed package object)
```
2.1) src 下面的路径包含的是导入路径或可执行文件名；<br>
2.2) pkg 下面主要存放用户执行 go install 后的归档文件；<br>
2.3) bin 下面主要存放用户执行 go install 后生成的可执行文件；<br>
Ps: go build 和 go install 应该以整个代码包为单位，而 go run 只需在保证源码文件(main所在的文件)依赖正确即可执行，但并不保证逻辑正确。<br>

## **2 Golang的命令行常用命令**
1. go build: 
用于编译指定的代码包或 Go 程序的命令，源文件会编程生成可执行文件并存放在指定的目录下(-o 目录名/可执行文件名)，默认为源文件所在目录，可执行文件名为源文件的文件名。
2. go clean：
用于清除因执行其它 go 命令而遗留的临时文件和目录。
3. go doc：
Go 代码文档化工具，用于显示 Go 语言代码以及程序实体的文档。
4. go env：
Go 环境变量信息显示工具。
5. go fmt：
用于格式化代码包的 Go 源码文件。
6. go get：
用于下载指定代码包及其依赖包，后面会用 go mod 替换（这也是趋势），后面会介绍。
7. go install：
用于编译或安装指定代码包及其依赖包，结合上述 GOPATH/bin 和 GOPATH/pkg 生成相应的文件。
8. go run：
用于编译并运行指定的源码文件。不生成可执行文件，直接运行。
9. go test：
用于测试指定的代码包，前提是代码包目录必须存在指定的测试源码文件。在一些代码模式的功能性能测试很有用。
10. go version：
显示当前安装的 Go 语言的版本信息。

## **3 Golang 的版本管理方式(Go mod)**
在 Golang 的推动过程中，许多 Gopher 都推动了包管理工具的建设，例如之前的 vendor 等，但是我学习 Golang 的时候比较晚，基本到了 Go1.9xxx 版本<br>
Golang1.11 之后官方正式退出 go mod 作为包管理工具，个人尝试后，觉得非常好用！(就我这种使用 golang 几乎没做过什么大项目，导入包基本就是 go get 一把梭的人来说，都觉得 go mod 很不错)<br>
想象一个场景： 项目 P 在刚开始时，引入了包 D1.0，那么就 import "D"---> go get D 好了，这一切很顺利，但是随着项目 P 需求日益增多，它又需要导入项目 C，C也用到了D1.0，而此时 D 已经升级为 D2.0 了，那如果要使用高版本的 D ，那么好像就只能手动把所有的 D 升级到2.0版本，如果一个这样的项目，似乎不是太大的问题，如果好包很大，耦合很多，那么………………<br>
go mod 就是为了解决这样一个问题：
>1)添加依赖<br>
>2)更新依赖<br>
>3)删除依赖<br>
>4)多版本依赖<br>

### **3.1 Go mod 的使用前提**
1) Go 版本 >= 1.11
2) 设置 GO111MODULE 变量：
export GO111MODULE=off, 不支持 go mod 功能，还是会通过 GOPATH 模式来查找路径；
export GO111MODULE=on, 支持 go mod 功能，摒弃 GOPATH；
export GO111MODULE=auto（Go1.11后的默认值），会根据当前工作目录来决定是否使用 go mod 模式
>当modules 功能启用时，依赖包的存放位置变更为$GOPATH/pkg，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。

### **3.2 Go mod 的使用**
测试的工程树如下：
```
github.com/xiaochengsky/golang-logger
    logs.go                                     (package log)
    logs_test.go                                (package log)
```

#### **3.1.1 添加依赖**
1) 在当前工作目录(GOPATH外，如果GOPATH内，则设置GO111MODULE=on)下创建 go.mod 文件；
$ cd <project path outside $GOPATH/src> 
$ go mod init <project name>

```
nikkoyang@nikkoyang-LB0:~$ cd ~/gomod/log_test/
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ 
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go mod init log_test
go: creating new go.mod: module log_test
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ ls -a
.  ..  go.mod  .idea  main.go
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ 
```

先看看此时 go.mod 文件：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.mod 
module log_test

go 1.12
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ 
```
只有 module名 和 Go 的版本名。

2) 现在再 import 要引用的包在 github 上的路径
```
package main

import (
	"github.com/xiaochengsky/golang-logger"
	"os"
)

var logger *log.Logger

func main() {
	file, _ := os.OpenFile("./logs.log", os.O_CREATE | os.O_RDWR | os.O_APPEND, 0x777)
	file.Chmod(os.ModePerm)
	
	logger = log.NewLogger(file)
	logger.SetLevel("debug")
	logger.Debug("====== go mod test ======")
}

``` 
注：我的 golang-logger 目录下的包名是"log"，上述使用的并非官方的 log 库。

3) 现在再在 go mod 模式下自动解决依赖问题：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go build
module log_test

go 1.12

require github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5
```
同时，该目录下会生成一个 go.sum 文件来记录依赖树(depender tree)：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.sum 
github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5 h1:9G7eA+vVbFn1Kf8mZoVdtdLh9sSQVodHcgKyvjfmEU8=
github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=
```
也可以使用 "go list -m all" 列出当前项目的依赖包和代码版本：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go list -m all
log_test
github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5
```
注：go mod 模式下，默认会拉取最新的 release tag，若无 tag，则拉取最近 commit 的包（我之前的 logger 日志库没有加 tag，也只 push 了一次）。

>依赖包的目录在 ~/GOPATH/pkg/mod 中，例如此工程下所在的位置：
```
nikkoyang@nikkoyang-LB0:~/GoProj/pkg/mod/github.com/xiaochengsky$ ls
golang-logger@v0.0.0-20190903073016-7d1d2ef198e5
```

#### **3.1.2 更新依赖(依赖包版本的切换)**
我已将 logger 日志库的重新推送了新的 tag V2.0，现在准备把此工程的 logger 库版本从 v0.0.0 更新到新的 tag。<br>
直接使用 go get <project address>@<tag>更新即可(也可以手动编辑 go.mod 文件的 require 项，重新 build 更新依赖)：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go get github.com/xiaochengsky/golang-logger@v2.0.0
go: finding github.com/xiaochengsky/golang-logger v2.0.0
go: downloading github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible
go: extracting github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ 
```
再次使用 "go list -m all" 列出当前项目的依赖包和代码版本，可以看到已经更新了依赖：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go list -m all
log_test
github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible
```
此时，再查看 go.mod 文件和 go.sum 文件，可以看到 go.mod 已经替换成了新的依赖，而 go.sum 也已经添加新的依赖到 depender tree 中。<br>
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.mod 
module log_test

go 1.12

require github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible


nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.sum 
github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5 h1:9G7eA+vVbFn1Kf8mZoVdtdLh9sSQVodHcgKyvjfmEU8=
github.com/xiaochengsky/golang-logger v0.0.0-20190903073016-7d1d2ef198e5/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=
github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible h1:xGyexoBk42rv2glUom9aWpB6qd+aBT44k4ebw4RW8mU=
github.com/xiaochengsky/golang-logger v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=
```

同时查看 /GOPATH/pkg，可以看到存在两个依赖的版本：
```
nikkoyang@nikkoyang-LB0:~/GoProj/pkg/mod/github.com/xiaochengsky$ ls
golang-logger@v0.0.0-20190903073016-7d1d2ef198e5
golang-logger@v2.0.1-0.20190903073016-7d1d2ef198e5+incompatible
```
注：打 tag 时，还是要遵循 semver 规范。

#### **3.1.3 删除(未使用的)依赖**
使用 go mod tidy 即可：<br>
1) 现将 import 和代码段全部注释：
```
package main

import (
	//"github.com/xiaochengsky/golang-logger"
	//"os"
)

//var logger *log.Logger

func main() {
	//file, _ := os.OpenFile("./logs.log", os.O_CREATE | os.O_RDWR | os.O_APPEND, 0x777)
	//file.Chmod(os.ModePerm)
	//
	//logger = log.NewLogger(file)
	//logger.SetLevel("debug")
	//logger.Debug("====== go mod test ======")
}
```

2) 执行 go mod tidy 命令：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go mod tidy
```

3) 查看 go.mod 和 go.sum 以及 go list -m all 的信息：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.mod 
module log_test

go 1.12

nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.sum
[输出为空](手动添加)

nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go list -m all
log_test
```
可以看到，go.mod 文件已经没有显示出依赖关系(和最初执行 go mod init一样)，go.sum 文件为空，go list -m all 也不存在依赖关系树<br>

4) 取消注释，执行 go build 即可恢复：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go build
go: finding github.com/xiaochengsky/golang-logger v2.0.0+incompatible
go: downloading github.com/xiaochengsky/golang-logger v2.0.0+incompatible
go: extracting github.com/xiaochengsky/golang-logger v2.0.0+incompatible
```
4) 再次查看 go.mod 和 go.sum 以及 go list -m all 的信息：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.mod 
module log_test

go 1.12

require github.com/xiaochengsky/golang-logger v2.0.0+incompatible

nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.sum 
github.com/xiaochengsky/golang-logger v2.0.0+incompatible h1:/BEZ/l3PnLtBta2QZilmJvYWHjLiR+8w+1I2yEuqiv8=
github.com/xiaochengsky/golang-logger v2.0.0+incompatible/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=

nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go list -m all
log_test
github.com/xiaochengsky/golang-logger v2.0.0+incompatible
```

#### **3.1.4 多版本依赖**
现在我重新推送了一个新的版本（新增了一个 logger 文件夹, 并附带了 logger.go 文件夹），附加了新的 tag(v6.0.0)。<br>
```
github.com/xiaochengsky/golang-logger
    logger/
        logger.go                     (package logger)
    logs.go                          (package log)
    logs_test.go                     (package log)
```
测试使用 v2.0.0 版本的 package log， 使用 v6.0.0 版本的 package loggers

修改代码如下：
```
package main

import (
	"github.com/xiaochengsky/golang-logger"
	"os"
	v6 "github.com/xiaochengsky/golang-logger/logger"
)

var logger *log.Logger

func main() {
	file, _ := os.OpenFile("./logs.log", os.O_CREATE | os.O_RDWR | os.O_APPEND, 0x777)
	file.Chmod(os.ModePerm)

	logger = log.NewLogger(file)
	logger.SetLevel("debug")
	logger.Debug("====== go mod test ======")

	v6.Logger()
}
```

先 go get 单独获取 tag v6.0.0 要使用的包：go get github.com/xiaochengsky/golang-logger/logger@v6.0.0<br>  
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ go get github.com/xiaochengsky/golang-logger/logger@v6.0.0
go: finding github.com/xiaochengsky/golang-logger/logger v6.0.0
go: finding github.com/xiaochengsky/golang-logger v6.0.0
go: downloading github.com/xiaochengsky/golang-logger v6.0.0+incompatible
go: extracting github.com/xiaochengsky/golang-logger v6.0.0+incompatible
```

此时再查看 go.sum 依赖树：
```
nikkoyang@nikkoyang-LB0:~/gomod/log_test$ cat go.sum 
github.com/xiaochengsky/golang-logger v2.0.0+incompatible h1:/BEZ/l3PnLtBta2QZilmJvYWHjLiR+8w+1I2yEuqiv8=
github.com/xiaochengsky/golang-logger v2.0.0+incompatible/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=
github.com/xiaochengsky/golang-logger v6.0.0+incompatible h1:Xe1Zos/ifmAsSWsVz0COITo9XKdQduyOgsC+T9PSwmk=
github.com/xiaochengsky/golang-logger v6.0.0+incompatible/go.mod h1:Rl1SW6Iho2JTkldmRKXJOckcivnyK/Gi3Q4ojKtz0A4=
```
可以看到，package log 使用的是 v2.0.0 版本的库，package logger 使用的是 v6.0.0 版本的库。

关于 Go mod 的更多 rules 和 issus 可以参考后面的官方文档链接，此处只是起到抛砖引玉的作用吧。

## **4 参考**
1):<br>
<https://golang.google.cn/doc/code.html#Introduction/> <br>
2):<br>
<<Go并发编程实战 郝林 第二版>> <br>
3):<br>
<https://juejin.im/post/5c8e503a6fb9a070d878184a/> <br>
<https://talks.godoc.org/github.com/myitcv/talks/2018-08-15-glug-modules/main.slide#1/> <br>
<https://github.com/golang/go/wiki/Modules/>  <br>

   

