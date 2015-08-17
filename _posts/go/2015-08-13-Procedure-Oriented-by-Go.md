---
layout: article
title:  "Golang学习系列(3)-面向过程的函数编程"
categories: go
disqus: true
image:
    teaser: teaser/gophertraining.jpg
---

{% highlight bash %}
{% raw %}
本文是学习Golang语言的系列文章之一，主要描述go语言中的流程控制及函数操作，以及进行实践的相关代码
{% endraw %}
{% endhighlight %} 

---


## 前言

Go可以用于写纯过程式程序，也可用于写面向对象程序，还可以写出将两者结合的程序。Go语言的过

程式编程是面向对象编程的基础，所以本文主要讨论下过程式编程中涉及的一些知识。

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

从上述格式可以看到，go中if语句的用法与java中有区别，它可以在条件判断语句里面声明一个变量，

判断语句可以不需要括号。这里需要主要作用域的问题。

#### swtich

Go语言中有两种类型的switch语句，表达式开关和类型开关，表达式开关在C、C++及Java中经常见到，

但类型开关属于Go的专有。Go的swtich不会自上向下贯穿，即不必再每个case末尾添加一个break语句，

注意啊，这里就比java等语言显得高明了许多，当然在需要的时候，可以通过显式地调用fallthrough

语句来这样做。

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

{% endraw %}
{% endhighlight %}

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

只有当至少有一个通信(接收或者发送)到达时才完成阻塞。一个包含default语句的select语句是非阻塞

的，并且会立即执行。


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

这里有时候可能使用平行赋值，例如：i,j = i+1,j-1

循环中有两个关键操作，一个是break,另一个是continue;break是跳出当前循环,即当前循环结束，而

continue是跳过本次循环，循环可能还未结束；break和continue还可以跟着标号，用来跳到多重循环

中的外层循环。

由于Go支持多值返回，对于"声明而未被调用"的变量，编译器会报错，这种情况使用_来丢弃不需要的返回值。

for循环中，有时还会用到goto语句进行跳转。

---

## 函数

函数是面向过程编程的根本，是Go语言的核心设计，用func来定义。

{% highlight bash %}
{% raw %}
格式：func funcName(input1 type1,input2 type2)(output1 type1,output2 type2){
    //逻辑处理代码
    //返回多个值
    return value1,value2
}
{% endraw %}
{% endhighlight %}

如果只有一个返回值且不声明返回值变量，那么可以省略"包括返回值" 的括号，还可以直接写返回的类型；

如果没有返回值，就直接省略最后的返回信息。

Go语言比C语言更先进的特性之一是函数能够返回多个值。

### 变参

Go语言函数支持变参。接收变参的函数有不定数量的参数。

格式：func myfunc(arg ... type){}

arg ... type表示Go语言接受不定数量的参数，且具有相同类型type。

### 传值与传指针

Go语言中的函数值传递与Java语言中一样，均是指当传递一个参数值到被调用函数里面时，实际上传了

这个值的一份copy,当在被调用函数中修改参数值的时候，调用函数中实参不会发生变化。


指针传递是指将变量存放在内存中的地址&x传入函数，若函数中改变了该地址的值，则意味着原传入

函数的变量值也被改变了。

{% highlight bash %}
{% raw %}
package valueaddress

import (
    "fmt"
)

func TestTransferValue() {
    x := 3
    fmt.Println("The initValue of x is :%d", x)
    x1 := addvalue(x)
    fmt.Println("The x+1= is :%d", x1)
    fmt.Println("The initValue of x is :%d", x)
}

func TestTransferAdderess() {
    x := 3
    fmt.Println("The initValue of x is :%d", x)
    x1 := addaddress(&x)
    fmt.Println("The x+1= is :%d", x1)
    fmt.Println("The initValue of x is :%d", x)
}

func addvalue(a int) int {
    a = a + 1
    return a
}

func addaddress(a *int) int {
    *a = *a + 1
    return *a
}
{% endraw %}
{% endhighlight %}


### defer 

Go语言中有延迟(defer)语句，可以在函数中添加多个defer语句。当函数执行到最后时，这些defer语句

会按照逆序执行，最后该函数返回。这种设计特别在因正常或因不知原因的异常出现关闭资源使用。

相关示例代码可参见《Go Web 编程》。

{% highlight bash %}
{% raw %}
示例一
package defertest

import (
    "fmt"
)

func TestDefer() {
    for i := 0; i < 5; i++ {
        fmt.Printf("The order is :%d\n", i)
    }
    
    for j := 0; j < 5; j++ {
        defer fmt.Printf("The reverse order is :%d\n", j) //输出4,3,2,1,0
    }
}
{% endraw %}
{% endhighlight %}

