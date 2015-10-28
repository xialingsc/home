---
layout: article
title:  "如何在Ubuntu中安装Go及Beego"
categories: go
disqus: true
image:
    teaser: teaser/golang.jpg
---

{% highlight bash %}
{% raw %}
刚学习Golang时,笔者对GOPATH比较困惑，在Ubuntu及Mac OS下分别使用Go后，对其有了一些理解。
本文主要回顾了安装Go过程中的步骤、问题及解决办法。
{% endraw %}
{% endhighlight %} 

---


##在Ubuntu中安装Go及Beego

### 安装Go

sudo apt-get install golang

也可以采用[go官网](http://golang.org/doc/install)方法进行安装，这可能需要翻墙哦。


### 设置环境变量GOPATH

1.概念

GOROOT是指安装支撑Go运行环境的目录。

GOPATH是指放置go程序代码的目录,该目录的第一级子目录包括三个子目录以上,即src,pkg,bin。


2.单一GOPATH设置

笔者使用zsh，故需要在.zshrc中增加GOPATH的设置，并让设置起作用。
{% highlight bash %}
{% raw %}
vi ~/.zshrc
#增加GOPATH设置,/usr/local/go为GOROOT,/home/xialingsc/go为GOPATH
export GOPATH=/home/xialingsc/go
export PATH=/usr/local/go/bin:$GOPATH/bin:$PATH
#保存退出并使其生效
source ~/.zshrc
{% endraw %}
{% endhighlight %}

3.多GOPATH设置

如果需要设定多个GOPATH,即有多个Go项目代码都可以运行,该如何处理呢？
{% highlight bash %}
{% raw %}
vi ~/.zshrc
export GOPATH=$HOME/go:$HOME/go-test::$HOME/ceshi:$HOME/luckperson
export PATH=/usr/local/go/bin:${GOPATH//://bin:}/bin:$PATH
#保存退出并使其生效
source ~/.zshrc
{% endraw %}
{% endhighlight %}

4.测试是否安装正确

在终端输入go version,返回版本号即表明安装正确。

### 安装Beego框架

go get github.com/astaxie/beego

这里需要说明的是，如果是多个GOPATH，则beego被默认安装在第一个GOPATH/src下。

如何测试已经安装正确了呢？

{% highlight bash  %}
{% raw %}
#本例以/home/xialingsc/go为GOPATH,进入该目录后，建立包目录，并在该包目录下创建hello.go
cd /home/xialingsc/go
mkdir main
vi hello.go

#hello.go内容 开始
package main
import "github.com/astaxie/beego"
func main() {
    beego.Run()
}
#hello.go内容 结束

#编译并运行
go build -o hello hello.go
./hello
#浏览器中查看
http://localhost:8080
如果看到beego欢迎页面，则表明安装成功了
{% endraw %}
{% endhighlight %}



### 安装bee 工具

go get github.com/beego/bee

为什么要安装bee？目的是在运行我们项目时方便快捷，直接可以通过bee run 就行了，框架去进行编译等动作。

可能遇到的问题：

若还出现package github.com/beego/bee: cannot download, $GOPATH not set. For more details 

see: go help gopath

请试着去掉sudo 或者 修改 /usr/local/bin 的权限


测试bee是否安装正确

可以通过在终端直接敲bee 回车，若出现如下内容就说明bee安装成功了

Bee is a tool for managing beego framework.

Usage:

bee command [arguments]

The commands are:

...


## 遇到的问题

回顾刚接触使用Go这门新语言时，遇到的最大问题就是在于GOPATH的设置上，如果我们在用go get进行相关包的安装时，一定要保证GOPATH的正确性。



## 参考资料

[golang官网](http://golang.org/doc/instal)

[Beego官网](beego.me)






























