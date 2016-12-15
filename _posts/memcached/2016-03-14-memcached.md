---
layout: post
title: "memcached之实际应用"
description: ""
category: cache
tags: []
---

> memcached 是个开源的，支持高性能，高并发的分布式内存缓存系统。

### memcached 作用

减少关系型数据库的访问压力，提高网站的整体访问速度。提升用户体验。

### 互联网常见的内存缓存服务软件

* 1.memcached ，纯内存型，常用于缓存网站后端各类数据。主要缓存用户请求的动态内容。
* 2.redis 可持久化储存，即使用内存，也使用磁盘。可以各种后端动态数据，也能作为数据库来使用。
* 3.squid，nginx ，内存或磁盘缓存，主要用于缓存web前端的服务内容，比如图片，附件，html静态代码等。实际生产中大多数会选择专业的CDN公司，如网宿，蓝汛。

### memcached的特点

* 1.协议简单。能直接通过telnet命令操作memcached服务存取数据。
* 2.支持epoll/kqueue 异步I/O模型，使用libevent作为实践处理通知机制。
* 3.采用key/value的键值数据类型。
* 4.内存缓存，存取速度快。
* 5.支持**分布式**集群。

tengine反向代理负载均衡的一致hash算法实现分布式memcached的配置。

```
http {
    upstream test {
        consistent_hash $request_uri;
        server 127.0.0.1:11211 id=1001 weight=3;
        server 127.0.0.1:11212 id=1002 weight=10;
        server 127.0.0.1:11213 id=1003 weight=20;
    }
}
```

### memcached内存分配机制

memcached采用了slab内存分配机制。过程如下：

* 1.提前将大内存分配大小为1MB的若干个slab，然后针对每个slab再进行小对象填充，这个小对象称为chunk，从而避免大量重复的初始化和清理任务，减小内存管理器的负担。
* 2.新增数据对象存储时，因memcached中保存着slab内空闲chunk列表，它会根据该列表选择chunk，然后将数据缓存于其中。
* 3.当有数据存入时，memcached根据接收到的数据大小，会选择最适合数据大小的slab。

### memcached采用了slab内存分配的缺点：

* 1.chunk存储item浪费空间。由于预先分配特定长度的内存，不可避免存储空间浪费，比如：将100字节的数据存储到128字节的chunk中，剩余的28字节就浪费了。

#### 优化方案：
第一，尽量把数据大小相同的数据存入同一台mamcached中。
第二，在启动mamcached的时候，指定-f参数，控制内存组之间的大小差异，默认值是1.25。

* 2.slab尾部剩余空间。由于slab class是由多个chunk块组成，最后的可能会有剩余的空间不够容纳一个chunk，导致这部分内存空间浪费。

解决办法就是：slab的大小等于chunk大小的整数倍。

### memcached才有LRU对象清除机制

memcached 不会主动去检测item对象是否过期，而是进行get操作时检查是否过期。如果内存被数据填充满了以后，采用最近最少使用算法释放内存空间。

但memcached中删除一个对象，一般不是立即释放内存空间，而是做删除标记，将指针放入slot回收插槽，下次分配直接使用。

---

### memcached服务安装

#### 先安装libevent以及nc

```
yum install libevent libevent-devel nc -y
rpm -qa libevent libevent-devel nc
```

#### 安装memcached

```
yum install memcached -y
rpm -qa memcached
```

### 启动memcached服务

```
which memcached
memcached -m 64m -p 11211 -d -u root -c 9000
memcached -m 64m -p 11212 -d -u root -c 9000  #可以同时开启多个memcached实例
#将启动命令加入/etc/rc.local文件中，即可开机启动。
```

### 向memcached中写入/取出数据

```
#set
printf "set key1 0 0 5 \r\ndnzhu\r\n" | nc 127.0.0.1 11211  #STORED  
#get
printf "get key1\r\n" | nc 127.0.0.1 11211  #dnzhu
#delete
printf "delete key1\r\n" | nc 127.0.0.1 11211  #DELETED
```

参数格式：command key flags exptime bytes\r\n data\r\n


### 关闭memcached服务

```
#关闭所有
killall memcached 或 pkill memcached
#关闭某个
kill `cat /var/run/11211.pid`
```

### 分布式缓存集群设计思想

* 1.每台memcached服务器的内容都不一样，所有的memcached服务器缓存内容大小接近于数据库的数据量。
* 2.通过客户端程序或者memcached的负载均衡器上的算法(一致hash算法)，让同一数据的内容都分配到一个memcached服务器上。
* 3.普通的hash算法对于服务器宕机会带来大量的缓存失效，可能会引起整个服务端雪崩。
* 4.一致hash算法可以让缓存服务器宕机对缓存数据失效降低到最低。推荐使用。

#### 一致hash算法

原理：首先想象一个0 ~ 2^32-1次方的数值空间，把这个空间看成收尾相接的圆环，然后计算不同的memcached服务器的哈希值，将这些值分散到这个圆环上，接着用相同的方法计算出存储不同数据的键的哈希值并映射到相同的圆环上，最后从数据映射到的位置I开始顺时针查找，将键对应的数据保存到查找到的第一个服务器上。这样就能最大限度的减少键的重新分布。

php程序实现一致哈希算法：

```
<?php
header('Content-type:text/html; charset=utf-8');
#抽象接口
interface hash {
    public function _hash($str);
}

interface distribution {
    public function lookup($key);
}
#一致hash算法实现
class Consistent implements hash,distribution{
    protected $point_num = 64;
    protected $posi = array();
    protected $server;

    #计算hash值
    public function _hash($str){
        return sprintf('%u',crc32($str));
    }

    #计算key分布到服务器
    public function lookup($key){
        foreach ($this->posi as $k => $v) {
            if($this->_hash($key) <= $k ){
                $this->server = $v;
                break;
            }
        }
        return $this->server;
    }

    #添加服务器节点
    public function addServer($server){
        for($i=1;$i<=$this->point_num;$i++) {
            $this->posi[$this->_hash($server.'_'.$i)] = $server;
        }
        $this->sortPosi();
    }
    #排序定位节点
    private function sortPosi(){
        ksort($this->posi);
    }

    #打印定位节点，用于测试
    public function printPosi(){
        echo '<pre>';
        print_r($this->posi);
    }
}

#调用
$hash = new Consistent();
$hash->addServer('a');
$hash->addServer('b');
$hash->addServer('c');
#test
$key = 'abc';
$server = $hash->lookup($key);
echo $key .'对应的服务器是：'.$server.'对应的hash值是：'.$hash->_hash($key);
echo '<hr />';
$hash->printPosi();
?>
```
---

### memcached在集群中的session共享

默认php.ini中session的类型和配置路径：

```
;session.save_handler = files
;session.save_path = "/tmp"

```

修改成：

```
session.save_handler = memcache
session.save_path = "tcp://10.20.0.240:11211"

```