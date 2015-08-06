---
layout: article
title:  "LVS实践系列(2)-在ubuntu系统实践lvs"
categories: linux
disqus: true
image:
    teaser: teaser/apache.jpg
---

{% highlight bash %}
{% raw %}
这篇短文主要描述在ubuntu环境下对lvs三种场景(VS/DR、VS/TUN、VS/NAT)进行实践的具体操作。
{% endraw %}
{% endhighlight %}

---

##同网段场景实践

实验环境及场景：各台服务器均通过一个路由器物理连接。

|| 机器 || 真实IP || 虚拟IP || 操作系统 || 内核版本||
|| client || 192.168.1.66 || ||redhat || Linux version 2.6.32-71.e16.i686 || 
|| load balancer || 192.168.1.179 ||192.168.1.200 ||ubuntu 12.04.3 LTS || Linux version 3.8.0-42-generic || 
|| real server1 || 192.168.1.67 || ||ubuntu 12.04.2 TLS || Linux version 3.8.0-44-generic || 
|| real server2 || 192.168.1.68 || ||ubuntu 12.04.2 TLS || Linux version 3.2.0-42-generic || 

### VS/DR 场景

测试曾按未使用keepalived和使用keepalived两种进行，这里只贴出了使用keepalived方式，因为更接近生产场景。

首先仍需在realserver(即192.168.1.67,192.168.1.68)上仍执行 dr-realserver.sh,代码同上.

其次，在确保keepalived安装成功的前提下，修改/etc/keepalived/keepalived.conf如下，然后通过

service keepalived start启动。

如果你还没成功安装，或者通过找不到该路径下的文件，请看keepalived的安装过程。

{% highlight bash %}
{% raw %}
! Configuration File for keepalived
global_defs {
    notification_email {                                                                                             
        test@localhost.com
    }
    notification_email_from localhost@localhost.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id LVS_DEVEL   #LVS_DEVEL可以修改成需要的编号
 }
 
 vrrp_instance VI_1 {
    state MASTER #MASTER-主机,BACKUP-备机
    interface eth0 #绑定到eth0网卡
    virtual_router_id 51 #VRID标记(0~255),主备保持相同
    priority 100    #优先级，MASTER要高于BACKUP的优先级(至少50)
    advert_int 1    #检查间隔时间，默认1秒
    authentication {
        auth_type PASS  #指定要使用哪一种认证，目前有PASS和AH两种
        auth_pass 1111  #指定要使用的密码字符串
    }

    virtual_ipaddress {
        192.168.1.200 dev eth0 label eth0:vip  #配置vip地址
     }
 }
 
 virtual_server 192.168.1.200 8099 {
    delay_loop 6
    lb_algo rr  #调度算法，目前为rr 
    lb_kind DR #包转发模式为DR
    nat_mask 255.255.255.0
    #persistence_timeout 50 #长时间超时设置50妙，此处屏蔽，主要是用于验证负责均衡效果
    protocol TCP    #使用TCP协议检查Realserver的状态
    real_server 192.168.1.67 8099 { #定义真实服务器
       weight 1    #权重为1
       TCP_CHECK {
          connect_timeout 3   #连接远程真实服务器超时时间(秒)
          nb_get_retry 3  #最大重试次数
          delay_before_retry 3    #连续两次重试的延迟时间(秒)
          connect_port 8099
       }
    }
 
    real_server 192.168.1.68 8099 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 8099
         }
    }
 }  
 最后，登陆client(即192.168.1.66),通过curl http://192.168.1.200:8099 可以看到交替67或68的响应交替出现，恭喜你又成功了。
{% endraw %}
{% endhighlight %}

keepalived优点

倘若我们停掉一台real server的http服务，通过ipvsadm -ln观察，load balancer中马上就能反映出该台服务器无法提供服务了，相关的原理可以参照《Keepalived权威指南中文.pdf》.如果不使用keepalived,则在客户端访问就会出现访问正常和访问错误交替出现。


###VS/TUN场景

该场景只进行了未使用keepalived的场景。

