---
layout: article
title:  "Golang学习系列(4)-面向对象编程"
categories: go
disqus: true
image:
    teaser: teaser/oo.jpg
---

{% highlight bash %}
{% raw %}
本文主要讲解Golang的面向对象编程，从关键概念出发，讲解了结构体、method及接口等内容
{% endraw %}
{% endhighlight %} 

---


## 几个关键概念

Go面向对象与其他语言不同之处在于它不支持继承，Go只支持聚合(或称组合)和嵌入。

{% highlight bash %}
{% raw %}
type ColoredPoint struct {
    color.Color  //匿名字段(嵌入)
    x,y int //真名字段(聚合)
}
{% endraw %}
{% endhighlight %}

Golang避开了"类"、"对象"、"实例"，而采用了"类型" 和 "值"，其中自定义类型的值可以包含方法。由于没有继承，因此没有虚函数。Go对此的支持则是采用类型安全的鸭子类型。即参数可以被声明为一个具体类型，也可以是提供了具体满足该接口的方法的值。

继承的一个优点是：有些方法在基类中实现一次，子类即可方便使用。Golang提供了两种解决方案，一种是使用了嵌入(其他语言叫委托delegation)；另一种方案是为每一种类型提供独立的方法,只是简单包装。

Golang面向对象的另一个不同是它的接口、值和方法都相互保持独立。接口用于声明方法签名，结构体用于声明聚合或者嵌入的值，而方法用于声明在自定义类型上的操作。

---

## struct

1.格式:

{% highlight bash %}
{% raw %}
type StructName struct {
    Property1 type1
    ...
}

示例
type Person struct {
    Id int
    Name string
    age int
}

声明使用：
(1)
var P Person
P.Id = 1
P.Name = "xialingsc"
P.age = 32 //这里注意，age为小写开头，某些场景下无法获取该属性

(2) P := Person(1,"xialingsc",32)
(3) P := Persion(Id:1,Name:"xialingsc",age:32)
{% endraw %}
{% endhighlight %}

2.struct匿名字段

上面关键概念中已提及匿名字段，下面是初始化方法

point := ColoredPoint{Color{Red:122,Green:245,Blue:231},x:10,y:20}

所有的内置类型和自定义类型都可以作为匿名字段。

sturct 不仅能将struct作为匿名字段，自定义类型、内置类型都可以作为匿名字段，而且可以在相应的字段上进行函数操作。若内嵌struct中含有与外层相同字段，则最外层的优先访问。这样可以去重载通过匿名字段继承的一些字段。

3.代码示例

{% highlight bash %}
{% raw %}
/**
*
*xialingsc
*时间：2015-3-09
*
**/

package structtest

import (
    "fmt"
 )

type Skills []string

//常规struct定义
type person struct {
   name  string
   age   int
   email string
   phone string
}

//匿名字段(嵌入字段)举例,实现字段的继承
type Student struct {
    person     //匿名字段，默认Student就包含了Human的所有字段
    speciality string
    int               //不仅仅是struct,所有内置类型和自定义类型都可以作为匿名字段
    Skills            //匿名字段，自定义的类型string slice
    phone      string //最外层优先访问，可以利用用着去重载通过匿名字段继承的一些字段
}

func TestStruct() {
    var P person
    P.name = "xialing"
    P.age = 33
    P.email = "xialingsc@gmail.com"
    P.phone = "1234567890"
    //另两种声明方式
    //  P := person{"xial", 33, "xialingsc@163.com", "1234"}
    //  P := person{name: "xialing", age: 33, email: "", phone: ""}
    fmt.Printf("The person's name is %s", P.name)
}

func TestOlderPerson() {
    var p1, p2 person
    p1 = person{"xial", 33, "xialingsc@163.com", "1234"}
    p2 = person{name: "zhuangw", age: 30, email: "", phone: ""}
    olderperson, diff := older(p1, p2)
    fmt.Printf("Of %s and %s,%s is older by %d years", p1.name, p2.name, olderperson.name, diff)
}

func older(p1, p2 person) (person, int) {
    if p1.age > p2.age {
        return p1, p1.age - p2.age
    }
    return p2, p2.age - p1.age
}

