---
layout: article
title:  "Golang学习系列(2)-语言基础"
categories: go
disqus: true
image:
    teaser: teaser/GoogleGo.jpg
---

{% highlight bash %}
{% raw %}
本文是学习Golang语言的系列文章之一，主要描述go语言的基础语法，以及进行实践的相关代码
{% endraw %}
{% endhighlight %} 

---


## Golang关键字

Go语言是一门类似C语言的编译型语言，共有25个关键字。

{% highlight bash %}
{% raw %}
break default func interface select case defer go map struct chan else goto package
switch const fallthrough if range type continue for import return var

{% endraw %}
{% endhighlight %}
---

## 第一个简单的Go程序

学习任何一门语言，基本都是从输出简单的aello world开始的，本文也仍觉得这种方式比较酷，@_@。

{% highlight bash %}
{% raw %}
本文以GOPATh=/home/xialingsc/go为例，作为代码空间。
在$GOPATH/src下建立main目录，
package main
import "fmt"

func main(){
    fmt.Println("Hello World,This is xialingsc");
}
{% endraw %}
{% endhighlight %}


解释：Go是通过package进行组织的,每一个可独立运行的Go语言程序，必定包含一个package main.在这个main包中必定包含一个入口函数main，而这个函数没有参数，也没有返回值。Go语言天生支持UTF-8编码，任何字符都可以直接输出,还可以用UTF-8中的任何字符作为标识符。

---

## Go的一些规则

Go采用默认行为而显得简洁。

- 大写字母开头的变量才可以被其他包读取，为公用变量；小写字母开头为私有变量，不可导出。

- 大写字母开头的函数也遵循上述规律

---

## 定义变量 

格式：var variableName type

//定义多个变量

var v1,v2,v3 string

//定义变量并初始化值

var v1 int = 10

//同时初始化多个变量

var v1,v2,v3 int = 6,7,8

//函数体内的简单声明赋值

a,b,c := 8,9,10

//_（下划线）是特殊变量名，任何赋予它的值都会被丢弃，为什么我们还用它？这就是go灵活的表现，在某些场景下，获取的返回值在接下来的程序没有实际用处，但如果声明了而不用，Go会在编译阶段报错提示。


---

## 常量 

常量即在编译阶段就确定下来的值，程序运行时无法改变。常量可定义为数值、布尔值或字符串等类型。

格式：const constName type = value

例如:const Pi float32 = 3.141516926 或 const Pi = 3.141516926


## 八种内置基础类型 

### Boolean

Go语言中，布尔值的类型为bool,值为true或false ，默认为false。

格式: var variableName bool

### 数值类型

(1)整数类型

整数类型分为无符号和带符号两种，Go同时支持int 和 uint两种，这两种类型长度相同，但具体长度取决于不同编译器的实现。

整数类型分为：rune、int8、int16、int32、int64和byte、uint8、uint16、uint32、uint64

byte 是uint8的别称

注意：这些类型的变量之间不允许互相赋值或操作。尽管int的长度是32bit,默认值为0，但int与int32不能互用。

(2)浮点数类型

浮点类型有两种，即float32和float64,默认为float64。

另外Go还支持复数，格式为RE+IMi,有两种，分别是complex64,complex128，默认为complex128

例如:var c complex64 = 10+12i
fmt.Printf("Value is : %v",c)

### 字符串

- Go中字符串都是采用UTF-8字符集编码，字符串是用一对双引号("")或反引号(``)来定义，类型为string

格式：var abcstr string = "abc"

- Go中字符串是不可变的，例如：abcstr[0] = 'c' 会编译报错。若真希望进行改变，则可实现为：

newstr := []byte(abcstr) //将字符串abcstr转换为[]byte类型

newstr[0] = 'c'

abcstr1 := string(newstr)

fmt.Println("%s \n",abcstr1)

- 可用采用"+"操作符连接两个字符串

- 修改字符串还可用利用切片(后续会介绍)进行修改

abcstr := "oldstr"

abcstr = "new" + abcstr[3:]

- 还可声明一个多行的字符串

{% highlight bash %}
{% raw %}
abcstr := `muli

            string`

{% endraw %}
{% endhighlight %}

### 错误类型

Go中内置了一个error类型，专门来处理错误信息。Go中package还有一个包errors来处理错误。

{% highlight bash %}
{% raw %}
err := errors.New("ceshi errors")
if err != nil {
    fmt.Print(err)
}
{r endraw %}
{% endhighlight %}

### iota枚举

Go中有一个关键字iota,主要用来声明enum的时候采用，默认开始值为0,每调用一次加1，若全新声明另一组enum(即一组新的const)，iota从0开始

{% highlight %}
{% raw %}
const (
    a = iota //x=0
    b = iota //y=1
    c        //c=2,省略时，默认与前一个值的字面相同
    d        //d=3
)

const e = iota //e=0
{% endraw %}
{% endhighlight %}


### array数组

格式:var arr [n]type,下标从0开始,数组作为参数传入函数，其实是值赋值，并非指针，指针可采用slice.

{% highlight bash %}
{% raw %}
//常规声明与赋值
var a [3]int
a[0] = 1
fmt.Println("The first number is %d\n",a[0]) //1
fmt.Println("The last number is %d\n",a[2]) //0