{% highlight bash %}
{% raw %}
lserver(即192.168.1.67,192.168.1.68)上执行ip-tun-realserver.sh,代码如下：
#!/bin/bash                                                                                                         
/sbin/ifconfig tunl0 192.168.1.200 broadcast 192.168.1.200 netmask 255.255.255.255 up
/sbin/route add -host 192.168.1.200 dev tunl0
echo "0" >/proc/sys/net/ipv4/ip_forward
echo "1" >/proc/sys/net/ipv4/conf/tunl0/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/tunl0/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/tunl0/rp_filter
echo "2" >/proc/sys/net/ipv4/conf/all/rp_filter
其次，在load balancer上执行ip-tun-vpserver.sh,代码如下：
#!/bin/bash                                                                                                         
vip=192.168.1.200
rs1=192.168.1.67
rs2=192.168.1.68
/sbin/ifconfig eth0:1 $vip netmask 255.255.255.255 broadcast $vip up
/sbin/route add -host $vip dev eth0:1
/bin/netstat -rn 
#采用加权最少链接算法通过设定不同权值进行实验
ipvsadm -A -t $vip:8099 -s wlc 
ipvsadm -a -t $vip:8099 -r $rs1:8099 -i -w 2 #w表示权值
ipvsadm -a -t $vip:8099 -r $rs2:8099 -i -w 1
echo "0" >/proc/sys/net/ipv4/ip_forward
echo "1" >/proc/sys/net/ipv4/conf/all/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/default/send_redirects
echo "1" >/proc/sys/net/ipv4/conf/eth0/send_redirects
/sbin/ipvsadm -ln
最后，登陆client(即192.168.1.66),通过curl http://192.168.1.200:8099 可以看到交替67或68的响应交替出现，恭喜你成功了。
{% endraw %}
{% endhighlight %}


## 不同网段的VS/NAT场景实验


实验环境及场景:各台服务器均物理连接在同一个路由器，但load balancer具有两个网卡，一个网卡连接外网，一个网卡连接内网。

该场景只进行了使用keepalived的实验.

|| 机器 || 真实IP || 虚拟IP || 操作系统 || 内核版本||
|| client ||172.20.10.* || || || Linux version 2.6.32-71.e16.i686 || 
|| load balancer || 内网：192.168.1.179 || ||ubuntu 12.04.3 LTS || Linux version 3.8.0-42-generic || 
||  || 172.20.10.2 ||172.20.10.200 ||ubuntu 12.04.2 TLS || Linux version 3.8.0-44-generic || 
|| real server1 || 192.168.1.67 || ||ubuntu  || Linux version  || 

{% highlight bash %}
{% raw %}
首先，在realserver(即192.168.1.67,192.168.1.68)上执行如下语句：
sudo route add default gw 192.168.1.179
sudo netstat -rn
其次，在确保keepalived安装成功的前提下，修改/etc/keepalived/keepalived.conf如下，然后通过service keepalived start启动
! Configuration File for keepalived
global_defs {
    notification_email {                                                                                             
        test@localhost.com
    }
    notification_email_from localhost@localhost.com
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id LVS_DEVEL   #LVS_DEVEL可以修改成需要的编号
}

vrrp_instance VI_1 {
    state MASTER #MASTER-主机,BACKUP-备机
    interface eth0 #绑定到eth0网卡
    virtual_router_id 51 #VRID标记(0~255),主备保持相同
    priority 100    #优先级，MASTER要高于BACKUP的优先级(至少50)
    advert_int 1    #检查间隔时间，默认1秒
    authentication {
        auth_type PASS  #指定要使用哪一种认证，目前有PASS和AH两种
        auth_pass 1111  #指定要使用的密码字符串
    }

    virtual_ipaddress {
        172.20.10.200
    }
}

virtual_server 172.20.10.200 8099 {
    delay_loop 6
    lb_algo rr  #调度算法，目前为rr,可以换成别的策略,例如 wlc,lblc等 
    lb_kind NAT #包转发模式为NAT
    #persistence_timeout 50 #长时间超时设置50妙，此处屏蔽，主要是用于验证负责均衡效果
    protocol TCP    #使用TCP协议检查Realserver的状态
    real_server 192.168.1.67 8099 { #定义真实服务器
        weight 1    #权重为1
        TCP_CHECK {
            connect_timeout 3   #连接远程真实服务器超时时间(秒)
            nb_get_retry 3  #最大重试次数
            delay_before_retry 3    #连续两次重试的延迟时间(秒)
            connect_port 8099
        }
    }

    real_server 192.168.1.68 8099 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
            connect_port 8099
        }
    }
}    
最后，找一台172.10.20.200同网关的机器，访问http://172.10.20.200:8099可以看到交替67或68的响应交替出现，恭喜你成功了。

{% endraw %}
{% endhighlight %}

