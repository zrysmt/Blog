---
title: HTTP协议实践篇--浏览器缓存总结、利用Fiddler和apache模拟
tags: 
- HTTP协议
- HTTP协议实践
- 浏览器缓存
date: 2016-09-27 00:00:00
categories: HTTP/TCP/IP
---
# 1.浏览器缓存
废话少说，我们先了解浏览器缓存的知识。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-1.png)
其中优先级是：Cache-Control>Expires>协商缓存
浏览器访问缓存的顺序是：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-2.png)
# 2.浏览器刷新的几种状态
- **普通模式** 我们下面的叙述在没有特殊说明的情况下就是这个模式
- **普通页面跳转**（点击页面链接跳转，window.open，在地址栏敲回车，刷新页面）
  + 无缓存情况下，请求会返回所有资源结果
  + 设置Expires并且未过期时，浏览器将不会发出http请求
  + 如果Expires过期，则会发送相应请求，并附带上Last-Modifed等信息，供服务器校验
- **页面刷新(F5)**
   这种情况一下，一般会看到很多304的请求，就是说即便资源设置了Expires且未过期，浏览器也会发送相应请求，**命中协商缓存**。
- **强制刷新(Ctrl+F5)**
   效果和无缓存时候一致，返回200的结果

# 3.强缓存
返回的http状态为**200**，在chrome的开发者工具的network里面size会显示为from cache
## 3.1 Cache-Control
请求指令
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-3.png)
响应指令
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-4.png)
这里需要注意`no-cache`对客户端和服务器含义是不同的，见下图
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-5.png)

## 3.2 Expires
资源失效的日期
```
Expires:  Wed, 21 Sep 2016 12:06:44 GMT
```

---
**实践：**
## 实践3-1 html文件中
在html文件<head>中

```html
<meta http-equiv="cache-control" content="no-cache">
<meta http-equiv="expires" content="Wed, 25 Oct 2016 13:19:55 GMT">
```
这些方法不常用，而且测试不能通过
## 实践3-2 PHP中设置

可以使用php设置，当然也可以在apche服务器中进行设置
```php
header("Cache-Control: public");
header("Pragma: cache");
$offset = 30*60*60*24; // cache 1 month
$ExpStr = "Expires: ".gmdate("D, d M Y H:i:s", time() + $offset)." GMT";
header($ExpStr);
```
chrome network工具栏显示
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-6.png)
**打开浏览器新窗口的方式测试，而不是F5刷新**
chrome network工具栏显示
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-7.png)

## 实践3-3 不需要PHP
可以像[浅谈浏览器http的缓存机制](http://www.cnblogs.com/vajoy/p/5341664.html)文中所使用的方法

- 在fildder右下角黑色区域--命令行，输入如：bpu localhost:8000 阻断来自localhost:8000的本地http请求      
- 点击被拦截的请求，可以在右栏直接修改报文内容（上半区域是请求报文，下半区域是响应报文），点击黄色的“Break on Response”按钮可以执行下一步（把请求发给服务器），点击绿色的按钮“Run to Completion”可以直接完成整个请求过程

---

# 4.协商缓存
当浏览器对某个资源的请求没有命中强缓存，就会发一个请求到服务器，验证协商缓存是否命中，如果协商缓存命中，请求响应返回的http状态为**304**并且会显示一个Not Modified的字符串
## 4.1 Last-Modified，If-Modified-Since
这对名词通常是成对出现的
`last-modified`：服务端设置的文档的最后的更新日期
`if-modified-since`用于指定这个时间以后的服务器资源，GMT格式

## 4.2 ETag、If-None-Match/if-match
这对名词通常也是成对出现的
`ETag`用于服务器向客户端传送的代表实体内容特征的标记信息
`If-None-Match/if-match`服务器给客户机传送网页的时候，可以传递代表实体内容特征的头字段（ETag），这种头字段被叫做实体标签。当客户机再次向服务端发请求的时候，会使用if-match携带实体标签信息

---

**实践：**
**php**
```php
header("Last-Modified:".gmdate("D, d M Y H:i:s") . " GMT" );
```
**使用fiddler请求**
```
如：
If-Modified-Since: Wed, 04 Oct 2016 13:32:30 GMT
Last-Modified: Wed, 05 Oct 2016 13:32:30 GMT
```
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/2/http2-8.png)
测试了几遍，并没有返回 304，原因不明

---

**参考阅读:**
- [php header()函数设置页面Cache缓存](http://www.111cn.net/phper/php/48528.htm)
- [在php编程中使用header()函数发送文件头，设置浏览器缓存，加快站点的访问速度](http://www.lampweb.org/seo/4/11.html)
- [浅谈浏览器http的缓存机制](http://www.cnblogs.com/vajoy/p/5341664.html)
- [HTML meta标签总结与属性的使用介绍](http://www.imooc.com/article/4475)