func TestNiMingStruct() {
    mark := Student{person{"haim", 2, "haim@163.com", "1987223"}, "baby", 0, []string{"English"}, "99999"}
    fmt.Println("His name is ", mark.name)
    fmt.Println("His age is ", mark.age)
    fmt.Println("His email is ", mark.email)
    fmt.Println("His work phone is ", mark.phone)
    fmt.Println("His home phone is ", mark.person.phone)
    mark.name = "xiaohaim"
    mark.age = 3
    mark.speciality = "children"
    fmt.Println("His name is ", mark.name)
    fmt.Println("His age is ", mark.age)
    fmt.Println("His specility is ", mark.speciality)
    //通过匿名字段访问和修改字段很有用
    mark.person = person{"xiaohaim-1", 4, "163@163.com", "1213"}
    mark.person.age = 5
    fmt.Println("His name is ", mark.name)
    fmt.Println("His age is ", mark.age)
    fmt.Println("His specility is ", mark.speciality)
    mark.int = 3
    fmt.Println("His student's no is ", mark.int)
    mark.Skills = append(mark.Skills, "Chinese", "Japan")
    fmt.Println("His skills is ", mark.Skills)
}

{% endraw %}
{% endhighlight %}


## interface

### 接口示例代码：
{% highlight bash %}
{% raw %}
/**
*接口测试
*interface是一组method的组合，通过interface来定义对象的一组行为
*如果某个对象实现了某个接口的所有方法，则此对象就实现了此接口，无需像java一样通过implements关键字显示指定
*interface到底能存什么值？可以存储实现这个interface的任意类型的对象
*xialingsc
*时间:2015-3-13
 */

 package interfacetest

 import (
    "fmt"
 )

type Human struct {
    name  string
    age   int
    phone string
}

type Student struct {
    Human
    school string
    loan   float32
}

type Teacher struct {
    Human
    salary     float32
    department string
}

//在person上定义了一个method
func (h Human) SayHi() {
    fmt.Printf("Hi,I am %s you can call me on %s\n", h.name, h.phone)
}

func (h Human) Sing(lyrics string) {
    fmt.Println("La la,la la la,....", lyrics)
}

func (h Human) Guzzle(beerStein string) {
    fmt.Println("Guzzle Guzzle...", beerStein)
}

func (t Teacher) SayHi() {
    fmt.Printf("Hi, I am %s,I work at %s.Call me on %s\n", t.name, t.department, t.phone)
}

func (s Student) SayHi() {
    fmt.Printf("This is Student.Sayhi,My name is %s------", s.name)
}


func (s Student) BorrowMoney(amount float32) {
    s.loan += amount
}

func (t Teacher) SpendSalary(amount float32) {
    t.salary -= amount
}

//定义interface
type Men interface {
    SayHi()
    Sing(lyrics string)
    Guzzle(beerStein string)
}

type YoungChap interface {
    SayHi()
    Sing(Song string)
    BorrowMoney(amount float32)

}

type ElderlyGent interface {
    SayHi()
    Sing(song string)
    SpendSalary(amount float32)
}
//根据上面的代码可知
//1.interface 可以被任意的对象实现
//2.Men interface 被 Human、Student、Teacher实现
//3.一个对象也可以实现多个接口，Student即实现了Men、YoungChap接口;Teacher实现了Men、ElderlyGent接口
//4.任意类型都实现了空interface(即 interface{},0个method方法)

func Testinterface() {
    mark := Student{Human{"Mark", 20, "66860989"}, "buaa", 4000.00}
    tom := Student{Human{"Tome", 21, "65860981"}, "buaa", 4000.00}
    jack := Teacher{Human{"Jack", 27, "87654091"}, 1500.89, "technolgy"}
    zhangy := Teacher{Human{"Zhangy", 47, "88768091"}, 6500.00, "technolgy"}
    //定义Men类型的变量i
    var imen Men
    //imen存储Student
    imen = mark
    fmt.Println("\n")
    fmt.Printf("This is Mark,a Student:")
    imen.SayHi()
    imen.Sing("November rain")
    //imen存储Teacher
    imen = jack
    fmt.Printf("This is Jack,a Teacher")
    imen.SayHi()
    imen.Sing("Born to be wild")

    //定义了slice Men
    fmt.Println("print all from slice Men")
    menArray := make([]Men, 3)
    //这三个都是不同类型的元素，但他们都实现了interface同一个接口
    menArray[0] = mark
    menArray[1], menArray[2] = tom, zhang
    for _, elementValue := range menArray {
        elementValue.SayHi()
    }                                                                                                                                                                  
 }
{% endraw %}
{% endhighlight %}

### 空接口测试

