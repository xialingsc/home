---
layout: article
title:  "简析Beego框架对Mysql数据库的支持及在项目中的应用"
categories: go
disqus: true
image:
    teaser: teaser/beego-mysql.jpg
---

{% highlight bash %}
{% raw %}
本文介绍了Beego框架对Mysql关系数据库的支持方法，并将在项目中遇到的一些问题及解决办法进行简单总结。
{% endraw %}
{% endhighlight %} 

---


##Beego是个啥

Beego 是一个快速开发Go应用的HTTP框架，它可以用来快速开发API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架，详见[Beego官网](http://beego.me/docs/intro/)。

---

##Beego架构

不废话，请右转到[Beego官网](http://beego.me/docs/intro/)

---

##Beego支持的数据库

笔者使用的Beego版本为1.4.3,官方介绍已支持的数据库有[Mysql](https://github.com/go-sql-driver/mysql)、[PostgreSQL](https://github.com/lib/pq)、[Sqlite3](https://github.com/mattn/go-sqlite3),这里结合官网描述及在项目中使用Mysql的方式进行简单小结。

---

##安装Beego ORM

go get github.com/astaxie/beego/orm

注：这里要求go语言的运行环境已经安装成功。

###1.前提

首先你需要安装Docker1.6.0及以上版本用于支持Registry2.0,其次还需要了解Docker的基本操作。

{% highlight bash %}
{% raw %}
如何从低版本升级到具体某个高版本，可以自行Google。也可以按如下方法升级到Docker最新版。
停止所有运行的容器
docker ps
docker stop contaierid
停止Docker服务
sudo service docker stop
下载最新的二进制文件
sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker 
&& chmod +x /usr/bin/docker
启动Docker服务
/etc/init.d/docker start
查看Docker版本
docker version
{% endraw %}
{% endhighlight %}

###2.创建Docker仓库数据和配置目录
{% highlight bash %}
{% raw %}
sudo mkdir -p /opt/docker/registry/data
sudo mkdir -p /opt/docker/registry/conf
{% endraw %}
{% endhighlight %}

###3.运行Docker仓库

运行仓库并命名为docker-registry,让镜像存储在宿主机上的/opt/docker/registry/data/下。

{% highlight bash %}
{% raw %}
sudo docker run -d -p 5000:5000 -v /opt/docker/registry/data:/tmp/registry-dev 
--name docker-registry registry:2.0.1
这里注意避免5000端口被占用而引起冲突,可以通过sudo docker ps 查看该容器是否已启动。接下来可以通过docker tag ,
docker push进行简单测试，具体用法可以查询docker help tag和docker help push.
{% endraw %}
{% endhighlight %}

###4.生成签名证书

{% highlight bash %}
{% raw %}
sudo openssl req -x509 -nodes -newkey rsa:2048 -keyout /opt/docker/registry/conf/docker-registry.key 
-out /opt/docker/registry/conf/docker-registry.crt
这里一定要注意：创建证书的时候，可以接收所有默认，直到CN位置时，如果你是准备让外网访问，就需要外网的域名；
如果是内网，可以输入运行私有仓库宿主机的别名。我们可以通过`ifconfig`查看ip,假定为10.10.62.103,通过
sudo vi /etc/hosts添加一行到该文件并保存退出，例如:10.10.62.103 devregistry。
这条命令主要是在/opt/docker/registry/conf/下创建证书docker-registry.key和docker-registry.crt,其中
docker-registry.crt放在随后与docker-registry进行交互的装有Docker客户端宿主机上。需要了解的是，这个宿主机
可以是运行docker-registry的server，也可以是能访问该域名或别名的装有docker的其他server。
{% endraw %}
{% endhighlight %}

###5.创建能够访问仓库的用户名和密码
为了让允许的用户登录访问，需要利用htpasswd创建用户和密码，并存储于/opt/docker/registry/conf/docker-registry.htpasswd文件.

(1)安装htpasswd
如果该命令已安装，可以略过此步,否则利用如下命令进行安装

`sudo yum install httpd-tools -y`

(2)创建用户和密码

`sudo htpasswd -c /opt/docker/registry/conf/docker-registry.htpasswd xl` 
第一个用户需要加-c参数，随后输入密码并确认。添加新用户不需要加-c参数。
例如：创建第二个用户：`sudo htpasswd /opt/docker/registry/conf/docker-registry.htpasswd testu`

###6.运行Nginx
{% highlight bash %}
{% raw %}
sudo docker run -d -p 443:443  -e REGISTRY_HOST="docker-registry" -e REGISTRY_PORT="5000" -e 
SERVER_NAME="localhost" --link docker-registry:docker-registry -v /opt/docker/registry/conf/
docker-registry.htpasswd:/etc/nginx/.htpasswd:ro -v /opt/docker/registry/conf:/etc/nginx/ssl:ro 
--name docker-registry-proxy containersol/docker-registry-proxy

这里使用了一个镜像去创建nginx容器，如果我们利用独立的nginx去进行配置的话，要求nginx版本在1.7.5以
上才能支持nginx.conf中add_header等配置。如果是作为内网使用，建议采用nginx容器这种方式就行。如果允
许让外网访问，建议先拷贝docker-registry-proxy容器中nginx.conf配置的内容，然后根据实际情况调整upstream中
相关ip,docker-registry.key,docker-registry.htpasswd等文件存放的位置。
注意这块nginx.conf配置的server非常重要，需要配置为之前提到的域名或别名。
{% endraw %}
{% endhighlight %}

到目前为止，我们已经成功运行了一个带有签名证书和用户名/密码的Docker Registry2.0了。


---


##如何进行测试才能说明已经搭建成功

注意：这里假定内网访问，且宿主机别名为devregistry

###1.验证登录
	
	方法一：telnet 域名/别名 443。
    再输入docker login -u testu -p 密码 -e '' devregistry:443，出现login Sucessed，就OK了。

	方法二:打开浏览器，在地址栏中输入https://域名:443/v2,在接受安全警告后输入用户名/密码，可以在页面上看到{}
	(方法二笔者未进行测试)

###2.在客户端配置证书
##### Ubuntu Docker 客户端

    (1)在hosts文件中加入devregistry并保存退出(上文已有描述，但这的确很重要)
	sudo vi /etc/hosts
	10.10.62.103 devregistry
	
	(2)启动docker服务
	sudo service docker start
	
	(3)创建一个扩展证书存放目录
	sudo mkdir /usr/share/ca-certificates/extra
	
	(4)拷贝docker-registry.crt证书至/usr/share/ca-certificates/extra
	
	(5)注册证书
	sudo dpkg-reconfigure ca-certificates
	

##### Mac Boot2docker 客户端

(1)方法一
{% highlight bash %}
{% raw %}
笔者曾按一些文档的描述尝试过，但并未成功。比如：进入boot2docker虚拟机,将之前生成的docker-registry.crt传入,并在
/var/lib/boot2docker/下创建bootlocal.sh，增加如下内容：
#!/bin/sh 
cat /var/lib/boot2docker/docker-registry.crt | sudo tee -a /etc/ssl/certs/ca-certificates.crt
这种方式在boot2docker启动时确实能将docker-registry.crt的内容自动添加到certificates.crt中，但重启docker或
boot2docker后，docker login devregistry:443仍然无法登录。
{% endraw %}
{% endhighlight %}
(2)方法二
为了简单，这里详细描述了操作步骤
{% highlight bash %}
{% raw %}
a.从mac进入到虚拟机
boot2docker ssh
b.在hosts文件中加入devregistry并保存退出
sudo vi /etc/hosts
10.10.62.103 devregistry
c.切换到root
d.进入目录
cd /etc/docker
e.创建certs.d目录
mkdir -p certs.d
f.进入certs.d
cd certs.d
g.创建devregistry:443目录
mkdir -p devregistry:443
h.将docker-registry.crt拷贝至devregistry:443
cp /var/lib/boot2docker/docker-registry.crt ./
i.停止并启动Docker
/etc/init.d/docker stop
/etc/init.d/docker start
{% endraw %}
{% endhighlight %}
测试：
docker login -u testu -p 密码 -e '' devregistry:443
Email: 
Login Succeeded
	
其实这是另一种在Ubuntu下配置证书的方法
	
	

###3.推送测试

假定以hello-world:latest镜像为例进行推送测试

标记镜像
docker tag hello-world devregistry:443/hello

推送镜像
docker push devregistry:443/hello


---


##需要注意
###1.参考文档资料的选择
官网介绍的资料比较全，如果你是首次搭建，一步步去做，你会发现停止容器、删除容器的操作在不断重复。同时，官网默认大家是掌握较多Linux及Nginx方面的相关知识,若按照官网介绍去操作，你一定会遇到坑，出现一些问题时就只能问Google,还相当郁闷。

国内很多同学写过类似的文档，找到适合自己的一篇文章，且与文档描述场景相同，比较难。

解决问题的大致方法是：静心分析,多google,多问问大牛,到论坛提问,静心分析

###2.心态总结

在遇到几个技术叠加在一起的新问题时，最好进行解耦，一步步确定问题并解决问题，同时保持小强心态，打不死。


---


##致谢

感谢Docker大咖Allen、孙健波、wangzhezhe、云栈科技王利俊的交流指导。

感谢以下文档的原作者

[Installing Docker Registry 2.0.1 using self signed certificates](http://mpas.github.io/post/2015/06/docker-setup-registry/)

[Running Secured Docker Registry 2.0](http://container-solutions.com/running-secured-docker-registry-2-0/)

[用 Nginx 来做私有 docker registry 的安全控制](http://cloud.51cto.com/art/201412/458680_all.htm)

[搭建docker内网私服（docker-registry with nginx&ssl on centos）](http://seanlook.com/2014/11/13/deploy-private-docker-registry-with-nginx-ssl/)

[腾讯云玩转Docker---搭建私有仓库](http://r2xe.com/wei-ming-ming-2/)

[部署 Docker Registry 服务](http://dockone.io/article/324)































