---
layout: article
title:  "简析Beego框架对Mysql数据库的支持及在项目中的应用"
categories: go
disqus: true
image:
    teaser: teaser/beego-mysql.jpg
---

{% highlight bash %}
{% raw %}
本文介绍了Beego框架对Mysql关系数据库的支持方法，并将在项目中遇到的一些问题及解决办法进行简单总结。
{% endraw %}
{% endhighlight %} 

---


##Beego是个啥

Beego 是一个快速开发Go应用的HTTP框架，它可以用来快速开发API、Web 及后端服务等各种应用，是一个 RESTful 的框架，主要设计灵感来源于 tornado、sinatra 和 flask 这三个框架，但是结合了 Go 本身的一些特性（interface、struct 嵌入等）而设计的一个框架，详见[Beego官网](http://beego.me/docs/intro/)。

---

##Beego架构

不废话，请右转到[Beego官网](http://beego.me/docs/intro/)

---

##Beego支持的数据库

笔者使用的Beego版本为1.4.3,官方介绍已支持的数据库有[Mysql](https://github.com/go-sql-driver/mysql)、[PostgreSQL](https://github.com/lib/pq)、[Sqlite3](https://github.com/mattn/go-sqlite3),这里结合官网描述及在项目中使用Mysql的方式进行简单小结。

---

##安装Beego ORM

go get github.com/astaxie/beego/orm

注：这里要求go语言的运行环境已经安装成功。

待续





























