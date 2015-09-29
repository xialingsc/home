---
layout: article
title:  "让多服务器间ssh登录变得更美好"
categories: others
disqus: true
image:
    teaser: teaser/ssh.jpg
---

{% highlight bash %}
{% raw %}
本文主要描述了如何设置ssh相关配置，实现在多台服务器间通过ssh登录避免每次都输入密码问题，省时省力。
这篇文章是原同事杨昌明所写，并分享于内部，今天再版此文既是学习，也是对在一起共事情景的怀念。
{% endraw %}
{% endhighlight %} 

---

## 背景

部门内部有几台服务器，大家经常登陆，但是每次使用SSH登陆过于麻烦，我们通过设置可以免去输入用户名和密码。

## 生成密钥

ssh-keygen 生成2个文件，位于~/.ssh/下，分别是公共密钥id_rsa.pub,和私有密钥id_rsa。

## 发送自己的公共密钥给服务器端

ssh-copy-id user@host

以上命令是给远程服务器发送自己的公共密钥，服务器端会要求输入密码。服务端拿到这个密钥后会在.ssh/下保存，文件名为authorized_keys；在mac平台下没有该命令，可以通过scp命令将公共密钥文件copy至服务端改名即可。

此时在登陆服务端，无需输入密码，登陆过程如下：

- 客户端首先将自己的公共密钥发送给服务器端存储起来
- 客户端发起登录请求，服务器端接受到后，发送一段随机的字符串给客户端
- 客户端接受该随机字符串后使用自己的私有密钥进行加密，然后发送给服务端
- 服务端接收后，使用存储的客户端公共密钥进行解密，解密成功，客户端即可登录，就无须输入密码

## 免去冗长的user@host

在本地配置config文件，位于~/ssh/config，在该文件加入如下几行配置：
{% highlight bash %}
{% raw %}
host 67
    HostName 192.168.1.67
    User gap
host 66
    HostName 192.168.1.66
    User gap
{% endraw %}
{% endhighlight %}

那么我们登陆时，输入如下命令，无须用户名和密码，从此世界干净了许多

ssh 67

## 其他

如果在同一电脑上使用两个github的账号怎么处理呢？ 该方法是另一同事马啸实践中处理的方式。

解决办法即变为：
{% highlight bash %}
{% raw %}
HOST github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa

HOST github_work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_work
这样的话,你就可以通过使用github.com别名github_work来明确说你要是使用id_rsa_work的SSH key来连接github，即使用工作账号进行操作
#push到github上去
$ git remote add origin git@github_work:xxxx/test.git
$ git push origin master
{% endraw %}
{% endhighlight %}

