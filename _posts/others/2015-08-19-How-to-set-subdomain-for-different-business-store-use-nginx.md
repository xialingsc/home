---
layout: article
title:  "如何利用Nginx实现同一商家不同门店的二级域名请求分发"
categories: others
disqus: true
image:
    teaser: teaser/nginx.jpg
---

{% highlight bash %}
{% raw %}
本文结合项目组实际需求，主要描述采用nginx正则表达式通过定义内部变量实现二级域名映射到
不同物理路径目录的过程。
{% endraw %}
{% endhighlight %} 

---


## 需求

目前公司接到的电商项目越来越多，这些需求中大多都有新增门店的情况，且各个门店希望有自己对应的二级域名。即当客户访问主域名时，比如www.nei.zm,能解析到公司主页内容的页面上；当客户直接输入a.nei.zm时，希望能解析到门店a的主页内容的页面上；当客户直接输入b.nei.zm时，希望能解析到门店b的主页内容的页面上；如何利用nginx实现该应用需求？

---

## 解决方案

采用nginx正则表达式通过定义内部变量实现二级域名映射到不同物理路径目录的过程。

---

## 详细步骤

### 测试环境

{% highlight bash %}
{% raw %}
操作系统：Mac OS X Yosemite 10.10.3
nginx版本: nginx version: nginx/1.9.3
web服务器： apache-tomcat-6.0.44
(这里省略了Unix安装nginx及Tomcat的步骤)
{% endraw %}
{% endhighlight %}


### 修改hosts内容

在/etc/hosts文件中增加如下几行内容

{% highlight bash %}
{% raw %}
127.0.0.1  nei.zm
127.0.0.1  www.nei.zm
127.0.0.1  a.nei.zm
127.0.0.1  b.nei.zm
这里以nei.zm及其子域名作为访问对象
{% endraw %}
{% endhighlight %}

### 增加测试内容

在$TOMCAT_HOME/webapps下创建zm目录及子目录a、b,同时在三个目录下创建内容不同的index.html文件,如图所示：

![alt text](/images/teaser/file.png "文件图片")

### 配置/usr/local/nginx/conf/nginx.conf文件

{% highlight bash %}
{% raw %}
server {
    listen       80;
    server_name  localhost;
    #二级域名不同访问不同子目录
    set $subdomain '';
    if ( $host ~* (\b(?!www\b)\w+)\.\w+\.\w+ ) {
        set $subdomain /$1;
    }
    
    location / {
        if ($subdomain != '') {
            proxy_pass http://127.0.0.1:8080/zm/$subdir/index.html;
        }
        #index  index.html index.htm;
        proxy_pass http://127.0.0.1:8080/zm/index.html;                                                                                                              
    }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;                                                                                                                                                      }
}

{% endraw %}
{% endhighlight %}

### 测试内容

(1)访问a.nei.zm如图所示
![alt text](/images/teaser/a-nei.png "a-nei.zm")


(2)访问b.nei.zm如图所示
![alt text](/images/teaser/b-nei.png "b-nei.zm")

(2)访问www.nei.zm或nei.zm如图所示
![alt text](/images/teaser/www-nei.png "www-nei.zm")







































动态获取远程代码。这个代码内部分为两部分，一是下载源码包，另一步是执行go install

为了让go get正常使用，需保证安装了合适的源码管理工具，并将这些命令加入到PATH中。

可通过go help remote 了解更多。


## go test

执行这个命令会自动读取源码目录下名为*_test.go文件，生成并运行测试用的可执行文件。

默认情况下不需要任何参数，也可带上参数，具体可参见go help testflag


## go doc

如何查看相应的package文档呢？

如果是builtin包，可以执行go doc builtin;如果是http包，执行go doc net/http;

查看某个包里面的函数，类似执行 godoc fmt Println,还可以查看相应代码 

godoc -src fmt Println


很棒的一点是，可以在终端执行godoc -http=:端口号，例如godoc -http=:8080 ,就可以在

浏览器中敲入127.0.0.1:8080进行文档内容的查看。


## 其他命令

{% highlight bash %}
{% raw %}
go fix        用来修复以前老版本的代码到新版本
go version    查看go当前的版本
go env        查看当前go的环境变量
go list       列出当前全部安装的package
go run        编译并运行go语言程序
{% endraw %}
{% endhighlight %}





























