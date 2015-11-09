---
layout: article
title:  "在Marathon上运行Docker容器"
categories: docker
disqus: true
image:
    teaser: teaser/docker-tomcat.jpg
---

{% highlight bash %}
{% raw %}
本文将描述通过Mesos 0.20.0内嵌对docker的支持如何在Marathon上运行Docker容器，官网原文请戳[这里](https://mesosphere.github.io/marathon/docs/native-docker.html)。
{% endraw %}
{% endhighlight %} 

---


### 配置

前提要求，每个mesos-slave节点需安装Docker 1.0.0及以上版本。

##### 1. 配置 mesos-slave
    
注：下面的所有命令都基于mesos-slave正在作为一个服务(使用mesosphere提供的包)在运行的假定。

(1) 针对指定Docker container更新slave配置

注意：容器化参数的顺序非常重要。在选择容器去运行任务的过程中它说明了被使用时的优先级

{% highlight bash %}
{% raw %}
$ echo 'docker,mesos' > /etc/mesos-slave/containerizers
{% endraw %}
{% endhighlight %}

(2) 针对将docker image拉取到salve可能的延迟时间而设置执行超时时间

{% highlight bash %}
{% raw %}
$ echo '5mins' > /etc/mesos-slave/executor_registration_timeout

{% endraw %}
{% endhighlight %}
(3)重启mesos-slave加载新配置项

##### 配置marathon

1.提前设置salve,增加marathon命令行参数--task_launch_timeout用于执行超时时间，单位为毫秒。

概述

为了使用本地容器支持，在你应用定义json中添加一个容器域。
{% highlight %}
{% raw %}
{
    "container": {
        "type":"DOCKER",
        "docker": {
            "network": HOST",
            "image": "group/image"
        },
        "volumes": [
        {
            "containerPath":/etc/a",
            "hostPath": "/var/data/a",
            "mode": "RO"
        },
        {
            "containerPath": "/etc/b",
            "hostPath": "/var/data/b",
            "mode": "RW"
        }
        ]
    }
}
{% endraw %}
{% endhighlight %}
其中volumes和type是可选的(默认为DOCKER).更多的类型将娓娓道来。

为了方便，mesos sandbox的mount 点可以从环境变量通过$MESOS_SANDBOX来获得。

桥接网络模式(Bridged Networking Mode)

(需要Mesos 0.20.1及Marathon 0.7.1)

桥接网络通过在Docker容器中绑定配置端口可以让程序运行更为简单。Marathon能够在Mesos和Docker中约束的主机端口设定的端口资源间的桥接缝隙。

动态端口映射

下面是应用定义示例：

{% highlight %}
{% raw %}
{
    "id": "bridged-webapp",
    "cmd": "python3 -m http.server 8080",
    "cpus": 0.5,
    "mem": 64.0,
    "instances":2,
    "container": {
        "type":"DOCKER",
        "docker": {
            "image": "python:3",
            "network":"BRIDGE",
            "portMappings": [
                { "containerPort": 8080,"hostPort": 0, "servicePort":9000,"protocol": "tcp"},
                { "containerPort": 161,"hostPort": 0, "servicePort":9000,"protocol": "tcp"},
            ]

        }
    }

}
{% endraw %}
{% endhighlight %}



































