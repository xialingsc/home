---
layout: article
title:  "如何在Ubuntu中获取现有git项目代码的操作步骤"
categories: others
disqus: true
image:
    teaser: teaser/gitlib.jpg
---

{% highlight bash %}
{% raw %}
本文是自己早前在部门内部写的一篇文章，主要目的是让git零基础的同学顺利获取代码，快速开展工作。
{% endraw %}
{% endhighlight %} 

---

系统环境：Ubuntu 12.04

目标对象：git零基础的同学

## 在ubuntu中安装git

$ sudo apt-get install git

【注：也可以通过源码进行安装，可参考《Pro.Git-zh_CN.pdf》】

## 配置本机git环境

{% highlight bash %}
{% raw %}
$ git config --global user.name "xialingsc"
$ git config --global user.email xialingsc@xxx.com
【注1：请根据实际情况自行修改字符xialingsc@xxx.com中的"xialingsc";
注2：--global 选项,表示用户目录下的配置文件只适用于该用户】
{% endraw %}
{% endhighlight %}

## 生成密钥

{% highlight bash %}
{% raw %}
ssh-keygen -t rsa -C "xialingsc@xxx.com"
回车之后，
出现：
Generating public/private rsa key pair.
Enter file in which to save the key (/home/xl/.ssh/id_rsa):
此时，输入/home/xl/.ssh/id_rsa，回车之后
出现：
Enter passphrase (empty for no passphrase):
此时可空，也可同git上密钥，回车之后
出现：
Enter same passphrase again:
再次确认之，我们可以在 /home/xl/.ssh/下看id_rsa.pub文件
{% endraw %}
{% endhighlight %}

ssh-keygen 生成2个文件，位于~/.ssh/下，分别是公共密钥id_rsa.pub,和私有密钥id_rsa。

## 提交密钥

vim /home/xl/.ssh/id_rsa.pub 打开后，复制其内容，到内部git中登陆自己的账号(该帐号需git管理员开通)，然后在个人account setting中，找到SSH KEY，将复制的密钥加入，（需要再次输入密码）【注：此处需要注意空格影响】

## 检验是否能链接上了内部git

$ ssh git@git.itari.com.cn 

若出现，
PTY allocation request failed on channel 0
Welcome to GitLab, xialingsc!
Connection to git.xxx.com closed.
表示可以成功链接

## 从现有仓库克隆page

将page克隆到当前目录下
$ git clone git@git.xxx.com:xialingsc/page 
将page克隆到指定目录下,例如:/home/xl/test/
$ git clone git@git.xxx.com:xialingsc/page /home/xl/test/

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

