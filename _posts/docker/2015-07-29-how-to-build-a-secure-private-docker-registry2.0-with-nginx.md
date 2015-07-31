---                                                                                                                                                                                                       
layout: article
title:  "如何利用nginx搭建一个较安全的私有Docker Registry 2.0"
toc: true
disqus: true
categories: docker
---


    作者：夏令
    时间：2015-07-29

    本文简要介绍了搭建Docker Registry2.0的场景，着重描述在Ubuntu 14.04上如何利用nginx来搭建一个相对安全的私有Docker Registry2.0，同时也将搭建过程中遇到的一些问题进行总结，目的是让感兴趣的同学能根据这篇文章内容自行成功搭建私有仓库，少走弯路。

##一、背景

2013年底至2014年初，部门大咖对Docker进行了技术预研，并在国内相关技术会议上进行了推荐，当时Docker还并未发布1.0版本。2014年年中，为了配合新产品的发版，我们在基础运行环境中预装了Java、Git、Zsh等多个工具。利用shell从git上拉取各个组件进行打包编译，通过fabric自动化脚本将该产品部署在Docker容器中，用于产品单机和集群环境的功能、性能测试。在这个过程中，我们解决了字符集、时间不同步等诸多问题，最终该产品的基础镜像已累积到上GB的大小。

##二、为什么要选择Nginx+Registry2.0

在开发Docker Web容器管理控制平台过程中，Registry1.0、2.0两个版本均有使用，但2.0的速度比1.0快。从基础镜像大小及公司内外网络带宽情况考虑，结合Docker官网上推荐的方式，选定了Nginx+Registry2.0的技术方案。

##三、选择靠谱的参考资料

为了构建安全私有仓库，先从Docker官网的[Delopying a registry server](https://docs.docker.com/registry/deploying/)和[Authentication](https://docs.docker.com/registry/authentication/)进行了学习，在实际操作过程中遇到一些问题(在[DockerOne](http://dockone.io/question/436)和[Docker官网](https://github.com/docker/docker-registry/issues/1023)的提问)后，又查找相关同行分享的搭建经验，咨询了多位Docker大咖后，结合国外同行一篇[文章](http://mpas.github.io/post/2015/06/docker-setup-registry/)的部分内容,完成了这个看上简单但非常糟心的任务。自己经历过的痛苦就尽量不要让别人去重复了，希望这篇文章是大家搭建过程中的靠谱资料。

##四、搭建过程
###1.前提

首先你需要安装Docker1.6.0及以上版本用于支持Registry2.0,其次还需要了解Docker的基本操作。

>如何从低版本升级到具体某个高版本，可以自行Google。也可以按如下方法升级到Docker最新版。

>-停止所有运行的容器

>`docker ps`

>`docker stop contaierid`

>-停止Docker服务

>`sudo service docker stop`

>-下载最新的二进制文件

>`sudo wget https://get.docker.com/builds/Linux/x86_64/docker-latest -O /usr/bin/docker && chmod +x /usr/bin/docker`

>-启动Docker服务

>`/etc/init.d/docker start`

>-查看Docker版本

>`docker version`


###2.创建Docker仓库数据和配置目录

`sudo mkdir -p /opt/docker/registry/data`

`sudo mkdir -p /opt/docker/registry/conf`


###3.运行Docker仓库

运行仓库并命名为docker-registry,让镜像存储在宿主机上的/opt/docker/registry/data/下。

`sudo docker run -d -p 5000:5000 -v /opt/docker/registry/data:/tmp/registry-dev --name docker-registry registry:2.0.1`

>这里注意避免5000端口被占用而引起冲突,可以通过`sudo docker ps` 查看该容器是否已启动。接下来可以通过`docker tag` ,`docker push`进行简单测试，具体用法可以查询`docker help tag`和`docker help push`.

###4.生成签名证书

`sudo openssl req -x509 -nodes -newkey rsa:2048  -keyout /opt/docker/registry/conf/docker-registry.key -out /opt/docker/registry/conf/docker-registry.crt`

>这里一定要注意：创建证书的时候，可以接收所有默认，直到CN位置时，如果你是准备让外网访问，就需要外网的域名；如果是内网，可以输入运行私有仓库宿主机的别名。我们可以通过`ifconfig`查看ip,假定为10.10.62.103,通过`sudo vi /etc/hosts`添加一行到该文件并保存退出，例如:10.10.62.103 devregistry。

>这条命令主要是在/opt/docker/registry/conf/下创建证书docker-registry.key和docker-registry.crt,其中docker-registry.crt放在随后与docker-registry进行交互的装有Docker客户端宿主机上。需要了解的是，这个宿主机可以是运行docker-registry的server，也可以是能访问该域名或别名的装有docker的其他server。


###5.创建能够访问仓库的用户名和密码
为了让允许的用户登录访问仓库，我们需要利用htpasswd创建用户和密码，并存储在/opt/docker/registry/conf/docker-registry.htpasswd文件中.
(1)安装htpasswd
如果该命令已安装，可以略过此步。否则利用如下命令进行安装

`sudo yum install httpd-tools -y`

(2)创建用户和密码

`sudo htpasswd -c /opt/docker/registry/conf/docker-registry.htpasswd xl` 

>第一个用户需要加-c参数，随后输入密码并确认。添加新用户不需要加-c参数。

>例如：创建第二个用户：`sudo htpasswd /opt/docker/registry/conf/docker-registry.htpasswd testu`

###6.运行Nginx
`sudo docker run -d -p 443:443  -e REGISTRY_HOST="docker-registry" -e REGISTRY_PORT="5000" -e SERVER_NAME="localhost" --link docker-registry:docker-registry -v /opt/docker/registry/conf/docker-registry.htpasswd:/etc/nginx/.htpasswd:ro -v /opt/docker/registry/conf:/etc/nginx/ssl:ro --name docker-registry-proxy containersol/docker-registry-proxy`

>这里使用了一个镜像去创建nginx容器，如果我们利用独立的nginx去进行配置的话，要求nginx版本在1.7.5以上才能支持nginx.conf中add_header等配置。如果是作为内网使用，建议采用nginx容器这种方式就行。如果允许让外网访问，建议先拷贝docker-registry-proxy容器中nginx.conf配置的内容，然后根据实际情况调整upstream中相关ip,docker-registry.key,docker-registry.htpasswd等文件存放的位置。注意这块nginx.conf配置的server非常重要，需要配置为之前提到的域名或别名。


到目前为止，我们已经成功运行了一个带有签名证书和用户名/密码的Docker Registry2.0了。

##五、如何进行测试才能说明已经搭建成功

###1.验证登录


###2.在客户端配置证书


###3.推送/拉取测试



##六、遇到的问题及解决办法


##七、致谢

##八、参考文档




























