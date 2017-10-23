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

### du 计算文件或目录的容量

格式：du [选项] [文件或目录]

选项：

* -h 人性化的显示容量信息。
* -s 仅显示总容量。
* -b 以字节数显式容量信息。

demo ：

```
du -sh /root        //查看root所占的磁盘容量
du -b hello.html    //查看hello.html 文件的字节数
```

### 查看文件内容(cat,more ,less,head, tail)

demo :

```
cat -b hello.html   //忽略空白行，显示行号
cat -n hello.html   //显示行号，包括空行

more hello.html     //通过空格键查看下一页
less hello.html     //通过方向键上下翻页

head -c 20k  hello.html     //显示hello文件前20k的内容
head -20  hello.html        //显示hello文件前20行内容

tail -c 20k hello.html      //显示hello文件末尾20k的内容
tail -20 hello.html         //显示hello文件末尾20行的内容
*tail -f hello.html          //动态的显示文件的内容 

wc -l hello.html            //显示文件的行数
wc -c hello.html            //显示文件的字节数
wc -w hello.html            //显示文件的单词个数

```

### grep 查找关键词并打印出匹配行

格式：grep [选项] 匹配模式 [文件]

选项：

* -i 忽略大小写。
* -v 取反匹配。
* -w 匹配单词。
* --color 显示颜色。

demo：

```
grep love hello.html            //在hello文件中查找love，并将匹配出的内容打印出来。
grep --color love hello.html    //对匹配的关键词显示颜色
grep -i like hello.html         //忽略大小写匹配
grep -v love hello.html         //不匹配love的行

```

### ln 链接文件

格式： ln [选项] [文件]

demo ： 

```
ln -s /usr/hello.html  /tmp/runtime/hello.html      //创建软链接文件
ln -s /usr/software/    /tmp/software               //创建软链接目录

ln /usr/hello.html  /tmp/test/hello.html            //创建硬链接文件

```

### tar 打包与解包文件

参数：

* -c    创建打包文件
* -r    追加文件到打包文件。
* -t    列出打包文档的内容。
* -x    释放打包文件。
* -C    指定解压路径。
* -f    指定打包后的名称。
* -j    通过bzip2格式解压。
* -z    通过gzip格式解压。
* --remove-files    打包后删除源文件。

demo1(打包)：

```
tar czvf my.tar file1   //单个文件压缩打包 

tar czvf my.tar file1 file2,...     //多个文件压缩打包 

tar czvf my.tar dir1        //单个目录压缩打包 

tar czvf my.tar dir1 dir2      //多个目录压缩打包

```

demo2(解压)：

```
tar -zxvf   bootstrap.tar.gz        //解压gzip文件
tar -jxvf   bootstrap.tar.bz2       //解压bzip2文件

```

### rsync 推送的命令

rsync命令是基于rsync服务，rsync服务不是独立的服务，而是基于**xinetd**。所以要使用该命令，必须在服务器上安装xinetd服务。

---

### ps 查看系统自启动服务

```
ps aux 查看系统自启动服务
```
---

### netstat 查看系统开启了哪些服务

* -t 列出tcp服务
* -u 列出udp服务
* -l 列出正在监听的网络服务
* -n 用端口号来显示服务
* -p 列出该服务的进程ID

```
netstat -tlunp   
```
---

### mount与umount

    mount   查看挂载硬件

    mount /dev/sr0  /mnt/cdrom/     挂载硬件

---


{% include JB/setup %}




