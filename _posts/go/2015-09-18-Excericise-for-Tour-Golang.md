---
layout: article
title:  "Golang学习系列(5)-Golang学习向导练习题"
categories: go
disqus: true
image:
    teaser: teaser/golang6.png
---

{% highlight bash %}
{% raw %}
本文主要记录了在tour.golang.org上练习的相关题目，也学习了一些别的同学相关写法，今天将其总结下来，希望自己能多多提高
{% endraw %}
{% endhighlight %} 

---


## sqrt函数

目标是利用牛顿公式实现类似开方功能，牛顿公式为z = z - (z*z -x)/(2*z)。最开始我写的是这样
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
)

func Sqrt(x float64) float64{
    var z float64 = float64(x)
    for i := 0 ;i<10; i++{ //当然这里10也可以改为100或1000等
        z = z - (z*z - x)/(2*z)
    }
    return z
}


func main(){
    fmt.Println(Sqrt(2))
}

--------
1.5
1.4375
1.4208984375
1.4161603450775146
1.4147828143349983
1.4143802114005837
1.4142623658001936
1.4142278559705035
1.4142177488197718
1.414214788550556
1.414214788550556

{% endraw %}
{% endhighlight %}
后来，觉得这样写需要有精度控制改为了
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "math"
)

func Sqrt(x float64) float64{
    z := float64(x)
    s := float64(0)
    for{
        z = z - (z*z - x)/(2*z)
        if math.Abs(s-z)<1e-10{ //返回1.414213562439504 ,若改为1e-15则为1.4142135623730963
            break
        }
        s = z
    }
    return s
}


func main(){
    fmt.Println(Sqrt(2))
}
{% endraw %}
{% endhighlight %}


## Slices练习

该函数是一个画图函数，要求一个拥有dy长度的切片，且切片中dx的每个元素为一个8位无符号整数。同时告诉我们说，在运行这个程序成功后，会显示一个蓝色的图形，具体图像取决于你使用的功能，比如(x+y)/2,x*y以及x^y。

{% highlight bash %}
{% raw %}
package main

import (
    "golang.org/x/tour/pic"
)

func Pic(dx,dy int) [][]uint8 {
    var ret = make([][]uint8 ,dy)
    for i := 0;i < dy;i++{
        ret[i] = make([]uint8 ,dx)
        for j := 0;j < dx;j++ {
            ret[i][j] = uint8(dx^dy +(dx+dy)/2)
        }
    }
    return ret
}

func main(){
    pic.Show(Pic)
}
{% endraw %}
{% endhighlight %}

## Maps练习

该函数实现数单词个数的功能，它应该返回一个包含字符串中单词个数的map.
{% highlight bash %}
{% raw %}
package main

import (
    "golang.org/x/tour/wc"
    "strings"
)

func WordCount(s string) map[string]int{
    vmap := make(map[string]int)
    strs := strings.Fields(s)
    length := len(strs)
    for i := 0;i<length;i++ {
        vmap[strs[i]] =vmap[strs[i]] + 1
        //(vmap[strs[i]])++ //看到有同学是这么写的，个人觉得这样写更简洁
    }
    return vmap
}

func main(){
    wc.Test(WordCount)
}
{% endraw %}
{% endhighlight %}

## 函数练习

学习中隐藏函数作为返回值这块挺有意思，记录了。
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
)

func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}


func main(){
    pos,neg := adder(),adder()
    for i := 0;i<10;i++ {
        fmt.Println(pos(i),neg(-2*i),)
    }
}

{% endraw %}
{% endhighlight %}

求解fibonacci数列。

咱们首先看一下fibonacci数列的定义

        / 0                  n = 0
f(n) = -  1                  n = 1
        \ f(n-1)+f(n+2)      n >2

正常情况下或教科书上，遇到此类问题常规办法可以采用递归解决，但递归也不是最好的解决办法。下面借鉴了一个取巧的办法。

{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
)

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
    x := 0
    y := 1
    return func() int {
            x,y = y,x+y
            return x
            }
}

func main(){
    f := fibonacci()
    for i := 0;i < 10 ;i++ {
        fmt.Println(f())
    }
}

{% endraw %}
{% endhighlight %}


## Method练习

Go中没有类，但可以在struct类型上定义方法或者包内的任何类型上，但无法定义在一个其他包中已定义的类型上。定义在struct类型上的方法参数来源于receiver。

