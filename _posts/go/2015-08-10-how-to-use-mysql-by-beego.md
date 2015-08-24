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

##利用orm进行CRUD操作

###引入driver

在使用orm时，需要在import 中引入orm包及mysql包，形如：

{% highlight bash %}
{% raw %}
import (
    "github.com/astaxie/beego“
    "github.com/astaxie/beego/orm"
    _ "github.com/go-sql-driver/mysql"
)
{% endraw %}
{% endhighlight %}

### 设置并注册信息

{% highlight bash %}
{% raw %}
dbUser := Cfg.String("db_user")
dbPwd  := Cfg.String("db_passowrd")
dbHost := Cfg.String("db_host")
dbPort := Cfg.String("db_port")
dbName := Cfg.String("db_name")

maxIdleConn, _ := Cfg.Int("db_max_idle_conn")
maxActiveConn, _ := Cfg.Int("db_max_open_conn")
dbUrl := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?charset=utf8", dbUser, dbPwd, dbHost, dbPort, dbName) + "&loc=" + url.QueryEscape("Asia/shanghai")
orm.RegisterDriver("mysql", orm.DR_MySQL)
orm.RegisterDataBase("default", "mysql", dbUrl, maxIdleConn, maxActiveConn)
#Cfg为beego.AppConfig变量
{% endraw %}
{% endhighlight %}

### 单一对象的CRUD操作

{% highlight bash %}
{% raw %}
(1)定义一个结构体
type UserInfo struct { //beego默认将其转为user_info表，也可根据实际情况改变，参看[模型定义](http://beego.me/docs/mvc/model/models.md)
    Id int `orm:"pk:auto"` //主键自增
    Name string
    ...
    TS  time.Time
}
//可以定义索引
func (u *UserInfo) TableIndex() [][]string {
    return [][]string{
        []string{"Name"},
    }
}

//注册模型
orm.RegisterModel(new(UserInfo))
//一般在初始化init方法中注册

(2)新增(创建C)操作
//RunSyncdb(name,force,verbose)功能：建表。
//name为数据库别名，且default不能换；force为是否强制建数据库；verbose是否打印建表过程
orm.RunSyncdb("default", false, true)
o := orm.NewOrm()
u := new(UserInfo)
UserInfo.Name = "xialingsc"
idInt64,err :=  o.Insert(UserInfo)
...

(3)查询(读取R)操作
一般应用中R操作分为获取列表和单个对象两种
a.获取列表
o := orm.NewOrm()
var userList []*UserInfo
这里使用了一个Filter进行了过滤，实际使用中可根据情况决定使用与否
_, err := o.QueryTable(new(UserInfo)).Filter("Name", d.GetSession("uid").(string)).All(&userList)

b.获取单个对象
o := orm.NewOrm()
user := new(UserInfo)
user.Name = "xialingsc"
o.Read(user)

(4)更新U操作
o := orm.NewOrm()
user := new(UserInfo)
user.Name = "xialingsc"
...//更多具体内容
o.Update(user)

(5)删除D操作
o := orm.NewOrm()
user := new(User)
user.Name = "xialingsc"
o.Delete(user)


{% endraw %}
{% endhighlight %}


## 注意事项

在将Go与Mysql进行结合运用的过程中，有一些细节需要注意，比如时间类型处置等问题。

(1)时间类型处置

当你在mysql中设置某字段类型为时间戳timestamp时，在Go语言中如何处理才能存入到该字段呢？我们既可以采用mysql在变动时自动加上时间戳的机制，还可以在go程序中利用time.Now()进行赋值。但问题的重点在于，我们取出时间在页面进行显示的时候，就傻眼了，明明是按'0000-00-00 00:00:00'格式的，结果变为了'0000-00-00 00:00:00 +0000 UTC'格式，如何处理呢？这里就必须利用特殊字符串对其进行格式化，类似为time.Now().Format("2006-01-02 15:04:05"),特殊字符串必须为"2006-01-02 15:04:05",必须为"2006-01-02 15:04:05",必须为"2006-01-02 15:04:05"，重要的事就得多说几遍。那些golang大贤们也老牛了，用诞生日作牛串。

(2)一对一、一对多、多对多的处理

参看[模型定义](http://beego.me/docs/mvc/model/models.md#表关系设置)、[模型定义](http://beego.me/docs/mvc/model/models.md#表关系设置)、[模型定义](http://beego.me/docs/mvc/model/models.md#表关系设置),重要的事又说了三遍！！


简单的使用方法就介绍到这里，还有更多具体应用，就需要自己去多探索和实践了。



















