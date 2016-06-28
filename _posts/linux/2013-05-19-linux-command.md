---
layout: post
title: "linux 常用命令手册"
description: ""
category: linux
tags: []
---

### find 查找文件或目录

---

格式：find [路径] [参数选项] 

选项：

* -empty  查找空文件或目录。
* -group  按组查找。
* -name   按名称查找。
* -iname  按名称查找，忽略大小写。
* -mtime  按修改时间查找。
* -size   按大小查找。
* -type   按类型查找，常见类型有：文件（f）,目录（d），设备（b,c）,链接（l）等。
* -user   按用户查找。
* -exec   对查到的文件或目录执行命令。
* -a      并且。
* -o      或者。

demo ：

```
    find -name aaa.html                 //当前目录下查找aaa.html的文件。
    find /usr -name "*.log"             //查找usr目录下以log结尾的文件。
    find /usr -iname Demo.html          //查找usr 目录下 demo.html文件，文件名忽略大小写。
    find / -empty                       //查找根目录中为空的文件或目录
    find / -group  tom                  //查找计算机中所属组为tom的档案。
    find / -mtime -3                    //查找计算机中3天内被修改过的文件。
    find / -mtime +4                    //查找计算机中4天前被修改过的文件。
    find / -mtime 2                     //查找计算机中2天前的当天被修改过的文件。
    find ./ -size +10M                  //查找当前目录下大于10M的文件。
    find ./ -type f                     //查找当前目录下所有的普通文件。
    find ./ -user tom                   //查找当前目录下tom的所有文件。
    find ./ -size 10M -exec ls -l {} \; //查找当前目录下大于1M的文件，并列出详情。
    find / -size 5M -a -type f          //查找计算机中大于5M的所有普通文件。

```

exec解释：
-exec  参数后面跟的是command命令，它的终止是以;为结束标志的，所以这句命令后面的分号是不可缺少的，考虑到各个系统中分号会有不同的意义，所以前面加反斜杠。 {} 花括号代表前面find查找出来的文件名。 

ps : 当执行删除命令的时候，一定要小心，可以使用-exec的安全模式：-ok 。这样在删除操作之前都会有提示。
![](http://pic1.xcarimg.com/img/yongche/2016/0628/2016062810380315077.jpg)

---


