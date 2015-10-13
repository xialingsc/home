---
layout: article
title:  "如何利用Shell统计行数及获取代码"
categories: shell
disqus: true
image:
    teaser: teaser/shell.jpg
---

{% highlight bash %}
{% raw %}
本文是以前同事付学良在部门内部写的一篇文章，最近自己也在协助申请软件著作权的事情，所以将相关内容进行完善整理，以便备用。
{% endraw %}
{% endhighlight %} 

---

每个软件公司都应该会去做同一件事，即将自己开发的产品进行软件著作权登记，那么需要提供哪些资料呢？今天简单聊聊为协助完成软件著作权，需要开发者提供的代码如何通过部分Shell来轻松完成。

## 需求

某天，一位MM轻盈地走到我身旁，

请问：“您是夏令吗？”

我说：“是，有什么事吗？”

“是这样，我们最近要进行你软件著作权登记，刚好是您开发的，需要您提供部分源代码和用户手册，您看什么时候能给我？.....”

简单地总结就是：你丫需要给我不少于6000行的代码和关于这个软件的用户手册，要不跟你没完。要是整个代码没有6000行，那就给全部代码，否则将一直骚扰你。

## 开始动手

### 统计当前代码行数

基本思路就是到项目目录，查找所有符合规则的source文件，然后利用cat合并文件到缓冲区，然后利用wc统计总共行数。

{% highlight bash %}
{% raw %}
find . \( -name '*.go' -o -name '*.xml' -o -name '*.properties' \) | xargs cat | wc -l
{% endraw %}
{% endhighlight %}

### 抽取前后3000行代码

这里简单描述下思路，先用cat执行文件合并find . -name '*.go' | xargs cat > file（ source的话只需要go文件即可，可以不统计其他后缀的），把所有文件合并成一个文本文件，然后利用SED取代码或者利用split来分割代码，还可以利用tail和head来获取。

1） 利用Sed

利用sed拿到1-3000行数据，利用w写入到新的head文件中即可，sed '1,3000 w head' file，取后边3000行需要自己计算下行数了，明显看到sed不是完成这个功能合适的工具。

2）利用Split

这个方式是最差的，后边3000行就比较悲催，咱们稍微了解下split的用法即可，split -l 3000 file split,这样的话就自动以3000为单位划分出splitaa、splitab等等的文件，前面3000肯定是没问题的，但是后边3000行是不完整的。

3）利用tail和Head

取得前3000行代码输出到head文件里面head -3000 file > head,取得后3000行代码tail -3000 file > foot,大功告成。这个方法应该最为简便。

还可以将head与tail组合使用：
{% highlight bash %}
{% raw %}
head -n 6000 file | tail -n +1 
输出file的1行至6000行内容。
{% endraw %}
{% endhighlight %}



### 用户手册

这事真还用Shell干不了， 得靠人来完成，所以，还是乖乖手写去吧。