{% highlight bash %}
{% raw %}
/**
*空interface 不包含任何的method,所有类型都实现了空interface
*空interface 对于描述起不到任何作用，但在需要存储任意类型的数值时有用，它可以存储任意类型的数值，类似C语言的void*类型
*非常重要的一点：一个函数把interface{}作为参数，那么它可以接受任意类型的值作为参数，如果一个函数返回interface{},则可以返回任意类型的值
*xialingsc
*时间:2015-3-13
 */

 package interfacetest
 import "fmt"
 func TestEmptyInterface() {
    //定义a 为空接口
    var a interface{}
    var i int = 4
    s := "Hi"
    //a可以存储任意类型
    a = i
    fmt.Printf("Ha Ha ,My type is int,the value is %d\n", a)
    a = s
    fmt.Printf("Ha Ha ,My type is string,the value is %s\n", a)
}
{% endraw %}
{% endhighlight %}

###interface函数参数测试

{% highlight bash %}
{% raw %}
/**
*interface的变量可以持有任意实现该interface类型的对象
*(1)通过定义interface参数，让函数接受各种类型的参数，例如fmt.Println方法,任何实现了String()方法的都能作为参数被fmt.Println调用
* 如果某个类型需要按特殊的格式输出，就必须实现这个Stringer接口
*(2)interface变量存储类型查看
*
*xialingsc
*时间:2015-3-13
 */

 package interfacetest

 import (
    "fmt"
    "strconv"
 )

type PHuman struct {
    name  string
    age   int
    phone string
}

//通过这个方法Human实现了Fmt.Stringer接口
//不实现方法，打印出来为This Human is: {Bob 39 00-86-109121}
//实现了，打印出来为This Human is: name:Bob-39years -00-86-109121---
func (h PHuman) String() string {
    return "name:" + h.name + "-" + strconv.Itoa(h.age) + "years -" + h.phone + "---"
}

func TestInterfaceParameter() {
    Bob := PHuman{"Bob", 39, "00-86-109121"}
    fmt.Println("This Human is:", Bob)
}

//-----------------------------------------
//(2)interface存储类型查看
//value,ok = element.(T),这个value就是变量的值，ok是一个bool类型，element是interface
//如果element里面存储了T类型的值，ok为true,否则ok为false
type Element interface{}
type List []Element

func TestInterfaceValueType() {
    list := make(List, 3)
    list[0] = 1       //an int
    list[1] = "hello" //a string
    list[2] = PHuman{"face", 10, "7654321"}
    for index, element := range list {
    //if value,ok := element.(int);ok{//comma-ok形式，功能同下
        switch value := element.(type) { //element.(type)不能在switch之外使用，在switch之外使用，必须使用comma-ok
            case int:
                fmt.Printf("list[%d] is an int and its value is %d\n", index, value)
             case string:
                fmt.Printf("list[%d] is an string and its value is %s\n", index, value)
            case PHuman:
                fmt.Printf("list[%d] is an PHuman and its value is %s\n", index, value)
            default:
                fmt.Printf("list[%d] is of a different type", index)
        }
    }
}

{% endraw %}
{% endhighlight %}

### 嵌入式interface测试

{% highlight bash %}
{% raw %}
/**
*嵌入式interface表示为如果一个interface1作为interface2的一个嵌入字段，
*那么interface2隐式的包含了interface1里面的method
*
*xialingsc
*时间:2015-3-13
*
 */

 package interfacetest

 import (
    "fmt"
    "io"
    "sort"
 )

type Interface interface {
    sort.Interface      //嵌入字段sort.Interface,把sort.Interface的所有method{即Len(),Less(i,j int),Swap(i,j int)}都包含进来
    Push(x interface{}) // a Push method to push elements into the heap
    Pop() interface{}   //a Pop elements that pops elements from the heap
}

type ReadWriter interface {
    io.Reader
    io.Writer
}

type Ceshi struct {
    lenght int
}

func (c Ceshi) Len() int {
    fmt.Println("The len of Ceshi")
    return 0
}

func (c Ceshi) Less(i, j int) bool {
    if i > j {
        return true
    }
    return false
}

func (c Ceshi) Swap(i, j int) {
}

func (c Ceshi) Push(x interface{}) {
}

func (c Ceshi) Pop() interface{} {
    return 4
}

func TestEmbededInterface() {
    var i Interface
    var c Ceshi = Ceshi{10}
    i = c
    fmt.Printf("This len of Interface:%d\n", i.Len())
}

{% endraw %}
{% endhighlight %}






