Method就涉及到interface,这部分可以细致看看。

{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
)

type IPAddr [4]byte


// TODO: Add a "String() string" method to IPAddr.
func (ip IPAddr)String() string {
    //return fmt.Sprintf("%v.%v.%v.%v",ip[0],ip[1],ip[2],ip[3])
    var ipaddr string
    ipaddrLen := len(ip)
    for i := 0;i < ipaddrLen; i++ {
        ipaddr += fmt.Sprintf("%v.",ip[i])
    }
    ipaddr = ipaddr[:len(ipaddr)-1] 
    return ipaddr
}

func main(){
    addrs := map[string]IPAddr{
        "loopback":{127,0,0,1},
        "googleDNS":{8,8,8,8},
    }
    for n,a := range addrs {
        fmt.Println("%v: %v\n",n,a)
    }
}

{% endraw %}
{% endhighlight %}

## Errors练习

error类型是一个interface,即
{% highlight bash %}
{% raw %}
type error interface {
    Error() string
}
{% endraw %}
{% endhighlight %}

只要实现了Error() string方法,也就实现了error接口。
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
   "math"
 )

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {
       return fmt.Sprintf("cannot negative number:%g",float64(e))
}

func Sqrt(x float64) (float64, error) {
       if x <= 0 {
            return 0 , ErrNegativeSqrt(x)
      } else {
            z := float64(2.)
            s := float64(0)
            for {
                 z = z - (z*z - x)/(2*z)
                 if math.Abs(s-z) < 1e-15 {
                  break
             }
           s = z
    }
    return s, nil
 }
}

func main() {
     fmt.Println(Sqrt(2))
     fmt.Println(Sqrt(-2))
}
{% endraw %}
{% endhighlight %}

## Readers

Go标准库中包括了很多实现，例如files,network connections,compressors,ciphers,and others.io.Reader的语法为:
{% highlight bash %}
{% raw %}
type Reader interface {
    func (T) Read(b []byte) (n int,err error)
}
{% endraw %}
{% endhighlight %}

下面是一个读取字符串的例子，每次输出8个字节
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "io"
    "strings"
)

func main(){
    r := strings.NewReader("Hello,Reader!")
    b := make([]byte,8)
    for {
        n, err := r.Read(b)
        fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
        fmt.Printf("b[:n] = %q\n", b[:n])
        if err == io.EOF {
              break
        }
    }
}
{% endraw %}
{% endhighlight %}

输出：
n = 8 err = <nil> b = [72 101 108 108 111 44 32 82]
b[:n] = "Hello, R"
n = 6 err = <nil> b = [101 97 100 101 114 33 32 82]
b[:n] = "eader!"
n = 0 err = EOF b = [101 97 100 101 114 33 32 82]
b[:n] = ""


下面是实现一个本节的练习
{% highlight bash %}
{% raw %}
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

func (r MyReader) Read(b []byte) (n int,e error) {
    b[0] = 'A'
    return 1,nil
 }

func main() {
    reader.Validate(MyReader{})
 }
{% endraw %}
{% endhighlight %}

本节第二个练习时，需要先搞懂rot13算法的含义
{% highlight bash %}
{% raw %}
package main

import (
    "io"
     "os"
     "strings"
 )

type rot13Reader struct {
     r io.Reader
}

func (rot *rot13Reader) Read(p []byte) (n int, err error) {
      n,err = rot.r.Read(p)
      for i := 0; i < len(p); i++ {
         if (p[i] >= 'A' && p[i] < 'N') || (p[i] >='a' && p[i] < 'n') {
          p[i] += 13
        } else if (p[i] > 'M' && p[i] <= 'Z') || (p[i] > 'm' && p[i] <= 'z'){
            p[i] -= 13
       }
    }
    return
}

func main() {
    s := strings.NewReader("Lbh penpxrq gur pbqr!")
    r := rot13Reader{s}
    io.Copy(os.Stdout, &r)
}
{% endraw %}
{% endhighlight %}


## Web Servers

Package http可以适用于任何实现了http.Handler接口的http请求。
{% highlight bash %}
{% raw %}
package http
type Handler interface {
    ServeHttp(w ResponseWriter , r *Request)
}
{% endraw %}
{% endhighlight %}

下面是具体是请求实例
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "log"
    "net/http"
)

 type Hello struct{}

