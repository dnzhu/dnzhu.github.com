---
layout: post
title: "python 函数"
description: ""
category: python
tags: []
---

> python内置了许多系统函数，直接调用就可以了。要调用一个函数，需要知道函数的名称和参数，但如果是自定义函数，需要注意的就是函数的参数传递。

---

### 内置函数(简单列举)

#### 数学函数：

```
abs() : 获取绝对值。一个参数（int，float）
max() : 获取最大值。接收任意多个参数
min() : 获取最小值。接收任意多个参数
```

#### 数据类型转换函数：

```
int() : 转为int类型。
float() : 转为浮点型。
bool() : 转为布尔型。
str() : 转为字符串型。都是接收一个参数。
isinstance() : 数据类型检查。
demo :
isinstance(x,(int,float)) :判断x是否是int 和float类型。
```

除了使用python内置函数以外，还可以使用自定义函数。

----

### 自定义函数

    自定义函数语法：def 函数名 括号，括号中的参数，冒号，然后缩进块中写函数体。然后return语句返回结果。

```
demo1：
def my_abs(x) :
    if not isinstance(x ,(int ,float)):
        arise TypeError('bad operation type')
    if x >=0:
        return x
    else :
        return -x
```

自定义函数还可以返回多个值

```
demo2:
import math

def move(x, y, step ,angle=0):
nx = x+step*math.cos(angle)
ny = y+step*math.sin(angle)
return nx, ny
```
调用move函数：

```
x ,y = move(100, 100, 60 ,math.pi / 6)
print(x ,y)

```
返回值其实是一个元祖，在语法上，tuple可以省略括号，而多个变量可以接收一个tuple，按位置赋给对应的值。

----

### 函数参数

**1.位置参数**

调用函数时，传入的参数是按照位置顺序依次调用的。

**2.默认参数** 

函数定义的时候，给参数加上默认值，但是默认值参数一定写在参数列表的后面。

ps ： 默认参数必须指向不变对象，str，none等都是写不可变对象。

**3.可变参数**

可变参数就是传入的参数个数是可变的，可变参数的类型是作为一个list或tuple传进来。

    格式 def function_name (* param_name):

函数定义的时候，需要传入可变参数，需要参数名前面加 “*”号。

**4.关键字参数**

关键字参数允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。

    格式  def function_name ( ** param_name):

函数在定义的时候，需要传入关键字参数，需要在参数名的前面加 “**”号。

```
demo ：
def person(name, age ,**kw):
print('name:' , name, 'age:' , age, 'other:', kw)
调用：
extra = {'job':'teacher','city':'beijing'}
person('jim' , 24, ** extra)
```

**5.命名关键字参数**

如果要限制关键字参数的名字，就可以用命名关键字参数。

```
demo :person函数只接收，city和job作为关键字参数，可以如下定义函数

def person（name, age ,* ,city , job）:
print( name ,age ,city , job)

```

如果函数中已经定义了一个可变参数，如果参数列表的后面跟的命名关键字参数就不需要写特殊分隔符“，*，”了。

```
def person ( name ,age , *args, city,job):
print(name ,age ,args, city, job)
```

**6.参数组合**

在函数定义中，可以用位置参数，默认参数，可变参数，关键字参数，命名关键字参数，这几种参数可以组合使用，**但是参数定义的顺序必须是：位置参数，默认参数，可变参数，命名关键字参数，关键字参数**

----

### 函数递归:

    函数递归就是函数内部再调用函数本身。
    使用递归函数的优点是逻辑简单清晰，缺点是过深的调用会导致栈溢出。





