---
layout: post
title: "python基本语法"
description: ""
category: python
tags: []
---

### list和tuple

---

#### list是一种有序的集合，可以随时添加和删除其中的元素。

```
student = ['jim','dnzhu','mark','jerry']
len(student) : 集合的大小。
student.append('haha') : 向集合的末尾追加一个元素。
student.insert(1,'luce') : 在集合下标1的位置插入一个元素“luce”。
student.pop() : 删除集合最后一个元素。
student.pop(i) : 删除集合位置i的元素。
```
 集合可以有多维，类似于其它语言中的数组的概念。

#### tuple被称为元祖，tuple和list非常类似，但是tuple一旦初始化就不能修改。

    tuple 没有insert和append方法。tuple中的元素在定义的时候就确定好了。
    注意：只有1个元素的tuple定义时必须加一个逗号,
    t = (1,) ;避免产生歧义。
    tuple中的元素可以是list类型。而list类型中的元素是可以改变的。

```
t = ('a','b',['c','d'])
t[2][0] = 'x'
t[2][1] = 'y'
print(t): ('a','b',['x','y'])
```

---

### dict和set

#### dict
python内置字典，dict全称dictionary 。 采用key-value的存储格式，类似于json的写法。

```
d = {'dnzhu':98,'mark' : 87,'jim':90}
d['dnzhu'] : 98
```

**添加元素：**

    dict['key'] = value
    ps : 一个key只能对应一个value，如果重复的key添加，后面的值会覆盖前面的值。

**判断key是否存在：**

    key in dict     返回布尔值。
    dict.get(key)  返回value或者none。

**删除元素：**
    dict.pop(key)

#### set

跟dict类似，是一组key的集合，但不能存储value。key不能重复，会自动去重。

注意： set的元素是一个集合。

```
s = set(1,2,4)   错误
s = set([1,2,4])  正确
```
**添加：**
    s.add(3)

**删除：**
    s.remove(4)

---

### 条件判断语句

demo1:

```
if xx:
    print('true')
else:
    print('false')
```
1.注意“：”的写法。2.严格的格式缩进“默认4空格”。3.语句末尾没有结尾的符号。

demo2:

```
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>
```

---

### 循环语句

    格式：for x in ...

demo1：

```
names = ['dnzhu','mark','jim']
for name in names :
    print(name)
```

demo2:

```
sum = 0
for i in [1,2,3,4,5,6,7,8,9,10] :
sum = sum + i
print(sum)
```
ps:1.冒号不能丢。空格缩进不能丢。

    input() : 允许用户输入内容。返回一个字符串类型
    int(a) : 将a转换为整型。
    range（101） ： 获取0-100 之间的范围。
    list[range(101)] : 得到一个0-100的list集合。