func (h Hello) ServeHTTP(w http.ResponseWriter,r *http.Request) {
     fmt.Fprint(w, "Hello!")
}

func main() {
    var h Hello
    err := http.ListenAndServe("localhost:4000", h)
    if err != nil {
      log.Fatal(err)
    }
}
{% endraw %}
{% endhighlight %}

下面是练习样例
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "log"
    "net/http"
)

type String string

type Struct struct {
       Greeting string
       Punct    string
       Who      string
}

func (s String) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       fmt.Fprint(w, "I'm a frayed knot.")
}

func (s Struct) ServeHTTP(w http.ResponseWriter, r *http.Request) {
       fmt.Fprint(w, s.Greeting, s.Punct, s.Who)
}

func main() {
    http.Handle("/string", String("I'm a frayed knot."))
    http.Handle("/struct", &Struct{"Hello", ":", "Gophers!"})
    log.Fatal(http.ListenAndServe("localhost:4000", nil))
}

{% endraw %}
{% endhighlight %}

## Images

Image也是一个接口，具体内容如下：
{% highlight bash %}
{% raw %}
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
{% endraw %}
{% endhighlight %}

Image使用简单示例
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "image"
)

func main() {
    m := image.NewRGBA(image.Rect(0, 0, 100, 100))
    fmt.Println(m.Bounds())
    fmt.Println(m.At(0, 0).RGBA())
}
{% endraw %}
{% endhighlight %}


下面是一个练习Image的示例
{% highlight bash %}
{% raw %}
package main

import (
    "golang.org/x/tour/pic"
    "image"
    "image/color"
)

type Image struct{
     Width,Height int
     color uint8
}

func (i *Image) Bounds() image.Rectangle{
      return image.Rect(0, 0, i.Width, i.Height)
}

func (i *Image) ColorModel()  color.Model {
      return color.RGBAModel
}

func (i *Image)At(x,y int)  color.Color {
      return color.RGBA{i.color+uint8(x), i.color+uint8(y), 255, 255}
}

func main() {
    m := Image{100,100,128}
    pic.ShowImage(&m)
}

{% endraw %}
{% endhighlight %}

## Goroutines

通过Go runtine管理的goroutine是一个轻量级的线程，通过go f(x,y,z)启动一个新的Go程，Goroutines在同样的地址空间运行，因此获取共用的内存时必须同步。sync是很有用的原生包。

### channel

channels是通过<-符号进行发送和接收值的管道类型，ch <- v表示发送v到channel ch;v := <-ch表示从ch接收并赋值给局部变量v,箭头表示了数据流的方向。同map与slice一样，channel必须在使用前通过make创建,即 ch := make(chan int)。默认情况下，发送和接收均是阻塞的直到另一端已经就绪。在没有明确的锁及条件变量情况下运行goruntine进行同步,下面channel常规使用示例：

{% highlight bash %}
{% raw %}
package main

import "fmt"

func sum(a []int, c chan int) {
    sum := 0
    for _, v := range a {
         sum += v
    }
   c <- sum // send sum to c
}

func main() {
    a := []int{7, 2, 8, -9, 4, 0}

    c := make(chan int)
    go sum(a[:len(a)/2], c)
    go sum(a[len(a)/2:], c)
    x, y := <-c, <-c // receive from c

    fmt.Println(x, y, x+y)
}
-----------------
-5 17 12 //这是在tour.golang.org中执行的结果
17 -5 12 //这是在自己本机执行的结果

分析：主程序执行a，c声明后，开启第一个分支A和第二个分支B，主程序继续向下执行，发现准备接收c时，
c目前为空，即阻塞进入等待；与此同时进行的分支A与分支B开始执行，返回了不同的c,这里有意思的是，
同样的程序运行时出现了上述两种不一样的结果，自己感觉goruntine无法保证运行结果一致性与机器的运
行速度有关。
{% endraw %}
{% endhighlight %}


示例二

{% highlight bash %}
{% raw %}
package main

import (
        "fmt"
)

var a string
var c = make(chan int)

func fun() {
   c <- 0
   a = "hello world"
}

func main() {
   go fun()
   fmt.Println(a)
   <-c
}
---------------
输出为空
说明：主线在执行Print时，支线在执行`c <- 0`，所以a还是空的，打印是空.
若将<-c 与 print打印互换，主线是阻塞状态，必须等管道c里有内容后才会继续执行,所以支线的`c <- 0`执行后，主线的`<-c`才会执行,主线`<-c`执行时，支线正在执行`a = "hello world"`然后主线就能正确打印出hello world了。
{% endraw %}
{% endhighlight %}


