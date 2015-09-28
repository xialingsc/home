---
layout: article
title:  "如何利用Docker搭建多结点Mesosphere场景"
categories: mesos
disqus: true
image:
    teaser: teaser/multi-node-mesosphere-docker.jpg
---

{% highlight bash %}
{% raw %}
本文主要记录经过简单几个步骤的执行，实现在多个servers上用Docker搭建Mesos集群的具体过程。
{% endraw %}
{% endhighlight %} 

---


## 准备工作

《道德经》中曾描述说：“道生一，一生二，二生三，三生万物”。老子他老人家的很多东西我是不懂的，但这里的实验场景有点在这个想法上进行，即在多结点的测试场景中，提前准备两个已安装Docker的Server，不再是单一的server了。当然你也可以使用多个Server进行实践，只要在下面每个步骤灵活地进行对应修改即可。

## 开始实践

### 设置IP变量

可以分别在每个server上的环境中定义IP变量，并在后续几步进行引用如下变量即可。为避免{HOST_IP_1}引起Markdown语法冲突，故本文全用真实IP.

{% highlight bash %}
{% raw %}
HOST_IP_1=192.168.1.145
HOST_IP_2=192.168.1.171
{% endraw %}
{% endhighlight %}

### 启动ZooKeepers

在第一个server上执行
{% highlight bash %}
{% raw %}
docker run -d \
--net="host" \
-e SERVER_ID=1 \
-e ADDITIONAL_ZOOKEEPER_1=server.1=${HOST_IP_1}:2888:3888 \
-e ADDITIONAL_ZOOKEEPER_2=server.2=${HOST_IP_2}:2888:3888 \
garland/zookeeper
{% endraw %}
{% endhighlight %}

在第二个Server上执行
{% highlight bash %}
{% raw %}
docker run -d \
--net="host" \
-e SERVER_ID=2 \
-e ADDITIONAL_ZOOKEEPER_1=server.1=${HOST_IP_1}:2888:3888 \
-e ADDITIONAL_ZOOKEEPER_2=server.2=${HOST_IP_2}:2888:3888 \
garland/zookeeper
{% endraw %}
{% endhighlight %}

### 启动Mesos Masters

在第一个Server上执行
{% highlight bash %}
{% raw %}
docker run --net="host" \ 
-p 5050:5050 \
-e "MESOS_HOSTNAME=${HOST_IP_1}" \
-e "MESOS_IP=${HOST_IP_1}" \
-e "MESOS_ZK=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181/mesos" \
-e "MESOS_PORT=5050" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_QUORUM=1" \
-e "MESOS_REGISTRY=in_memory" \
-e "MESOS_WORK_DIR=/var/lib/mesos" \
-d \
garland/mesosphere-docker-mesos-master
{% endraw %}
{% endhighlight %}

在第二个Server上执行
{% highlight bash %}
{% raw %}
docker run --net="host" \
-p 5050:5050 \
-e "MESOS_HOSTNAME=${HOST_IP_2}" \
-e "MESOS_IP=${HOST_IP_2}" \
-e "MESOS_ZK=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181/mesos" \
-e "MESOS_PORT=5050" \
-e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_QUORUM=1" \
-e "MESOS_REGISTRY=in_memory" \
-e "MESOS_WORK_DIR=/var/lib/mesos" \
-d \
garland/mesosphere-docker-mesos-master
{% endraw %}
{% endhighlight %}

### 启动Marathon

这一步可以在不同于host1，2上的安装了Docker的server上运行
{% highlight bash %}
{% raw %}
docker run -d -p 8080:8080 \
garland/mesosphere-docker-marathon --master zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181/mesos --zk zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181/marathon
{% endraw %}
{% endhighlight %}

### 启动Mesos Slaves

在两个Server上均执行
{% highlight bash %}
{% raw %}
docker run -d --entrypoint="mesos-slave" \
-e "MESOS_MASTER=zk://${HOST_IP_1}:2181,${HOST_IP_2}:2181/mesos"  -e "MESOS_LOG_DIR=/var/log/mesos" \
-e "MESOS_LOGGING_LEVEL=INFO" garland/mesosphere-docker-mesos-master:latest
{% endraw %}
{% endhighlight %}

### 其他

接下来的测试内容可看早前写的另一篇文章[如何在Mac上利用Docker部署一个单节点Mesos集群](http://xialingsc.github.io/home//mesos/How-to-deploy-Mesos-By-Docker-On-Mac-/)第六、七步内容。


### 参考文献

原文英文资料在[这里](https://github.com/sekka1/mesosphere-docker#multi-node-setup)
