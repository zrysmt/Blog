---
title: PHP爬虫最全总结2-phpQuery，PHPcrawer，snoopy框架中文介绍
tags: 
- 爬虫
- PHP
categories: PHP
---

第一篇文章介绍了使用原生的PHP和PHP的扩展库实现了爬虫技术。本文尝试使用PHP爬虫框架来写，首先对三种爬虫技术[phpQuery](https://github.com/punkave/phpQuery)，[PHPcrawer](http://phpcrawl.cuab.de/)， snoopy进行对比，然后分析模拟浏览器行为的方式，重点介绍下snoopy

> 所有代码挂在我的[github](https://github.com/zrysmt/PHPSpider)上

# 1.几种常用的PHP爬虫框架对比
## 1.1 [phpQuery](https://github.com/punkave/phpQuery)
**优势：**类似jquery的强大搜索DOM的能力。
pq()是一个功能强大的搜索DOM的方法，跟jQuery的$()如出一辙，jQuery的选择器基本上都能使用在phpQuery上，只要把“.”变成“->”,Demo如下(对应我的github的Demo5)

```php
<?php 
 require('phpQuery/phpQuery.php');
 phpQuery::newDocumentFile('http://www.baidu.com/'); 
 $menu_a = pq("a"); 
 foreach($menu_a as $a){
    echo pq($a)->html()."<br>";
 } 
 foreach($menu_a as $a){
    echo pq($a)->attr("href")."<br>";
 } 
?>
```
## 1.2 [PHPcrawer](http://phpcrawl.cuab.de/)
**优势：**过滤能力比较强。
官方给的Demo如下（我的github中对应demo4）：

```php
<?php 
    include("PHPCrawl/libs/PHPCrawler.class.php");
    class MyCrawler extends PHPCrawler 
    { 
      function handleDocumentInfo(PHPCrawlerDocumentInfo $PageInfo) 
      { // As example we just print out the URL of the document 
        echo $PageInfo->url."<br>"; 
      } 
    }
    $crawler = new MyCrawler(); 
    $crawler->setURL("www.baidu.com"); 
    $crawler->addURLFilterRule("#\.(jpg|gif)$# i");
    //过滤到含有这些图片格式的URL
    $crawler->go();
 ?>
```
## 1.3 snoopy
**优势：**提交表单，设置代理等
Snoopy是一个php类，用来模拟浏览器的功能，可以获取网页内容，发送表单，
demo如下（对应github中的demo3）：

```php
include 'Snoopy/Snoopy.class.php';
$snoopy = new Snoopy();
$url = "http://www.baidu.com";
// $snoopy->fetch($url);
// $snoopy->fetchtext($url);//去除HTML标签和其他的无关数据
$snoopy->fetchform($url);//只获取表单
//只返回网页中链接 默认情况下，相对链接将自动补全，转换成完整的URL。
// $snoopy->fetchlinks($url);
var_dump($snoopy->results);
```
## 1.4 [phpspider](http://www.sphider.eu/)
**优势：**安装配置到数据库
提供了安装配置，能够直接连接mysql数据库，使用也是比较广泛，这里我们暂时不单独介绍。
# 2.模拟用户行为
## 2.1 file_get_contents

```php
<?php
$opts = array(
  'http'=>array(
    'method'=>"GET",
    'header'=>"Accept-language: en\r\n" .
              "Cookie: foo=bar\r\n"
  )
);

$context = stream_context_create($opts);

/* Sends an http request to www.example.com
   with additional headers shown above */
$fp = fopen('http://www.example.com', 'r', false, $context);
fpassthru($fp);
fclose($fp);
?>
```
## 2.2 curl

```php
$ch=curl_init();  //初始化一个cURL会话
curl_setopt($ch,CURLOPT_URL,$url);//设置需要获取的 URL 地址
// 设置浏览器的特定header
curl_setopt($ch, CURLOPT_HTTPHEADER, array(
  "Host: www.baidu.com",
  "Connection: keep-alive",
  "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
  "Upgrade-Insecure-Requests: 1",
  "DNT:1",
  "Accept-Language: zh-CN,zh;q=0.8,en-GB;q=0.6,en;q=0.4,en-US;q=0.2",
  "Cookie:_za=4540d427-eee1-435a-a533-66ecd8676d7d;"    
));
$result=curl_exec($ch);//执行一个cURL会话
```
## 2.3 snoopy
- 表单提交

我们的一个例子
form-demo.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>form-demo</title>
</head>
<body>
    <form action="./form-demo.php" method="post">
        用户名:<input type="text" name="userName"><br>
        密 码:<input type="password" name="password"><br>
        <input type="submit">
    </form>
</body>
</html>
```
form-demo.php

```php
<?php 
    $userName = $_POST['userName'];
    $password = $_POST['password'];
    if($userName==="admin"&&$password==="admin"){
        echo "hello admin";
    }else{
        echo "login error";
    }
 ?>
 ```
提交表单
```php
<?php
include 'Snoopy/Snoopy.class.php';
$snoopy = new Snoopy();
$formvars["userName"] = "admin";
//userName 与服务器端/表单的name属性一致
$formvars["password"] = "admin";
$action = "http://localhost:8000/spider/demo3/form-demo.php";//表单提交地址
$snoopy->submit($action,$formvars);
echo $snoopy->results;
?>
```
> 问题1：openssl extension required for HTTPS 增加对https的支持

```
php.in ==> ;extension=php_openssl.dll 去除注释
```

> 问题2：405 Not Allowed增加

```php
$snoopy->agent = "(compatible; MSIE 4.01; MSN 2.5; AOL 4.0; Windows 98)"; //伪装浏览器
$snoopy->referer = "http://www.icultivator.com"; //伪装来源页地址 http_referer
$snoopy->rawheaders["Pragma"] = "no-cache"; //cache 的http头信息
$snoopy->rawheaders["X_FORWARDED_FOR"] = "122.0.74.166"; //伪装ip
```
> 问题3 : snoopy使用代理

```php
$snoopy->proxy_host = "http://www.icultivator.com";
// HTTPS connections over proxy are currently not supported
$snoopy->proxy_port = "8080"; //使用代理
$snoopy->maxredirs = 2; //重定向次数
$snoopy->expandlinks = true; //是否补全链接 在采集的时候经常用到
$snoopy->maxframes = 5; //允许的最大框架数
```

**问题：**
其实尝试了网站进行提交表单是有问题的。这样简单的处理是不能提交表单的，使用代理也是有问题
的。snoopy框架确实会有很多问题，后面有解决思路了再说。

**参考阅读：**
- [cURL、file_get_contents、snoopy.class.php 优缺点](https://my.oschina.net/junn/blog/147936)
- [开源中国-PHP爬虫框架列表](http://www.oschina.net/project/lang/22?tag=64&show=news)
- [phpQuery](http://blog.johnsonlu.org/phpphpquery/)
- [Snoopy下载地址](https://sourceforge.net/projects/snoopy/)
- [Snoopy —— 强大的PHP采集类使用详解及示例：采集、模拟登录及伪装浏览器](http://www.thinksaas.cn/topics/0/558/558466.html)
- [开源中国-snoopy博客列表](https://www.oschina.net/search?scope=blog&q=Snoopy)

