---                                                                                                                                                                                                       
layout: article
title:  "浅谈基于Fabric实现项目自动部署"
categories: python
disqus: true
image:
    teaser: teaser/chilun.jpg
---

{% highlight bash %}
{% raw %}
本文从运行环境、python相关知识、fabric参考资料、安装fabric、fabric入门及接口等方面进行了简要介绍，
在此基础上介绍了利用fabric实现web项目自动部署的思路及详细过程。
{% endraw %}
{% endhighlight %} 

---

##背景

在向服务器部署项目过程中，实施人员最大的烦劳莫过于修改过多的配置文件了。 假设你一时疏忽忘记修改或改错某些配置，还不得不手动执行停止服务、修改文件、启动服务等操作。 这个过程是一次，也可能需要重复多次。 你或许需要同时部署多台服务器，一样的操作执行多遍，你是不是会觉得又忙，又无趣，还高度紧张无比抓狂？ 今天向大家介绍的Fabric，它可以让上述工作变得轻松自如，让我们心情大好。 这篇文章主要目的是讲清楚怎么用Fabric实现项目自动部署，对python、shell相关知识不展开描述， 也不对Fabric所有功能进行介绍。

---

##运行环境

系统:Ubuntu 12.04 
(Mac OS 可以使用gsed)

Shell:z-shell

Web容器：tomcat

---

##相关知识

1.python相关知识

由于Fabric是python库，所以我们有必要先了解python的相关语法。 在学习python时，笔者参看的是《简明Python教程》，它比较适合初学python的同学。 不仅从安装、基本概念、运算符、控制流、函数、模块、数据结构、对象、输入输出、 异常、标准库等多个方面进行了介绍，而且有部分练习实例。 另一个python入门指南网址：http://www.pythontab.com/html/pythonshouce27/ 感兴趣的同学还可以看看：https://wiki.python.org/moin/BeginnersGuideChinese

2.简单shell知识

需要了解sed删除、替换内容

---

##什么是Fabric

Fabric是一个python（2.5-2.7）库和命令行工具，主要在于简化使用ssh完成应用程序部署或系统管理任务。 简单说它的特色是执行本地脚本，就可以完成远程服务器部署。 官方文档： http://fabric.readthedocs.org/en/1.8/tutorial.html

---

##安装Fabric

安装Fabric前提是先装了python的相关环境，python安装可以参看《简明Python教程》。 Fabric有三种安装方式， pip install fabric sudo easy_install fabric sudo apt-get install fabric 笔者采用的是最后一种。

---

##Fabric入门

参看网址： http://wklken.me/posts/2013/03/25/python-tool-fabric.html http://www.pythonforbeginners.com/systems-programming/how-to-use-fabric-in-python/

---

##Fabric接口

{% highlight bash %}
{% raw %}
Fabric 在fabric.api中提供了简单而又强大的命令集合。 你可以简单的这样使用：
local #执行本地命令
run #以用户级别去某具体主机执行远程脚本
sudo #在远程服务器上sudo a command
put #上传本地文件至远端服务器指定目录
get #从远端服务器下载指定文件
prompt #接受用户输入并返回(同raw_input)
reboot #重启远程系统
{% endraw %}
{% endhighlight %}

---

###基于Fabric实现项目自动部署

1.想法

在部署一个项目时通常会按如下几步进行：登录服务器、停止服务、删除原项目(若有)、 部署新项目、修改相关配置文件、启动服务。基于这个想法，可以采用Fabric分步骤编写实现脚本。

2.实现

{% highlight bash %}
{% raw %}
(1)引入Fabric.api
#在文件头增加 import fabric from fabric.api import *
(2)服务器主机信息
#服务器主机IP及密码，若不写密码，执行时需输入 env.hosts = ['gap@10.11.44.197'] env.password = '123456'#示例
(3)停止服务
#定义停止服务函数
def shutdown():
    run('cd ~')
    result = run('ps -ef |grep catalina.home=/home/gap/app/apache-tomcat-5\.5\.14 |grep -v grep |awk \'{print $2 }\'')
    if result != '':
        run('kill -9 %s'%result)
    #注:run('/home/gap/app/apache-tomcat-5.5.14/bin/shutdown.sh')不一定能停止服务， 主要原因是shutdown.sh只会关闭startup.sh启动的线程，而无法关闭部署应用的后台线程
(4)删除原项目
#定义删除原项目函数 
def rmfile(): 
    run('rm -rf /home/gap/app/apache-tomcat-5\.5\.14/webapps/gap41') 
    print 'delete file successful'
(5)部署新项目
#定义复制文件及目录函数 
def copyfile(): 
    local_directory = '~/GAP41'#根据实际情况修改 with lcd(local_directory): 
    local('scp -r gap41 gap@10\.11\.44\.197:/home/gap/app/ apache-tomcat-5\.5\.14/webapps/') #注：此处可以优化为先在本地打成tar,然后复制过去，再解压
(6)修改相关配置文件
配置文件修改清单：
   - db相关
     db.xml
     applicationContext.xml
     .......
     定义修改配置文件函数
     #这部分内容太多，举一个修改oscache.properties实例即可
     def modifyfile():
        ...... 
        # modify oscache.properties 
        #3.oscache.properties 
        fabric.contrib.files.sed('/home/gap/app/apache-tomcat-5.5.14 /webapps/gap41/WEB-INF/classes/oscache\.properties', before='cache\.path=c:\\\\\\\\myapp\\\\\\\\cache', after='cache\.path=/tmp/gap41/cache', use_sudo=True,backup='.bak')
        print 'modify oscache.properties successful' ......
(7)启动服务
#定义启动服务函数 def startup(): run('cd ~') run('set -m; /home/gap/app/apache-tomcat-5.5.14 /bin/startup\.sh') #注：单独使用/home/gap/app/apache-tomcat-5.5.14/bin/startup\.sh'无法启动， 需要加set -m; 才能让tomcat在后台运行。 如果使用/home/gap/app/apache-tomcat-5.5.14/bin/catalina\.sh run', 则在执行脚本的当前窗口会出现后台日志，这种情况 应该也不是我们想看到的结果。
 
如果想增加执行环境的变量，则可以引入prefix,如下所示
from fabric.context_managers import (prefix)
def startup():                                                                                                  
    '''startup tomcat '''
    with prefix('export LANG=en_US.UTF-8'):
    run('cd ~')
    run('set -m; %s/bin/startup\.sh'%env.server_tomcat_path)

{% endraw %}
{% endhighlight %}






