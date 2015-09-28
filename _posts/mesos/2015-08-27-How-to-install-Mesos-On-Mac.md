---
layout: article
title:  "在MacOS上安装Mesos"
categories: mesos
disqus: true
image:
    teaser: teaser/mesos.jpg
---

{% highlight bash %}
{% raw %}
本文记录了在MacOs上安装Mesos的具体过程，主要参考资料来源于Apache官网..
{% endraw %}
{% endhighlight %} 

---


## Mesos是什么"玩意儿"


官网原文，猛戳[这里](http://mesos.apache.org/gettingstarted/).

### A distributed systems kernel

{% highlight bash %}
{% raw %}
Mesos is built using the same principles as the Linux kernel, only at a different level of abstraction.
The Mesos kernel runs on every machine and provides applications (e.g., Hadoop, Spark, Kafka,
Elastic Search)with API’s for resource management and scheduling across entire datacenter and 
cloud environments.
{% endraw %}
{% endhighlight %}

简单翻译下就是说：Mesos是在不同抽象层次上基于类Linux核心机制构建的分布式系统框架，Mesos核心运行在每一个机器上并可以通过跨数据中心和云环境提供基于资源管理与调度接口的应用，例如：Hadoop,Spark,Kafka,Elastic Search。


### Project Features

接下来我们看看Mesos具有哪些特点：
{% highlight bash %}
{% raw %}
Scalability to 10,000s of nodes
Fault-tolerant replicated master and slaves using ZooKeeper
Support for Docker containers
Native isolation between tasks with Linux Containers
Multi-resource scheduling (memory, CPU, disk, and ports)
Java, Python and C++ APIs for developing new parallel applications
Web UI for viewing cluster state
{% endraw %}
{% endhighlight %}

我们可以看到，Mesos具有如下特点：

- 可提供上万规模的运行节点
- 采用ZooKeeper作为服务发现并实现主从式容错和扩容
- 支持Docker容器
- 任务与Linux容器是天然隔离的
- 基于内存、CPU、硬盘、端口的多资源调度
- 为开发类似新应用提供了Java、Python及C++接口
- 提供web端集群状态查看视图

---

## 下载Mesos

官网推荐下载最新版本程序，笔者使用的是0.22.1版本,Mesos运行在Linux(64bit)或MacOS(64bit)

{% highlight bash %}
{% raw %}
普通用户
wget http://www.apache.org/dist/mesos/0.22.1/mesos-0.22.1.tar.gz
tar -zxf mesos-0.22.1.tar.gz

高级用户
git clone https://git-wip-us.apache.org/repos/asf/mesos.git
{% endraw %}
{% endhighlight %}


## 安装

下面是针对MacOs Yosemeit的安装指导，若是不同系统，请通过安装不同包文件

{% highlight bash %}
{% raw %}

install Command Line Tools.
$ xcode-select --install

# Install Homebrew.
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

# Install libraries.
$ brew install autoconf automake libtool subversion maven
{% endraw %}
{% endhighlight %}

其实有些工具已经安装过的，就不需要安装了，比如maven等

---

## 构建Mesos

{% highlight bash %}
{% raw %}
# Change working directory.
$ cd mesos

# Bootstrap (Only required if building from git repository).
$ ./bootstrap


# Configure and build.
$ mkdir build

$ cd build
$ ../configure
$ make

{% endraw %}
{% endhighlight %}

若想快速构建而减少日志输出，可以在make时增加参数-j <number of cores> V=0


{% highlight bash %}
{% raw %}
# Run test suite.
$ make check

# Install (Optional).
$ make install
{% endraw %}
{% endhighlight %}


## 运行示例

{% highlight bash %}
{% raw %}
# Change into build directory.
$ cd build

# Start mesos master (Ensure work directory exists and has proper permissions).
$ ./bin/mesos-master.sh --ip=127.0.0.1 --work_dir=/var/lib/mesos

# Start mesos slave.
$ ./bin/mesos-slave.sh --master=127.0.0.1:5050

# Visit the mesos web page.
$ http://127.0.0.1:5050


# Run C++ framework (Exits after successfully running some tasks.).
$ ./src/test-framework --master=127.0.0.1:5050

# Run Java framework (Exits after successfully running some tasks.).
$ ./src/examples/java/test-framework 127.0.0.1:5050

# Run Python framework (Exits after successfully running some tasks.).
$ ./src/examples/python/test-framework 127.0.0.1:5050

{% endraw %}
{% endhighlight %}


## 后记

在实践过程中，还需要将/var/lib/mesos的权限赋予当前用户，否则会出现“/var/lib/mesos/replicated_log/LOCK: Permission denied Failed to recover the log”，修改其权限的方式为:

sudo chown `whoami` /var/lib/mesos




























