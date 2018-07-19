---
layout: post
title: "go 基本功"
description: ""
category: go
tags: [go]

---

    1.每个代码文件都以package 关键字开头，随后跟着包的名字。标准库中的包，只需要给出包名，编译器会自动到GOROOT和GOPATH环境变量所在位置去查找。

    2.在Go语言里，标识符要么从包里公开，要么不从包里公开。当代码导入了一个包时，程序可以直接访问这个包中任意一个公开的标识符。这些标识符以大写字母开头。以小写字母开头的标识符是不公开的，不能被其他包中的代码直接访问。

    3.在Go语言中，所有变量都被初始化为其零值。对于数值类型，零值是0；对于字符串类型，零值是空字符串；对于布尔类型，零值是false；对于指针，零值是nil。对于引用类型来说，所引用的底层数据结构会被初始化为对应的零值。但是被声明为其零值的引用类型的变量，会返回nil 作为其值。

---

### 函数

    Go语言使用关键字func声明函数，关键字后面紧跟着函数名、参数以及返回值。

    变量声明运算符（:=）。这个运算符用于声明一个变量，同时给这个变量赋值。这个运算符声明的变量和其他使用关键字var 声明的变量没有任何区别。
    
    在Go语言中，通道（channel）和映射（map）与切片（slice）一样，也是引用类型，不过通道本身实现的是一组带类型的值，这组值用于在goroutine之间传递数据。
    
    指针变量可以方便地在函数之间共享数据。使用指针变量可以让函数访问并修改一个变量的状态，而这个变量可以在其他函数甚至是其他goroutine 的作用域里声明。
    
    Go语言支持闭包，函数可以直接访问到那些没有作为参数传入的变量。匿名函数并没有拿到这些变量的副本，而是直接访问外层函数作用域中声明的这些变量本身。
    
    关键字 defer 会安排随后的函数调用在函数返回时才执行。defer可以保证函数一定会被调用。哪怕函数意外崩溃终止，也能保证关键字defer安排调用的函数会被执行。
    
    interface关键字声明了一个接口，这个接口声明了结构类型或者具名类型需要实现的行为。一个接口的行为最终由在这个接口类型中声明的方法决定。

    如果声明函数的时候带有接收者，则意味着声明了一个方法。这个方法会和指定的接收者的类型绑在一起。

---

### 值类型与指针类型的不同

```
//申明一个方法，指定defaultMatcher类型的值作为接收者
func (m defaultMatcher) Search (feed *Feed, searchTerm string)
//申明一个方法，指定defaultMatcher类型的指针作为接收者
func (m *defaultMatcher) Search (feed *Feed, searchTerm string)
```

两者区别：

    使用指针作为接收者声明的方法，只能在接口类型的值是一个指针的时候被调用。使用值作为接收者声明的方法，在接口类型的值为值或者指针时，都可以被调用。

---

### 数组

    数组是一个长度固定的数据类型，用于存储一段具有相同的类型的元素的连续块。数组存储的类型可以是内置类型，如整型或者字符串，也可以是某种结构类型。

**声明和初始化数组**

```
var arr [5]int    //初始化为零值
arr := [5]int{1,2,3,4,5}    //初始化并赋值
arr := [...]int{1,2,3,4,5}  //容量由初始化值决定
arr := [5]int{1:10,2:20}  //指定某个值
```
---

### 切片的实现和使用

**申明和初始化切片**

```
arr := []int   //空切片
var arr make([]string, 5)  //切片的长度和容量为5
arr := make([]string, 5, 6)   //切片长度为5,容量为6
```
    ps1:不管是使用 nil 切片还是空切片，对其调用内置函数append、len 和cap 的效果都是一样的。

    ps2:切片的容量不能小于切片长度。切片和数组区别在于，数组固定长度，切片不固定长度


**使用切片创建切片**

```
slice := []int{10, 20, 30, 40, 50}
newSlice := slice[1:3]
newSlice = append(newSlice,90)
len := len(newSlice)  // len=2
cap := cap(newSlice)  // cap=4 

otherSlice := slice[2:3:4]
len := len(otherSlice)  // len=1
cap := cap(otherSlice)  // cap=2 
```

    对底层数组容量是k 的切片slice[i:j]来说, 长度: j - i 容量: k - i

    函数append 会智能地处理底层数组的容量增长。在切片的容量小于1000 个元素时，总是会成倍地增加容量。一旦元素个数超过1000，容量的增长因子会设为1.25，也就是会每次增加25%的容量。

**将一个切片追加到另一个切片**

```
s1 := []int{1,2}
s2 := []int{3,4}
fmt.Printf("%v\n", append(s1, s2...))
```

**迭代切片**

```
slice := []int{10,20,30,40}

for k,v := range slice {
    fmt.Printf("index:%d,value:%d\n",k,v)
}
```

**创建二维切片**

```
slice := [][]int{{10}, {100, 200}}
slice[0] = append(slice[0],20)
fmt.Printf("%v\n", slice[0])
```
---

### 映射的内部实现和基础功能

    映射是一个存储键值对的无序集合。之所以是无序的，因为映射的底层实现是用的散列表。

**创建和初始化**

```
//make初始化
dict := make(map[string]int)
//字面量初始化
dict := map[string]string{"red":"#ffffff","black":"#000000"}

//使用切片作为map的值
dict := map[int][]string{}
```

**map使用**

```
colors := map[string]string{"red": "#ffffff", "blue": "#cccccc"}
value, isTure := colors["red"]
if isTure {
    fmt.Printf("val1:%s\n", value)
}

val := colors["blue"]
if val != "" {
    fmt.Printf("val2:%s \n", val)
}

delete(colors, "red")   //从map中删除某个key

```