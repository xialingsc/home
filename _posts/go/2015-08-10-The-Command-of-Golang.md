---
layout: article
title:  "Golang学习系列(1)-那些简单的重要命令"
categories: go
disqus: true
image:
    teaser: teaser/go-command.jpg
---

{% highlight bash %}
{% raw %}
本文是学习Golang语言的系列文章之一，主要描述学习golang过程中碰到的浅显易懂的命令,
一方面是加深记忆，另一方面是想说明这些命令也是入门级学徒必须掌握的基础。
{% endraw %}
{% endhighlight %} 

---


## go build

这个命令主要用于测试编译。在包的编译过程中，在必要情况下，还可以同时编译与之关联的包。

{% highlight bash %}
{% raw %}
-普通包：执行完go build ,不会产生任何文件，如果需要在$GOPATH/pkg下生成相应文件，则要
执行go install.

-main 包：执行完go build,会在当前目录下生成一个可执行文件，如果需要在$GOPATH/bin下生成
对应文件，需要执行go install 或使用 go build -o outputpath/

如果只想编译某个文件，只需在在后面加上文件名即可，例如:go build hello.go

非main包在默认情况下编译输出的是package名，main包是第一个源文件的文件包，也还可以指定编
译输出的文件名，例如，go build -o xialingsc.exe

go build 会忽略目录下以"_"或"."开头的go文件

如果源代码针对不同的操作系统需要不同的处理，那么可以根据不同的操作系统后缀来命名文件。
例如，readfile_linux.go,readfile_drawin.go,readfile_windows.go.go build 会选择性编译
文件，Linux系统只编译readfile_linux.go，其他文件则被忽略。
{% endraw %}
{% endhighlight %}
---

## go clean

用来移除当前源码包里编译生成的文件，这些文件包括，_obj(旧的objects目录，Makefiles遗留)，

_test(旧的test目录)，_testmain.go（旧的gotest文件），test.out、build.out（旧的test记录）

*.[568ao] object文件,DIR（.exe）(由go build产生)，DIR.test（.ext)(由go test -c产生)，

MAINFILE(.exe) (由go build MAINFILE.go产生)

该命令最大的作用，清除编译文件后，上传git，保持源码清洁。

---

## go fmt

帮助格式化写好的代码文件，让写代码时不关心格式，写完后，轻松执行go fmt 文件名.go就好

提高效率。更多的时候可以采用gofmt,同时增加-w的参数，否则格式化结果不会写入文件，例如：

gofmt -w src 来格式真个项目。

---

## go get

动态获取远程代码。这个代码内部分为两部分，一是下载源码包，另一步是执行go install

为了让go get正常使用，需保证安装了合适的源码管理工具，并将这些命令加入到PATH中。

可通过go help remote 了解更多。


## go test

执行这个命令会自动读取源码目录下名为*_test.go文件，生成并运行测试用的可执行文件。

默认情况下不需要任何参数，也可带上参数，具体可参见go help testflag


## go doc

如何查看相应的package文档呢？

如果是builtin包，可以执行go doc builtin;如果是http包，执行go doc net/http;

查看某个包里面的函数，类似执行 godoc fmt Println,还可以查看相应代码 

godoc -src fmt Println


很棒的一点是，可以在终端执行godoc -http=:端口号，例如godoc -http=:8080 ,就可以在

浏览器中敲入127.0.0.1:8080进行文档内容的查看。


## 其他命令

{% highlight bash %}
{% raw %}
go fix        用来修复以前老版本的代码到新版本
go version    查看go当前的版本
go env        查看当前go的环境变量
go list       列出当前全部安装的package
go run        编译并运行go语言程序
{% endraw %}
{% endhighlight %}





