//其他声明与赋值
a := [3]int{1,2,3} //声明长度为3并赋值
b := [5]int{1,2,3} //声明长度为5，并赋予前三个分别为1，2，3，其他为0
c := [...]int{1,2,3} //省略长度，又Go自动根据元素个数来计算长度
{% endraw %}
{% endhighlight %}

Go中支持嵌套数组，即多维数组
{% highlight bash %}
{% raw %}
multiArray := [2][3]int{[3]int{1,2,3},[3]int{4,5,6}}
由于内外部类型一致，均为int，则可以省略为:
multiArray := [2][3]int{{1,2,3},{4,5,6}}

例如：
dobuleArray := [2][4]int{{1, 2, 3, 4}, {5, 6, 7, 8}}
fmt.Println("The first element of DobuleArray is %d:", dobuleArray[0][0])
fmt.Println("The last element of DobuleArray is %d:", dobuleArray[1][3])
{% endraw %}
{% endhighlight %}


### slice 

这是Go语言具有的特性之一，是具有动态数组特性的引用类型，满足初始定义时并不知道需要多大的数组场景。

格式:var sliceArr []type ,比array少了类型长度

例如：buf := []byte{'a','b','c'}

slice可以从一个数组或一个已存在的slice中再次声明，通过array[i:j]来获取，i为开始位置，j为结束位置,但不包含array[j],长度为j-i

{% highlight bash %}
{% raw %}
//声明一个长度为10个元素类型为byte的数组
var initArray = [10]byte{'a','b','c','d','e','f','g','h','i','j'}
//声明两个byte的slice
var aslice,bslice []byte
//指向数组不同位置
aslice = initArray[2:5] //aslice则包含initArray[2],initArray[3],initArray[4]
bslice = initArray[3:5] //bslice则包含initArray[3],initArray[4]

{% endraw %}
{% endhighlight %}

slice的简便操作
{% highlight bash %}
{% raw %}
默认开始位置为0，initArray[:2]等价与initArray[0:2]
slice第二个序列默认是数组的长度,initArray[2:]等价于initArray[2:len(initArray)]
initArray[:] 等价于 initArray[0,len(initArray)]
slice的长度len与容量cap是不同的概念，后者是指从slice指定的位置开始，一直到原数组的最后一个值
因为slice是引用类型，所以引用改变原来值后，其他的所有引用也都会改变该值
从概念上而言，slice像一个结构体，这个结构体包含了三个元素，一个指针，长度len及最大长度cap
slice内置函数:len,cap,append(追加，然后返回同类型slice),copy(从源slice的src复制元素到目标dst,
并返回复制的元素的个数

用法：
slice := []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g'}
copySlice := []byte{'x', 'y', 'z'}
copy(slice, copySlice)
fmt.Printf("The Element of copySlice is %s\n", copySlice)
fmt.Printf("The Element of slice is %s\n", slice)
sliceEven := make([]int, 0)
sliceEven = append(sliceEven, 1, 2, 3, 4, 5)

{% endraw %}
{% endhighlight %}


### map

格式：map[keyType]valueType

map的读取和设置与slice一样，通过key操作，只是slice的index是`int`类型，而map多了其他类型,可以为int,string及所有定义了==与!=操作的类型

//声明一个key是字符串，值为int的map,使用前必须使用make初始化

var dictionary map[string]int

dictionary = make(map[string]int)


//另一种map的声明方式,采用make初始化

numbers := make(map[string]int)

numbers["one"] = 1 //赋值

fmt.Println("测试输出：",numbers["one"])


使用map需要注意的几点内容：
{% highlight bash%}
{% raw %}
map是无序的，每次打印输出都不一样
map 长度不固定，属于引用类型
内置len函数，返回map拥有的key的数量
修改map的值，可以直接numbers["one"] = 12
map初始化可以通过key:val方式实现，map内置有判断是否存在key的方式，通过delete删除map的元素

//初始化一个map
rating := map[string]float32{"c": 5, "FO": 3.4, "python": 2.3}
ratingvalue, ok := rating["c#"]
if ok {
    fmt.Println("c is in the map and it's value is:%d\n", ratingvalue)
} else {
    fmt.Println("c isn't in the map")
}
//删除
delete(rating, "c")
fmt.Println("itretor map start:")
for _, value := range rating {
    fmt.Println("the element of map is %s", value)
}

{% endraw %}
{% endhighlight %}



## make及new操作

make用于内建类型(map,slice,channel)的内存分配

new用于各种类型的内存分配，new 返回指针，new(T)分配了零值填充的T类型内存空间，返回一个*T类型的值

make(T,args)与new(T)的区别：

make只能创建slice,map和channel,并返回一个有初始值(非零)的T类型，而不是*T.make返回初始化后的(非零)值.

{% highlight bash %}
{% raw %}
常用类型变量未填充前的默认值
int     0
int8    0
int32   0
int64   0
uint    .0x0
rune    0 //rune实际类型为int32
byte    0x0 //byte实际是uint8
float32 0 //长度为4 byte
float64 0  //长度为 8byte
bool    false 
string  ""
{% endraw %}
{% endhighlight %}





