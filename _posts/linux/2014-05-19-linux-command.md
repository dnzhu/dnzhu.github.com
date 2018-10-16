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

### nl 统计文件的行号

nl 命令类似于cat -n 命令，但是比cat -n 更强大，他能将行号用0 填充，并且可以指定行号位数。

格式 ：nl [-command] file

command ：

    -b a : 表示不论是否为空行，也同样列出行号(类似 cat -n);
    
    -b t : 如果有空行，空的那一行不要列出行号(默认值);
    
    -n ln : 行号在萤幕的最左方显示;

    -n rn : 行号在自己栏位的最右方显示，且不加 0 ;

    -n rz : 行号在自己栏位的最右方显示，且加 0 ;

    -w : 行号栏位的占用的位数(默认是6位);

    -p : 在逻辑定界符处不重新开始计算。 

demo : 过滤空行，行号最右侧显示并且占用3位

```
 nl -b t -n rz -w 3 2015.04.01.log

   001   -------
   002   XXXXXX

   003   -------
   ……

```

----

### which查找可执行文件的位置

which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。

格式： which [-param] command

参数：
    
    -n 　指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。

    -p 　与-n参数相同，但此处的包括了文件的路径。

    -w 　指定输出时栏位的宽度。

    -V 　显示版本信息。

区别： 

    which  查看可执行文件的位置。
    
    whereis 查看文件的位置。 
    
    locate   配合数据库查看文件位置。
    
    find   实际搜寻硬盘查询文件名称。

demo1：用 which 找出 which 

```
which which
    
    alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot  --show-tilde'

    /usr/bin/which
```

demo2 : which cd 
cd是bash的内建命令，在path目录中找不到。

```
which cd 
    
    /usr/bin/which no cd in ($PATH)
```

----

### whereis查找文件命令

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。没有参数返回所有信息。比find的效率要高。

格式 : whereis [-param] command

参数：

    -b   定位可执行文件。

    -m   定位帮助文件。

    -s   定位源代码文件。

    -u   搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件。

    -B   指定搜索可执行文件的路径。

    -M   指定搜索帮助文件的路径。

    -S   指定搜索源代码文件的路径。

PS : 
linux系统内所有的文件都记录在一个数据库文件中，whereis命令是从这个数据库文件中进行检索，而不是像find命令扫描整个磁盘，所以会比find效率高。但数据库文件不是时时更新的，默认一星期更新一次，所以whereis找出来的命令可能被删除，或者新增命令检索不出来。