{% highlight bash %}
{% raw %}
示例二
func ReadWrite() bool {
    file.Open("file")
    defer file.Close()
    if failureX {
        return false
    }
    if failureY {
        return false
    }
    return true
}
{% endraw %}
{% endhighlight %}


### 函数作为值、类型

在Go语言中函数也是一种变量，可以通过type定义它，它的类型是所有拥有相同的参数，相同的返回值。格式：

type typeName func(input1 inputType1,input2 inputType2 [,...]) (return1 returnType1 [,...])


{% highlight bash %}
{% raw %}
/**
*函数测试
*作者：xialingsc
*时间：2015-3-09
*
**/

package functest

import (
    "fmt"
)

type testInt func(int) bool //声明一个函数类型

func Testfunc() {
     x := 3
     y := 4
     z := 5
     max_xy := max(x, y)
     max_yz := max(y, z)
     fmt.Printf("max(%d,%d)=%d\n", x, y, max_xy)
     fmt.Printf("max(%d,%d)=%d\n", y, z, max_yz)
     fmt.Printf("max(%d,%d)=%d\n", x, z, max(x, z))
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

func isOdd(integer int) bool {
    if integer%2 == 0 {
        return false
    }
    return true
}

func isEven(integer int) bool {
    if integer%2 == 0 {
        return true
    }
    return false
}

//函数类型作为一个参数
func filter(slice []int, f testInt) []int {
    var result []int
    for _, value := range slice {
        if f(value) {
            result = append(result, value)
        }                                        
    }
    return result
}

func TestFuncAsPara() {
    slice := []int{1, 2, 3, 4, 5, 6, 7}
    fmt.Println("slice=", slice)
    odd := filter(slice, isOdd)
    fmt.Println("the odd element of slice is:", odd)
    even := filter(slice, isEven)
    fmt.Println("the even element of slice is:", even)
}
{% endraw %}
{% endhighlight %}


### main函数和init函数

Go里面有两个保留函数，一个是init函数和main函数，前者能够在不同package中被调用，后者只

能用于package main。这两个函数共同点是在定义时不能有任何的参数和返回值。尽管一个包里可

以写任意多个init函数，但建议只写一个，便于阅读。每个init是可选的，但一个package有且只有

一个main函数。

执行过程：程序的初始化和执行都起始于main包，若main包还导入了其他的包，在编译时就会依次导

入，若一个包被多个同时导入，则只会被导入一次。当一个包被导入时，如果该包还导入了其他包，

那么会先将其他包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数，

依次类推，等所有导入的包被加载完毕，就开始对main包中的包级常量和变量进行初始化，然后执行

main包中的init函数，最后执行main函数。


### Panic和Recover


Go语言没有像java语言的异常机制，不能抛出异常，而是使用panic和recover机制，正常情况下，

代码中都没有或者很少有panic的东西，但它的确很强大。panic是一个内建函数，可以中断原有的

控制流程，进入一个令人恐慌的流程当函数F调用panic时，函数F的执行被中断，但是F中的延迟函

数会正常执行然后F返回到调用它的地方。Recover是一个内建函数，可以让进入令人恐慌的流程中

的goroutine恢复过来recover仅在延迟函数中有效，在正常的执行过程中，调用recover会返回nil，

并且没有任何效果

{% highlight bash %}
{% raw %}
/**
*
*作者：xialingsc
*时间：2015-3-09
*
**/

package panicrecover

import (
    "fmt"
    "os"
)

var user = os.Getenv("USER")

func init() {
    fmt.Println("This is panicrecover init()")
    if user == "" {
        fmt.Println("The user is nil")
        panic("no value for  $USER")
    } else {
        fmt.Println("The user is %s", user)
    }
}

func throwsPanic(f func()) (b bool) {
    defer func() {
        fmt.Println("This is throwsPanic func")
        if x := recover(); x != nil {
            b = true
        }
}()

f() //执行函数f,如果f中出现了panic,那么就恢复回来
    return
}

{% endraw %}
{% endhighlight %}


## import

格式： import "path/package"

import (
    "path1/package1"
    "path2/package2"
    ...
)

{% highlight bash %}
{% raw %}
相对路径：import "./model" //当前文件同一目录的model目录，不建议这么用

绝对路径：import "shorturl/model" //加载gopath/src/shorturl/model模块

点操作示例 ：import (
            . "fmt"
         ) 
//.的含义表示这个包导入之后，在调用这个包的函数时，你可以省略前缀的包名
fmt.Println("ceshi") => Println("ceshi")

别名操作示例：import (
            f "fmt"
          )
//fmt.Println("ceshi")  => f.Println("ceshi")

_操作示例 ： import (
            _ "ray/tech/goceshi"
         )
// _是引入goceshi该包，不直接使用包里面的函数，而是调研了该包里的init函数。


{% endraw %}
{% endhighlight %}





























