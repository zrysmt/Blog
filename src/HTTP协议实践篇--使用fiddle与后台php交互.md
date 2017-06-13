---
title: HTTP协议实践篇--使用Fiddler与后台php交互
tags: 
- HTTP协议
- HTTP协议实践
- Fiddler
categories: HTTP/TCP/IP
---

工具：
- PHP、Apache服务器，端口号这里设置为8000,如果本机没有安装php环境，可以选择wamp或者xampp集成的php环境开发器
- [fiddler](http://www.telerik.com/fiddler)是比较好用的抓包工具，它是免费的，具体使用我们不单独介绍。

# 1.模拟form表单提交数据

**html表单写法**

```html
<form action="http://127.0.0.1:8000/test.php" method="post" >
    <input type="text" name="name">
    <input type="submit" value="提交">
</form>
```
method改为`post`,`get`去体会它们的区别
- 最大的区别是`get`方式，提交的话会将提交的数据放在url中，如`http://127.0.0.1:8000/test.php?name=hello`
`post`请求提交的name=hello会放在header请求体内，具体位置在【**报文主体**】
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/1/http1.jpg)
- get传送的数据量较小，不能大于2KB。post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB
- 其实，两种方式都可以向服务器传送数据，向服务器上获取数据。
**test.php**

```php
<?php 
    $name1 = $_POST['name'];
    $name2 = $_GET['name'];
    echo "$name1";
    echo "$name2";
 ?>
```
完整的请求头是

```
POST http://127.0.0.1:8000/test.php HTTP/1.1
Host: 127.0.0.1:8000
Connection: keep-alive
Content-Length: 10
Cache-Control: max-age=0
Origin: null
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

name=hello
```
最后的结果，分别两种方式提交，总会有一个会显示没有定义,一个显示出请求的数据

下面就用Fiddler模拟
- 使用`post`方式
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/1/http2.jpg)
`Content-Type`不能忽略
```
Content-Type: application/x-www-form-urlencoded
Content-Length: 10
Host: 127.0.0.1:8000
```
点击右上角【Execute】，就能模拟一个form表单提交数据了。
右边会显示一条我们刚刚的HTTP请求。
- 使用`get`方式
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/1/http3.jpg)
【Execute】执行

# 2.模拟文件操作
## 2.1 上传文件
post方式提交请求，很少会用get方式去请求文件
**html**

```html
<form action="http://127.0.0.1:8000/test/upload_file.php" method="post" enctype="multipart/form-data">
    <input type="text" name="name"><br>
    <input type="file" name="file" id="file"><br>
    <input type="submit" value="提交">
</form>
```
**php**
```php
<?php 
    $name = $_POST['name'];

    echo "$name";
    echo "<br>";
    echo $_FILES["file"]["name"];//$_FILES["file"]通过 HTTP POST 方式上传到当前脚本的项目的数组。
    echo "<br>";

    if (file_exists("upload/" . $_FILES["file"]["name"])) {
      echo $_FILES["file"]["name"] . " already exists. ";
    }else{
      move_uploaded_file($_FILES["file"]["tmp_name"],
      "upload/" . $_FILES["file"]["name"]);
      echo "Stored in: " . "upload/" . $_FILES["file"]["name"];
    }
 ?>
```
**请求头文件**
- 在chrome中

```
POST http://127.0.0.1:8000/test/upload_file.php HTTP/1.1
Host: 127.0.0.1:8000
Connection: keep-alive
Content-Length: 291
Cache-Control: max-age=0
Origin: null
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/53.0.2785.116 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryC1Pk1uMWXzAMqRMF
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8

------WebKitFormBoundaryC1Pk1uMWXzAMqRMF
Content-Disposition: form-data; name="name"

wenjian
------WebKitFormBoundaryC1Pk1uMWXzAMqRMF
Content-Disposition: form-data; name="file"; filename="1.txt"
Content-Type: text/plain

This is a txt.
------WebKitFormBoundaryC1Pk1uMWXzAMqRMF--

```
- 在Firfox中

```
POST http://127.0.0.1:8000/test/upload_file.php HTTP/1.1
Host: 127.0.0.1:8000
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:47.0) Gecko/20100101 Firefox/47.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: keep-alive
Content-Type: multipart/form-data; boundary=---------------------------31340552315478
Content-Length: 302

-----------------------------31340552315478
Content-Disposition: form-data; name="name"

txt文件
-----------------------------31340552315478
Content-Disposition: form-data; name="file"; filename="1.txt"
Content-Type: text/plain

This is a txt.
-----------------------------31340552315478--

```
其中最大的差异也就是`boundary`分界线
分界线里的是上传文件的信息，如果是个图片，我们会看到
```
Content-Disposition: form-data; name="file"; filename="CCGIS2.png"
Content-Type: image/png

//下面这些是图片的信息

```
**响应头信息**差别并不大
```
HTTP/1.1 200 OK
Date: Tue, 20 Sep 2016 06:48:58 GMT
Server: Apache/2.4.16 (Win64) PHP/5.6.13
X-Powered-By: PHP/5.6.13
Content-Length: 43
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8

wenjian<br>1.txt<br>Stored in: upload/1.txt
```
**用Fiddler模拟**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/HTTP%E5%8D%8F%E8%AE%AE/1/http4.jpg)
这里的头信息完全是照抄在chrome请求的头信息，其中最重要的是`Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryC1Pk1uMWXzAMqRMF`,我们用这一行就可以去执行，当然Fiddler会自动加入
```
Host: 127.0.0.1:8000
Content-Length: 295
```
有意思的是我们在`Request Body`中可以修改`filename`就可以修改上传后的文件名，我们也可以添加些内容，上传到服务器端。
```
------WebKitFormBoundaryC1Pk1uMWXzAMqRMF
Content-Disposition: form-data; name="name"

wenjian
------WebKitFormBoundaryC1Pk1uMWXzAMqRMF
Content-Disposition: form-data; name="file"; filename="3.txt"
Content-Type: text/plain

This is a txt. 这里是加入的内容 hahahha 哈哈哈
------WebKitFormBoundaryC1Pk1uMWXzAMqRMF--
```
我们甚至可以修改boundary:`Content-Type: multipart/form-data; boundary=----123456`
那么`Request Body`中
```
------123456
Content-Disposition: form-data; name="name"

wenjian
------123456
Content-Disposition: form-data; name="file"; filename="4.txt"
Content-Type: text/plain

This is a txt.hahahha 哈哈哈
------123456
```
值得一提的是，`Ruqest Body`右边的`Upload File`可以将选择的文件放在请求体中。注意修改`name`属性，与php文件的获取字段相同。
## 2.2 请求文件
请求文件其实很简单
POST/GET http://127.0.0.1:8000/test/upload/1.txt
【Excute】执行即可

既然我们是来实践HTTP协议的，那么很重要的文件缓存这方面的我们也可以实践，这些内容我打算单独写一篇博客

-----------------------华丽的分界线----------------------------
感觉这些技术很基础，但是也很黑客，我们完全可以使用这些技术去“黑”些网站，哈哈，当然要遵纪守法，当然有这些技术还是不够的。






