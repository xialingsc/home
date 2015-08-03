---
layout: article
title:  "LVS是什么鬼"
categories: linux
disqus: true
image:
    teaser: teaser/lvs-architecture.jpg
---


>本文根据官方文档简要介绍了LVS技术的概念、内容、特点及相关调度算法，为进一步在实际场景下实践该技术作准备。


##背景

随着网络时代的到来，越来越多的应用都服务化，也需要越来越多的服务器进行支撑，网络带宽技术发展迅猛，应用系统的瓶劲出现在服务端的情况越来越多，如何能更好的满足大应用的骤然增加，就要求我们的有性价比高、可伸缩性强、高可用性、易于管理透明的服务器集群方案，这就是我学习LVS技术的初衷。

##什么是LVS技术

LVS 是 Linux Virtual Server的简称，是基于一组真实服务器依靠负载均衡运行在Linux操作系统上的可扩展高易用的虚拟服务器。主要基于IP层和基于内容请求分发的负载平衡调度解决办法，并在Linux内容中实现这些方法，将一组服务器构成一个实现可伸缩的、高可用网络服务的虚拟服务器。 是国内最早出现的自由软件项目之一。

##LVS内容

当前LVS框架包含三种IP负载均衡技术的IP虚拟服务器软件IPVS、基于内容请求分发的内核Layer-7交换机KTCPVS和集群管理软件。

1.IP虚拟服务器软件IPVS

IPVS包括
(1)通过网络地址转换(Network Address Translation)将一组服务器构成一个高性能的、高可用的虚拟服务器（即VS/NAT技术）,例如Cisco的LocalDirector、F5的Big/IP和Alteon的ACEDirector

其原理：通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分派给后端的真实服务器；真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回给客户，完成整个负载调度过程。

(2)IP隧道实现虚拟服务器方法(VS/TUN)

为了解决VS/NAT技术调度器处理能力瓶颈问题，调度器把请求报文通过IP隧道转发至真实服务器，而真实服务器将响应直接返回给客户。最大特点就是：调度器只处理请求报文。

(3)通过直接路由实现虚拟服务器的方法(VS/DR)

通过改写请求报文的MAC 地址，将请求发送到真实服务器，而真实服务器将响应直接返回给客户。该方法优点:无IP隧道开销，集群中真实服务器不必须支持IP隧道协议要求，但要求调度器与真实服务器都有一块网卡连在同一物理网段上。

{% highlight bash %}
{% raw %}
IPVS 八种负载调度算法：
(1)轮叫(Round Robin)：顺序轮流分配，均等对待
(2)加权轮叫(Weighted Round Robin)：询问真实服务器负载情况，动态调整权值，能者多劳
(3)最少链接(Least Connections)：给链接数最小的
(4)加权最少链接(Weighted Least Connections)：
(5)基于局部性的最少链接(Locality-Based Least Connections)：针对目标IP地址的负载均衡，主要用于Cache集群系统。该算法根据请求的目标IP地址找出该目标IP最近使用的服务器，若该服务器是可用的且没有超载，将请求发送给它；若服务器不存在，或者已超载且有服务器处于一半的工作负载，则用“最少链接”的原则
(6)带复制的基于局部性最少链接(Locality-Based Least Connections with Replication):与LBLC算法的不同之处是它要维护从一个目标IP地址到一组服务器的映射
(7)目标地址散列(Destination Hashing):根据请求的目标IP地址，作为散列键从静态分配的散列表找出对应服务器
(8)源地址散列(Source Hashing):根据请求的源IP地址，作为散列键从静态分配的散列表找出对应的服务器
{% endraw %}
{% endhighlight %}

2.基于内容请求分发的内核Layer-7交换机KTCPVS

目的：避免用户空间与核心空间的切换和内容复制的开销,基于内容选择服务器。在Linux操作系统内核中，实现Layer-7交换，称为KTCPVS(Kernel TCP Virtual Server)

3.集群管理软件


##LVS特点

{% highlight bash %}
{% raw %}
1.功能
实现三种IP负载均衡技术和八种连接调度算法的IPVS软件，在IPVS内部，采用了高效的Hash函数和垃圾回收机制，能正确处理所调度报文相关的ICMP消息。虚拟服务的设置数目没有限制，每个虚拟服务有自己的服务器。
2.适用性
可运行任何TCP/IP的操作系统，负载均衡绝大多数的TCP和UDP协议(TCP HTTP，FTP，PROXY，SMTP，POP3，IMAP4，DNS，LDAP，HTTPS，SSMTP等 
UDP DNS，NTP，ICP，视频、音频流播放协议等 )
3.性能
据说可支持几百万个并发链接。配置100M网卡，采用VS/TUN或VS/DR调度技术，集群系统吞吐量可高达1Gbits/s
4.可靠性
5.软件许可证：GPL(GNU Public License)方式发行
{% endraw %}
{% endhighlight %}

##参考文档

[LVS英文站点](http://www.linuxvirtualserver.org/)
[LVS手册](http://zh.linuxvirtualserver.org/handbooks)
