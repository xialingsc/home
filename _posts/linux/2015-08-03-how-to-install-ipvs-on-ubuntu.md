---
layout: article
title:  "LVS实践系列(3)-在LoadBalancer的Ubuntu12.04下安装ipvs"
categories: linux
disqus: true
image:
    teaser: teaser/ipvs.jpg
---

{% highlight bash %}
{% raw %}
这篇短文主要描述在LVS技术实践过程中的ubuntu环境下描述如何安装ipvsadm,以及遇到的问题以及解决方式
{% endraw %}
{% endhighlight %} 

---


LVS是在Linux系统基础上建立虚拟服务器，实现真实服务器节点之间的负载均衡技术。它是基于内核实现的，2.6.X内核默认集成了LVS模块。笔者使用的Ubuntu12.04也已经含有该内核。

##ipvs检查

{% highlight bash %}
{% raw %}
查看内核版本：
sudo cat /proc/version
查看系统版本：
sudo cat /etc/issue
检验是否安装ipvs内核方法
modprobe -l grep ipvs
若出现：
kernel/net/netfilter/xt_ipvs.ko
kernel/net/netfilter/ipvs/ip_vs.ko
kernel/net/netfilter/ipvs/ip_vs_rr.ko
kernel/net/netfilter/ipvs/ip_vs_wrr.ko
kernel/net/netfilter/ipvs/ip_vs_lc.ko
kernel/net/netfilter/ipvs/ip_vs_wlc.ko
kernel/net/netfilter/ipvs/ip_vs_lblc.ko
kernel/net/netfilter/ipvs/ip_vs_lblcr.ko
kernel/net/netfilter/ipvs/ip_vs_dh.ko
kernel/net/netfilter/ipvs/ip_vs_sh.ko
kernel/net/netfilter/ipvs/ip_vs_sed.ko
kernel/net/netfilter/ipvs/ip_vs_nq.ko
kernel/net/netfilter/ipvs/ip_vs_ftp.ko
kernel/net/netfilter/ipvs/ip_vs_pe_sip.ko
则表明系统已经集成该内核
{% endraw %}
{% endhighlight %}


##安装ipvsadm

{% highlight bash %}
{% raw %}
sudo apt-get install ipvsadm
安装后，通过 sudo ipvsadm 查看是否安装成功。
若出现：
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
则表示安装成功。
注意：若只执行ipvsadm,可能出现：
Can not initialize ipvs: No space left on device
Are you sure that IP Virtual Server is built in the kernel or as module?
原因是权限不够

ipvsadm 配置

可通过下面语句配置ipvsadm是否随机启动，以none/master/backup/both四种的哪一种方式

sudo dpkg-reconfigure ipvsadm


{% endraw %}
{% endhighlight %}

##安装keepalived

若不动态监测真实服务器的心跳，不需执行此步。若需安装，则可按如下步骤进行。

这里主要针对Ubuntu系统,redhat 和 Centos因系统区别,网上资料很多,此处不再叙述。

###1.源码安装keepalived之前需要安装几个程序库

{% highlight bash %}
{% raw %}
apt-get install libssl-dev
apt-get install openssl
apt-get install libpopt-dev
{%endraw %}
{% endhighlight %}

###2.源码安装keepalived

{% highlight bash %}
{% raw %}
下载：从www.keepalived.org/download.html下载keepalived for Linux version 1.2.13
解压 sudo tar -zxvf keepalived-1.2.13.tar.gz
进入keepalived-1.2.13,指定keepalived-1.2.13安装路径
cd keepalived-1.2.13
./configure -prefix=/usr/local/keepalived
编译 make
安装 make install
拷贝并配置keepalived为系统服务
cp /usr/local/keepalived/etc/rc.d/init.d/keepalived /etc/init.d/
然后将keepalived中/etc/rc.d/init.d/functions改为/lib/lsb/init-functions,将/etc/sysconfig/keepalived
改为/usr/local/keepalived/etc/sysconfig/keepalived，将daemon keepalived ${KEEPALIVED_OPTIONS}
改为daemon keepalived start。
mkdir /etc/keepalived
cp /usr/local/keepalived/etc/keepalived/keepalived.conf /etc/keepalived/
cp /usr/local/keepalived/sbin/keepalived /usr/sbin/
{% endraw %}
{% endhighlight %}

###3.安装daemon

`apt-get install daemon`

###4.创建目录

`mkdir -p /var/lock/subsys`

###5.设置为系统服务

`update-rc.d keepalived defaults`

###6.系统启动或停止

`service keepalived start|stop`


##相关命令

###1.ipvsadm相关命令

{% highlight bash %}
{% raw %}
ipvsadm可以定义一个集群服务,定义realserver，同时对集群进行查看。
a.定义集群服务
添加或修改集群服务：ipvsadm -A|E -t|u|f VIP:port -s 调度算法
    -A 添加一个新的集群服务
    -E 修改一个服务
    -t 基于tcp的集群服务
    -u 基于udp的集群服务
    -f 基于防火墙标记的集群服务
    -s 指定调度算法 
    -p 设定持久连接时间
    -C 清空规则
    -R 重新载入规则
    -S 保存规则
    
删除一个集群服务：  ipvsadm -D -t|u|f VIP:port

b.定义realserver
添加或者修改REALSERVER:ipvsadm -a|e -t|u|f VIP:port -r REALSERVER[:port] -g|-i|-m [-w 权重]
    -g LVS-DR直接路由模型
    -i LVS-TUN隧道模型
    -m LVS-NAT模型
删除一个REALSERVER:    ipvsadm -d -t|u|f VIP:port -r REALSERVER[:port]

c.集群查看

ipvsadm -L/l -n 查看
ipvsadm -lcn 查看持久连接状态
{% endraw %}
{% endhighlight %}


###2.其他相关命令

{% highlight bash %}
{% raw %}
添加/删除默认网关sudo route add/delete default gw 192.168.1.XXX
增加/删除网关sudo route add/del 192.168.1.XXX
增加子网 route add -net 192.168.1.0 netmask 255.255.255.0 dev eth0
增加子网 route delete -net 192.168.1.0 netmask 255.255.255.0  eth0
查看路由 sudo route -n
查看所有ip  sudo ip a
解除绑定sudo ip -f inet addr delete 192.168.1.200/32 dev eth0
跟踪访问 sudo traceroute
8099端口监控 sudo tcpdum port 8099
{% endraw %}
{% endhighlight %}