使用缓存Buffer的格式，buffer的长度为make声明时的第二个初始化参数，ch := make(chan int, 100)。只有buffer被占满时，发送给buffer channel才会被阻塞；只有buffer是空时，接收才会被阻塞。
{% highlight bash %}
{% raw %}
package main

import "fmt"

func main() {
    ch := make(chan int, 2)
    ch <- 1
    ch <- 2
    fmt.Println(<-ch)
    fmt.Println("---分割线---")
    fmt.Println(<-ch)
}

------------
1
---分割线---
2
{% endraw %}
{% endhighlight %}

发送者在没有值等待发送时可以关闭channel,接收者可以通过第二个标志符测试是否channel已经关闭，例如 v ,ok := <-ch;若没有更多的值被接收或者channel已经关闭，则ok为false.可以利用for i :=rang c 格式进行循环接收，直到c中没有需要再被传送的值了。只有发送者能关闭channel,接收者不能关闭，若发送过程中关闭channel，会引起一个恐慌panic。channel不像文件，通常情况下你不必关闭它，只有在接收者明确被告知不会有新值被传送时才有必要关闭，例如某个range循环结束。
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
)

func fibonacci(n int, c chan int) {
    x, y := 0, 1
    for i := 0; i < n; i++ {
        c <- x
        x, y = y, x+y
    }
    close(c)
}

func main() {
    c := make(chan int, 10)
    go fibonacci(cap(c), c)
    for i := range c {
         fmt.Println(i)
    }
}

-------------------
0
1
1
2
3
5
8
13
21
34


{% endraw %}
{% endhighlight %}


### Select

select可以监听channel上的数据流动，它默认是阻塞的，只有当监听的channel中发送或接收可以进行时才会运行，当多个channel都准备好的时候，将随机选择其一执行。select里面还有default语法，default就是当监听的channel都没有准备好的时候默认执行的。
{% highlight bash %}
{% raw %}
package main

import (
    "fmt"
    "strconv"
)

func fibonacci(c, quit chan int) {
     x, y := 0, 1
     for {
         fmt.Println("=========")
         select {
              case c <- x:
                fmt.Println("00000000")
                x, y = y, x+y
                fmt.Println("1111111111")
              case <-quit:
                fmt.Println("quit")
                return
         }
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
        for i := 0; i < 10; i++ {
            fmt.Println("---i----" + strconv.Itoa(i))
            fmt.Println(<-c)
            fmt.Println("----j---")
        }
        quit <- 0
    }()
    fibonacci(c, quit)
}

-------------------
/Users/xialing/go-test2/src/src  [/Users/xialing/go-test2/src]
=========
---i----0
0
----j---
---i----1
00000000
1111111111
=========
00000000
1111111111
=========
1
----j---
---i----2
1
----j---
---i----3
00000000
1111111111
=========
00000000
1111111111
=========
2
----j---
---i----4
3
----j---
---i----5
00000000
1111111111
=========
00000000
1111111111
=========
5
----j---
---i----6
8
----j---
---i----7
00000000
1111111111
=========
00000000
1111111111
=========
13
----j---
---i----8
21
----j---
---i----9
00000000
1111111111
=========
00000000
1111111111
=========
34
----j---
quit

这里为了详细了解该执行过程，在相关地方进行了后台输出。
执行过程说明:主程序执行到fib时，输出=========,这时分支程序输出---i---0,主分支中执行`c<-x`后，分支程序即可输出0,后又输出---j---,分支程序继续运行，输出---i---1,遇到<-c进入等待，主程序继续往下执行case中的语句，输出00000000，执行`x, y = y, x+y`,输出1111111111后结束第一次for循环执行；主程序进入第二次for循环并输出=========,遇到c<-x赋值后继续执行，输出00000000，执行`x, y = y, x+y`,输出1111111111后,进入第三次for循环输出==========，此时分支程序获得c，输出1，分支程序循环输出---i---2...

这里的问题是，在第二次for循环遇到c<-x时，为啥分支程序没有及时打印c,而是要输出00000000，
执行`x, y = y, x+y`,输出1111111111后？
{% endraw %}
{% endhighlight %}

