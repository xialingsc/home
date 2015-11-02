---
layout: article
title:  "在Mesos上运行各种服务系列之Mac单机基本框架搭建"
categories: mesos
disqus: true
image:
    teaser: teaser/mesos-1.jpg
---

{% highlight bash %}
{% raw %}
本文主要描述可以在Mesos上运行的各种服务，它是区别于运行数据处理框架的另一种运用Mesos的形式。
{% endraw %}
{% endhighlight %} 

---


## 主要内容

各种Service是在传统企业业务体系中扮演着十分重要的角色，本文主要将介绍运行在mesos上的各种服务及不同调度框架。

## 各种服务

传统企业将很多大量长期运行的服务作为日常业务运转的一部分，比如：Web服务，数据管道服务等。每一个服务是一个自包含、被独立部署和管理的实用功能单元。面向服务架构(SOA)和微服务架构都鼓励让你将复杂的服务变为松散的应用组合。当今越来越多的应用都是由众多的微服务组成，正是这样带来了很多的优点，像代码复用，轻松扩容，单点失效，多平台灵活部署等。

上面描述了那么多，跟Mesos有几个关系？

别急呀，马上就有关系啦。我们知道Mesos是批量处理，实时，和其他处理的框架，它们的工作通常是短暂的。企业基础设施运行以长期运行性的，具有比数据处理框架不同的要求很多应用和服务。这些长期运行的服务不仅是重要的业务，但也消耗了基础设施资源的很大部分。因此，将服务运行在Mesos上的能力是非常重要的。

为了可扩展的运行服务，要求基础设施满足如下一些要求：

- 部署服务时依赖服务或相关约束是需要关注的。
- 确保该服务的所有的依赖关系在服务器启动之前都已就绪。
- 当多个实例服务在运行时服务发现和负载均衡变得越来越重要。服务发现回答了提供服务的示例运行在什么地方？负载均衡决定哪个服务在请求时应该给予响应。
- 一旦服务部署后，对于该服务的健康检查也是很重要的，健康监控信息还能被进一步利用，比如服务伸缩及失败重启等。
- 高可用性

如果我们把mesos作为数据中心的核心，服务调度是大规模基础设施中最重要的部分。尽管早前依赖与mesos的框架几乎都面向数据处理，但是运行服务对于企业应用场景而言也是必须的。很明显，很多应用都充分利用mesos提供的基础设施进行写入或重写，但也有很多应用只需要启动执行结点和一旦挂了就再次重启即可。调度框架就可以让你做到这一点。服务调度器也称为元框架或元调度器，因为它们为运行其上的应用程序充当媒介。服务调度主要负责使其原本服务失败后手工完成的复杂性可以变为服务，让它们重新开始并根据需要扩展它们。

## 调度框架

### Marathon

Marathon是一个在Mesos上运行长服务的框架。那些服务具有高可用性，即意味着marathon应该能在一些服务示例失败时另外启动相应服务和进行扩展。marathon在数据中心或集群中表现为init.d,这意味着它能够确定运行在其上的服务是否总在运行。marathon被设计来运行任务并保证它们在运行。Marathon能运行类似于hadoop的其它框架。
Marathon有非常全面的Rest API去管理服务生命周期，以及提供交互的Java,Scala,python,ruby。


安装zookeeper
{% highlight bash %}
{% raw %}
1.从http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.6/下载
2.tar zxf zookeeper*.gz
3.cd zookeeper*
4.mv conf/zoo_sample.cfg conf/zoo.cfg
#启动redis服务
5. ./bin/zkServer.sh start
#测试是否运行成功
6. ./bin/zkCli.sh -server 127.0.0.1:2181
{% endraw %}
{% endhighlight %}


安装Marathon
{% highlight bash %}
{% raw %}
1.install Mesos
2.wget http://downloads.mesosphere.io/marathon/v0.8.0/marathon-0.8.0.tgz
3.tar xzf marathon-*.tgz
4.rm marathon-*.tgz
5.cd marathon-*
6.ubuntu@master:~/marathon $./bin/start –-master zk://127.0.0.1:2181/mesos –-zk zk://localhost:2181/marathon
7.通过webui的8080端口进行访问
{% endraw %}
{% endhighlight %}

marathon提供了很多选项适用于你的需求。完整列表可以从这里获得[http://mesosphere. github.io/marathon/docs/command-line-flags.html](http://mesosphere. github.io/marathon/docs/command-line-flags.html)


有没有太冗长的感觉？

简单小结下：

1）安装Mesos,安装zookeeper,安装marathon

2）分别启动之(注意修改相关路径及参数)
{% highlight bash %}
{% raw %}
cd ~/Software/Mesos/mesos-0.22.1/build/bin
sudo ./mesos-master.sh --ip=127.0.0.1 --work_dir=/var/lib/mesos
cd ~/Software/Mesos/zookeeper-3.4.6/bin
./zkServer.sh start
cd ~/Software/Mesos/marathon-0.8.0/bin/
./start --master  zk://127.0.0.1:2181/mesos --zk zk://localhost:2181/marathon
然后在浏览器中输入http://127.0.0.1:5050后查mesos相关信息
在新开的浏览器中输入http://127.0.0.1:8080端口查看marathon相关信息
{% endraw %}
{% endhighlight %}

