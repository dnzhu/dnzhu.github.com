---
layout: post
title: "fast-cgi与php-fpm区别"
description: ""
category: php-basic
tags: []
---

> 关于fastcgi与php-fpm之间的关系，我在网上翻了很多的答案，基本看了个遍，真是众说纷纭，没一个权威性的定义。


### CGI是什么？

---
CGI是为了保证web server传递过来的数据是标准格式的,说的正式点，cgi 是一种协议。

web server（比如说nginx）只是内容的分发者。比如，如果请求/index.html，那么web server会去文件系统中找到这个文件，发送给浏览器，这里分发的是静态数据。好了，如果现在请求的是/index.php，根据配置文件，nginx知道这个不是静态文件，需要去找PHP解析器来处理，那么他会把这个请求简单处理后交给PHP解析器。Nginx会传哪些数据给PHP解析器呢？url要有吧，查询字符串也得有吧，POST数据也要有，HTTP header不能少吧，好的，CGI就是规定要传哪些数据、以什么样的格式传递给后方处理这个请求的协议。

当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以规定CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器。

---

### FastCGI又是什么？

---
Fastcgi是用来提高CGI程序性能的。那么CGI程序的性能问题在哪呢？

"PHP解析器会解析php.ini文件，初始化执行环境"，就是这里了。标准的CGI对每个请求都会执行这些步骤（不闲累啊！启动进程很累的说！），所以处理每个时间的时间会比较长。这明显不合理嘛！

那么Fastcgi是怎么做的呢？

首先，Fastcgi会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。这就是fastcgi的对进程的管理。

---


### PHP-FPM又是什么？
---
PHP-FPM是一个实现了Fastcgi的程序，后来被php官方收录了。　

大家都知道，PHP的解释器是php-cgi。php-cgi只是个CGI程序，他自己本身只能解析请求，返回结果，不会进程管理（皇上，臣妾真的做不到啊！）所以就出现了一些能够调度php-cgi进程的程序，比如说由lighthttpd分离出来的spawn-fcgi。好了PHP-FPM也是这么个东东，在长时间的发展后，逐渐得到了大家的认可（要知道，前几年大家可是抱怨PHP-FPM稳定性太差的），也越来越流行。

---
ps : 这里只是从概念上，把这几个名词加以说明，详细内容，推荐大家阅读《HTTP权威指南》。
{% include JB/setup %}


