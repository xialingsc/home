---
layout: article
title:  "Golang学习系列(2)-面向过程的函数编程"
categories: go
disqus: true
image:
    teaser: teaser/GoogleGo.jpg
---

{% highlight bash %}
{% raw %}
本文是学习Golang语言的系列文章之一，主要描述go语言中的流程控制及函数操作，以及进行实践的相关代码
{% endraw %}
{% endhighlight %} 

---


## 前言

Go可以用于写纯过程式程序，也可用于写面向对象程序，还可以写出将两者结合的程序。Go语言的过程式编程

是面向对象编程的基础，所以本文主要讨论下过程式编程中涉及的一些知识。

---

## 流程控制

Go语言的流程控制分为三大类：条件判断、循环控制和无条件跳转。

### 条件判断

条件判断又称分支语，包括三类，即if、switch和select。

#### if

格式：
{% highlight bash %}
{% raw %}
if optionalStatment1;booleanExpression1 {
    block1;
} else if optionalStatement2;booleanExpression2{
    block2;
} else {
    block3;
}
{% endraw %}
{% endhighlight %}

从上述格式可以看到，go中if语句的用法与java中有区别，它可以在条件判断语句里面声明一个变量，判断语句

可以不需要括号。这里需要主要作用域的问题。

#### swtich

Go语言中有两种类型的switch语句，表达式开关和类型开关，表达式开关在C、C++及Java中经常见到，但类型开关

属于Go的专有。Go的swtich不会自上向下贯穿，即不必再每个case末尾添加一个break语句，注意啊，这里就比java

等语言显得高明了许多，当然在需要的时候，可以通过显式地调用fallthrough语句来这样做。

(1)表达式开关

格式：
{% highlight bash %}
{% raw %}
switch optionalStatement;optionalExpression {
case expressionList1:block1
...
case expressionListN:blockN
default:blockD
}
{% endraw %}
{% endhighlight %}

代码示例：
{% highlight bash %}
{% raw %}
func BoundedInt(minimum,value,maximum int) {
    switch{
    case value < minimum:
        return minimum
    case value >maximum:
        return maximum
    }
    return value
}

经典用法；
switch Suffix(file) {
case ".gz":
    return GzipFileList(file)
case ".tar",".tar.gz",".tgz":
    return TarFileList(file)
case ".zip":
    return ZipFileList(file)
}

{% endraw %}
{% endhighlight %}

{% highlight bash %}
{% raw %}
这里介绍了goto的用法，但goto在程序中使用时需要多加注意。
package gotofallthrough
import "fmt"
func TestGoFallthrough() {
    i := 0
Here: //以冒号结束作为标签
    fmt.Println(i)
    i++
    if i < 5 {
        goto Here
    } else {
        fmt.Println("goto is over")
    }
    switch i {
    case 4:
        fmt.Println("The integer is 4")
    case 5:
        fmt.Println("The integer is 5")
        fallthrough
    case 6:
        fmt.Println("The integer is 6")
    default:
        fmt.Println("The default tag")
    }
}
{% endraw %}
{% endhighlight %}
(2)类型开关

格式：
{% highlight bash %}
{% raw %}
switch optionalStatement;typeSwitchGuard {
case typeList1:block1
...
case typeList2:blockN
default:blockD
}

{% endraw %}
{% endhighlight %}


代码示例（这里还包括了goto的用法）：
{% highlight bash %}
{% raw %}
func classifier(items...interface{}){
    for i,x := range items{
        switch x.(type){
        case bool:
            fmt.Printf("param #%d is a bool\n",i)
        case float64:
            fmt.Printf("param #%d is a float64\n",i)
        case int,int8,int16,int32,int64:
            fmt.Printf("param #%d is an int \n",i)
        case uint,uint8,uint16,uint32,uint64:
            fmt.Printf("param #%d is an unsigned int \n",i)
        case nil:
            fmt.Printf("param #%d is nil \n",i)
        case string:
            fmt.Printf("param #%d is string \n",i)
        default:
            fmt.Printf("param #%d's type is unknow\n",i)
        }
        
    }
}

#### select 

{% highlight bash %}
{% raw %}
格式：select {
    case sendOrReceive1: block1
    ...
    case sendorReceive2: block2
    default:blockD
}
{% endraw %}
{% endhighlight %}

一个select 语句的逻辑结果为：一个没有default语句的select语句会阻塞，

只有当至少有一个通信(接收或者发送)到达时才完成阻塞。一个包含default语句的select语句是非阻塞的，

并且会立即执行。


select 属于并发编程的范畴，这块具体示例代码，将在并发编程中进行详细描述。

---

### for 循环

Go中使用两种类型的for语句进行循环，一种是无格式的for语句，另一种是for ... range语句。

for循环是Go中最强大的控制逻辑。

{% highlight bash %}
{% raw %}
以下是for语法示例(参见《Go语言程序设计》)
for { //无限循环
    block
}

for booleanExpression { //while循环
    block
}

for optionalPreStatement;booleanExpress;optionalPostStatement { //分号在可选的前置或后置声明存在时使用
    block
}

for index,char := range aString{ //一个字符一个字符地迭代一个字符串
    block
}

for index := range aString { //一个字符一个字符地迭代一个字符串
    block //char ,size := utf8.DecodeRunInString(aString[index])
}

for index,item := range anArrayOrSlice { //数组或者切片迭代
    block
}

for index := range anArrayOrSlice { // 数组或者切片迭代
    block //item := anArrayOrSlice[index]
}

for key, value := range aMap { //映射迭代 
    block
}

for key := range aMap { //映射迭代
    block //value:=aMap[key]
}

for item := range aChannel { //通道迭代
    block
}

{% endraw %}
{% endhighlight %}

---





























