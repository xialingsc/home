---
layout: article
title:  "LVS实践系列(2)-在RealServer的Ubuntu12.04环境下源码安装apache2"
categories: linux
disqus: true
image:
    teaser: teaser/apache.jpg
---

{% highlight bash %}
{% raw %}
这篇短文主要描述在ubuntu环境下源码安装apache2的遇到的问题以及解决方式。
思路：
1.ubuntu12.04默认安装下没有安装C++，需要先安装C++编译所依赖的相关组件。
2.安装apache多个依赖库，包括：apr, apr-util,pcre,zlib-devel.
3.安装apache2
{% endraw %}
{% endhighlight %}

---

##c++编译依赖组件

刚装好的Ubuntu系统中已经有GCC了，但是这个GCC什么文件都不能编译，因为没有一些必须的头文件，所以要安装build-essential这个软件包，安装了这个包会自动安装上g++,libc6-dev,linux-libc-dev,libstdc++6-4.1-dev等一些必须的软件和头文件的库。

安装所需要的软件包： 
`sudo apt-get install build-essential`


##安装apache依赖库

为了方便，我们先进入/usr/local/src目录
{% highlight bash %}
{% raw %}
1.源码安装apr
(1)下载
sudo wget http://archive.apache.org/dist/apr/apr-1.4.5.tar.gz
(2)解压
tar -zxvf apr-1.4.5.tar.gz
或者 gzip -d apr-1.4.5.tar.gz ,tar xvf apr-1.4.5.tar
(3)进入apr-1.4.5,指定apr安装路径
cd apr-1.4.5
./configure -prefix=/usr/local/apr
(4)编译
make
(5)安装
make install

2.源码安装apr-util
(1)下载
sudo wget http://archive.apache.org/dist/apr/apr-util-1.3.12.tar.gz
(2)解压
tar -zxvf apr-util-1.3.12.tar.gz
(3)进入apr-util-1.3.12,指定apr-util安装路径时一并带上 apr安装路径
cd apr-util-1.3.12
./configure -prefix=/usr/local/apr-util --with-apr=/usr/local/apr
(4)编译
make
(5)安装
make install

3.源码安装pcre
(1)下载
sudo wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.10/pcre-8.10.zip
(2)解压
sudo unzip pcre-8.10.zip -d ./
(3)进入pcre-8.10,指定pcre安装路径
cd pcre-8.10
./configure -prefix=/usr/local/pcre
(4)编译
make
(5)安装
make install

4.安装zlib-devel
sudo apt-get install zlib1g-dev
若不安装该包，则在安装Apache2时会出现checking whether to enable mod_deflate... configure: error: 
mod_deflate has been requested but can not be built due to prerequisite failures错误
此包不建议下载安装，因多次尝试未成功。
{% endraw %}
{% endhighlight %}

##安装Apache2

为了方便，我们先进入/usr/local/src目录

{% highlight bash %}
{% raw %}
(1)手工下载 httpd-2.4.10.tar.gz  地址：http://httpd.apache.org/download.cgi#apache24
(2)解压
tar -zxvf httpd-2.4.10.tar.gz
(3)进入httpd-2.4.10,指定Apache安装路径
cd httpd-2.4.10
./configure -prefix=/usr/local/apache2 --enable-deflate --enable-expires --enable-headers --enable-modules=most 
--enable-so --with-mpm=worker --enable-rewrite --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util 
--with-pcre=/usr/local/pcre

此步可能以下几种错误：
a.出现checking whether to enable mod_deflate... configure: error: mod_deflate has been requested but 
can not be built due to prerequisite failures
解决办法：sudo apt-get install zlib1g-dev

b. 出现/bin/bash: bison: command not found
解决办法：sudo apt-get install bison

c.出现/bin/bash: flex: command not found
解决办法：sudo apt-get install flex

(4)检查apache2是否有安装错误
echo $?
若返回0，则表示安装正确

(5)编译
make
(6)安装
make install
{% raw %}
{% endhighlight %}

##Apache2启动、停止、重启

前提apahce安装目录为/usr/local/apache2,若不是，请相应修改

{% highlight bash %}
{% raw %}
(1)apahce启动
/usr/local/apache2/bin/apachectl start

(2)apache停止
/usr/local/apache2/bin/apachectl stop 

(3)apache重新启动
/usr/local/apache2/bin/apachectl restart

(4)在重启 Apache 服务器时不中断当前连接
/usr/local/sbin/apachectl graceful 

(5)若apache安装成为linux的服务则可以： 
service httpd start 启动 
service httpd restart 重新启动 
service httpd stop 停止服务

{% endraw %}
{% endhighlight %}

##如何检查是否工作

打开浏览器，输入http://127.0.1.1

可以看到`Its work` 表示已经启动了


##后记

在我们安装apache2是配置了一堆参数，他们的目的是什么？为什么要这么配？

./cofigure后面的参数说明：
--prefix=/opt/demo/apache2 这个选项指定apache的安装位置，可以根据实际情况更改，如/usr/local等

--enable-rewrite 实时重写URL

--enable-so DSO动态模块加载，很多第三方模块都需要此选项支持

--enable-ssl 使用安全套接字层(SSL)和传输层安全(TLS)协议实现高强度加密传输

--enable-vhost-alias 提供大批量虚拟主机的动态配置支持

--enable-proxy 提供HTTP/1.1的**/网关功能支持

--enable-proxy-connect mod_proxy的扩展，提供HTTP connect方法支持

--enable-proxy-http mod_proxy的扩展，提供http支持

--enable-proxy-ajp mod_proxy的扩展，提供Apache JServ Protocol支持，ajp是对tomcat提供整合的新模块

--with-mpm=prefork MPM是你想要使用的多路处理模块的名字，这里选择prefork，worker模式的mpm是一种混合型模式，可以处理海量请求的同时又保持了基于进程mpm的稳定性，具体可以google一下

##参考文档

[linux(ubuntu 14.04)编译安装apache](http://blog.csdn.net/ejc2001/article/details/24765355)

[apache安装过程中的错误](http://blog.chinaunix.net/uid-20765335-id-1850491.html)

[安装apache可能遇到的问题](http://dinghaoliang.blog.163.com/blog/static/12654071420097254115963/)

