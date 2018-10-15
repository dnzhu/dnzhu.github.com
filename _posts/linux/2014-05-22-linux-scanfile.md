---
layout: post
title: "linux 查看文件常用命令"
description: ""
category: linux
tags: []
---

### 1.cat 查看文件内容。

这个命令常用来显示文件内容，或者将几个文件连接起来显示，或者从标准输入读取内容并显示，它常与重定向符号配合使用。 

格式 ： cat [-command] filename

command :

    -A, --show-all           等价于 -vET

    -b, --number-nonblank    对非空输出行编号

    -e                       等价于 -vE

    -E, --show-ends          在每行结束处显示 $

    -n, --number     对输出的所有行编号,由1开始对所有输出的行数编号

    -s, --squeeze-blank  有连续两行以上的空白行，就代换为一行的空白行 

    -t                       与 -vT 等价

    -T, --show-tabs          将跳格字符显示为 ^I

    -u                       (被忽略)

    -v, --show-nonprinting   使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外

功能 ： 

    1.一次显示整个文件:cat filename

    2.从键盘创建一个文件:cat > filename 只能创建新文件,不能编辑已有文件.

    3.将几个文件合并为一个文件:cat file1 file2 > file

demo1 : 把file1.log的文件内容加上行号
(包含空行)后输入file2.log这个文件里

```
cat -n file1.log file2.log 
```

demo2 : 把 file1.log 和 file1.log 的文件内容加上行号（空行不加）之后将内容附加到 log.log 里

```
cat -b file1.log file2.log log.log
```

demo3 : 使用here doc来生成文件

```
cat >log.txt <<EOF
 > hello 
 > world
 > linux
 > PWD=$(pwd)
 >EOF

cat log.txt
 hello 
 world
 linux
 PWD=/tmp/runtime/

```
----

### 2.more命令文件内容。

more和cat的功能一样都是查看文件里的内容，但有所不同的是more可以按页来查看文件的内容，还支持直接跳转行等功能。

格式： more [-command] file

command : 
    
    +n  从笫n行开始显示

    -n  定义每屏为多少行

    +/pattern   在每个档案显示前搜寻该字串(pattern),然后从该字串前两行之后开始显示


常用操作命令 ：

    Ctrl+F  向下滚动一屏

    空格键  向下滚动一屏

    Ctrl+B  返回上一屏

    =       输出当前行的行号

    !命令   调用Shell，并执行命令 

    q       退出more


demo :  每屏显示3行

```
more -3 log2014.log

    2014-01
    2014-02
    2014-03
    
```
----

### 3.less 命令也是对文件或其它输出进行分页显示的

使用 [pageup]或[pagedown] 等按键的功能来往前往后翻看文件，也可以对文件进行向上或向下搜索。

格式 ：less [-command] filename

command : 
    
    -b <缓冲区大小> 设置缓冲区的大小

    -e  当文件显示结束后，自动离开

    -f  强迫打开特殊文件，例如外围设备代号、目录和二进制文件

    -g  只标志最后搜索的关键词

    -i  忽略搜索时的大小写

    -m  显示类似more命令的百分比

    -N  显示每行的行号

    -o <文件名> 将less 输出的内容在指定文件中保存起来

    -s  显示连续空行为一行

    -S  行过长时间将超出部分舍弃

    -x <数字> 将“tab”键显示为规定的数字空格

常用操作命令：

    /字符串：向下搜索“字符串”的功能

    ?字符串：向上搜索“字符串”的功能

    n：重复前一个搜索（与 / 或 ? 有关）

    N：反向重复前一个搜索（与 / 或 ? 有关）

    b  向后翻一页

    d  向后翻半页

    Q  退出less命令

    u  向前滚动半页

    y  向前滚动一行

    空格键 滚动一行

    回车键 滚动一页

    [pagedown]： 向下翻动一页

    [pageup]：   向上翻动一页


其它有用的命令

    v - 使用配置的编辑器编辑当前文件

    h - 显示 less 的帮助文档

    &pattern - 仅显示匹配模式的行，而不是整个文件

demo1 : ps查看进程信息并通过less分页显示 

```
ps -ef | less 
```

demo2 : history 通过less分页显示 

```
history | less 
```

----

### 4.head 查看文件内容

head命令用来显示开头或结尾某个数量的文字区块。默认显示前10行。

格式 ：head [-command] filename

command ： 
    
    -q 隐藏文件名

    -v 显示文件名

    -c<字节> 显示字节数

    -n<行数> 显示的行数

demo1 : 显示文件的前n行

```
head -n 5 log2014.log
 2014-01

 2014-02

 2014-03

 2014-04

 2014-05
```

demo2 : 显示文件前n个字节

```
head -c 20 log2014.log
 2014-01

 2014-02

 2014
```

demo3 : 显示文件除了最后n行的全部内容

```
head -n -10 log2014.log
2014-01

2014-02

2014-03
```
---- 

### 5.tail 从指定点开始将文件写到标准输出

tail命令的-f选项可以方便的查阅正在改变的日志文件,并且不但刷新,可以查看到最新的日志内容。

格式 : tail [-command] filename

command : 

    -f 循环读取

    -q 不显示处理信息

    -v 显示详细的处理信息

    -c<数目> 显示的字节数

    -n<行数> 显示行数

    --pid=PID 与-f合用,表示在进程ID,PID死掉之后结束. 

    -q, --quiet, --silent 从不输出给出文件名的首部 

    -s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒

demo1 : 显示文件末尾n行内容 

```
tail -n 5 log2014.log 
 2014-09

 2014-10

 2014-11

 2014-12
```

demo2 : 循环查看文件内容

```
tail -f log.log
```

demo3 : 从第5行开始显示文件

```
tail -n +5 log2014.log
```
ps: tail 命令的主要用途还是用于时时查看日志文件。






