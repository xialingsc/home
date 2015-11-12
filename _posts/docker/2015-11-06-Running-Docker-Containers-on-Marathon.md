---
layout: article
title:  "在Marathon上运行Docker容器"
categories: docker
disqus: true
image:
    teaser: teaser/marathon-docker.jpg
---

{% highlight bash %}
{% raw %}
本文将描述通过Mesos 0.20.0内嵌对docker的支持如何在Marathon上运行Docker容器，官网原文请见底部描述戳.
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

###### 概述

为了使用本地容器支持，在你应用定义json中添加一个容器域。
{% highlight bash %}
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

###### 桥接网络模式(Bridged Networking Mode)

(需要Mesos 0.20.1及Marathon 0.7.1)

桥接网络通过在Docker容器中绑定配置端口可以让程序运行更为简单。Marathon能够在Mesos和Docker中约束的主机端口设定的端口资源间的桥接缝隙。

(1)动态端口映射

下面是应用定义示例：

{% highlight bash %}
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
    },
    "healthChecks": [
        {
            "protocol":"HTTP",
            "portIndex": 0,
            "path": "/",
            "gracePeriodSeconds": 5,
            "intervalSeconds": 20,
            "maxConsecutiveFailures":3
        }
    ]


}
{% endraw %}
{% endhighlight %}

这里"hostPort": 0  保留了在Marathon的传统含义，这是一个从Mesos资源提供范围内的随机端口。每个任务返回的主机端口可以通过REST API和Marathon web UI的任务详情进行查看。"hostPort"是可选的，同时默认为0.

"containerPort" 是指容器里面应用监听端口。

由于v0.9.0:"containerPort" 是可选的同时默认为0.当设置"containerPort":0时，Marathon就分配containerport与分配地hostport为相同值。这对于只公布端口用于P2P与外界进行交流的应用非常有用。如果不设置"containerPort":0,就会错误地公布私有容器端口而这通常是与主机端口不相同的。

"servicePort"对于每一个已定义端口的服务做服务发现而言是一个设置辅助端口的参数。分配的"servicePort"值是不会被Marathon自身使用/理解，但是可以通过基础负载均衡去支持使用。这部分可以看[Service Discovery Load Balancing doc page](https://mesosphere.github.io/marathon/docs/service-discovery-load-balancing).这个参数是可选的，默认为0.与hostPost一样，如果值为0,将会分配一个随机端口。一个servicePort值通过Marathon分配指定后，Marathon将维护其在集群中的唯一性。通过[local_port_min,local_port_max]设定服务端口值默认推荐在10000至20000中进行选择。

"protocol"参数是可选的，默认为"tcp"

(2)静态端口映射

它也可以具体指定非0主机端口。当确定这么做时，应该确保目标端口在资源提供的范围内！mesos-salve默认的端口资源为[31000-32000].也可以设成[8000-9000]

{% highlight bash %}
{% raw %}
--resources="ports(*)[8000-9000,31000,32000]"
{% endraw %}
{% endhighlight %}

可以看Docker如何处理网络部分的网络配置文档[Network configuration](http://docs.docker.com/engine/userguide/networking/)

###### 使用Private Docker Repsitory

这部分可以看私有仓库文档[private registry](https://mesosphere.github.io/marathon/docs/native-docker-private-registry.html)来了解如何用Marathon从一个私有仓库去初始化拉取镜像的。

###### 高级用法

1.强行拉取

Marathon 0.8.2使用了mesos 0.22.0支持的一个选项即在运行每个任务前强行先拉取Docker。

{% highlight bash %}
{% raw %}
{
  "type": "DOCKER",
  "docker": {
    "image": "group/image",
    "forcePullImage":true
  }
}
{% endraw %}
{% endhighlight %}

2.命令vs 参数

从0.7.0开始，在JSON应用中Marathon支持一个args参数。对于相同应用同时提供了cmd和args两个参数时，args是失效的。cmd是早先版本的使用方式(它的值经由Mesos进行封装，例如/bin/sh -c '${app.cmd}'). 而具化了命令的args就允许容器安全使用类似自定义的Docker的ENTRYPOINT参数。例如，可以在Dockerfile中使用ENTRYPOINT如下定义：
{% highlight bash %}
{% raw %}
FORM busybox
MAINTAINER support@mesosphere.io

CMD ["inky"]
ENTRYPOINT ["echo"]
{% endraw %}
{% endhighlight %}

下面这段json 定义将下载"mesosphere/inky"Docker 容器 同时执行 echo hello:
{% highlight bash %}
{% raw %}
{
    "id": "inky",
    "container": {
        "docker": {
            "image": "mesosphere/inky"
         },
         "type":"DOCKER",
         "volumes": []
     },
     "args": ["hello"],
     "cpus": 0.2,
     "mem": 32.0,
     "instances": 1
}

{% endraw %}
{% endhighlight %}

命名参数可以为连续数组，例如argc,argv
{% highlight bash %}
{% raw %}
"args":[
    "--name", "etcd0",
    "--initial-cluster-state","new"
]
{% endraw %}
{% endhighlight %}

###### Privileged Mode 和Docker选项的随意性

从0.7.6版本开始，Marathon对于Docker容器支持两个新的key:privileged和parameters. privileged 标志用来控制是否运行用户以特权模式运行容器。默认为false. parameters 对象是为了通过Mesos containerizer 执行docker run 命令而补充随意的命令行参数。注意，以这种方式传递参数不一定保证在未来得到支持，因为Mesos可能不总是经由CLI 与Docker进行交互。

下面罗列了两个参数的使用方法
{% highlight bash %}
{% raw %}
{
    "id": "privileged-job",
    "container": {
        "docker": {
            "image": "mesosphere/inky",
            "privileged": true,
            "parameters": [
                {"key": "hostname","value": "a.corp.org"},
                {"key": "volumes-from","value": "another-container"},
                {"key": "lxc-conf","value": "..."},

            ]
         },
         "type":"DOCKER",
         "volumes": []
     },
     "args": ["hello"],
     "cpus": 0.2,
     "mem": 32.0,
     "instances": 1
}

{% endraw %}
{% endhighlight %}



##### 参考文档

官网文档:请戳[这里](https://mesosphere.github.io/marathon/docs/native-docker.html)




























