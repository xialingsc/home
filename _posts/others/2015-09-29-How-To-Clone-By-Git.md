---
layout: article
title:  "如何在Ubuntu中获取现有git项目代码的操作步骤"
categories: others
disqus: true
image:
    teaser: teaser/gitlab.jpg
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

