---
title: PHP爬虫最全总结1
tags: 
- 爬虫
- PHP
categories: PHP
---

　爬虫是我一直以来跃跃欲试的技术，现在的爬虫框架很多，比较流行的是基于python，nodejs，java，C#，PHP的的框架，其中又以基于python的爬虫流行最为广泛，还有的已经是一套傻瓜式的软件操作，如八爪鱼，火车头等软件。
  
　今天我们首先尝试的是使用PHP实现一个爬虫程序，首先在不使用爬虫框架的基础上实践也是为了理解爬虫的原理，然后再利用PHP的lib，框架和扩展进行实践。

> 所有代码挂在我的[github](https://github.com/zrysmt/PHPSpider)上。

# 1.PHP简单的爬虫--原型

**爬虫的原理：**

- 给定原始的url；
- 分析链接，根据设置的正则表达式获取链接中的内容；
- 有的会更新原始的url再进行分析链接，获取特定内容，周而复始。
- 将获取的内容保存在数据库中（mysql）或者本地文件中

下面是网上一个例子，我们列下来然后分析
从`main`函数开始

```php
<?php
/**
 * 爬虫程序 -- 原型
 *
 * 从给定的url获取html内容
 * 
 * @param string $url 
 * @return string 
 */
function _getUrlContent($url) {
    $handle = fopen($url, "r");
    if ($handle) {
        $content = stream_get_contents($handle, -1);
        //读取资源流到一个字符串,第二个参数需要读取的最大的字节数。默认是-1（读取全部的缓冲数据）
        // $content = file_get_contents($url, 1024 * 1024);
        return $content;
    } else {
        return false;
    } 
} 
/**
 * 从html内容中筛选链接
 * 
 * @param string $web_content 
 * @return array 
 */
function _filterUrl($web_content) {
    $reg_tag_a = '/<[a|A].*?href=[\'\"]{0,1}([^>\'\"\ ]*).*?>/';
    $result = preg_match_all($reg_tag_a, $web_content, $match_result);
    if ($result) {
        return $match_result[1];
    } 
} 
/**
 * 修正相对路径
 * 
 * @param string $base_url 
 * @param array $url_list 
 * @return array 
 */
function _reviseUrl($base_url, $url_list) {
    $url_info = parse_url($base_url);//解析url
    $base_url = $url_info["scheme"] . '://';
    if ($url_info["user"] && $url_info["pass"]) {
        $base_url .= $url_info["user"] . ":" . $url_info["pass"] . "@";
    } 
    $base_url .= $url_info["host"];
    if ($url_info["port"]) {
        $base_url .= ":" . $url_info["port"];
    } 
    $base_url .= $url_info["path"];
    print_r($base_url);
    if (is_array($url_list)) {
        foreach ($url_list as $url_item) {
            if (preg_match('/^http/', $url_item)) {
                // 已经是完整的url
                $result[] = $url_item;
            } else {
                // 不完整的url
                $real_url = $base_url . '/' . $url_item;
                $result[] = $real_url;
            } 
        } 
        return $result;
    } else {
        return;
    } 
} 
/**
 * 爬虫
 * 
 * @param string $url 
 * @return array 
 */
function crawler($url) {
    $content = _getUrlContent($url);
    if ($content) {
        $url_list = _reviseUrl($url, _filterUrl($content));
        if ($url_list) {
            return $url_list;
        } else {
            return ;
        } 
    } else {
        return ;
    } 
} 
/**
 * 测试用主程序
 */
function main() {
    $file_path = "url-01.txt";
    $current_url = "http://www.baidu.com/"; //初始url
    if(file_exists($file_path)){
        unlink($file_path);
    }
    $fp_puts = fopen($file_path, "ab"); //记录url列表
    $fp_gets = fopen($file_path, "r"); //保存url列表
    do {
        $result_url_arr = crawler($current_url);
        if ($result_url_arr) {
            foreach ($result_url_arr as $url) {
                fputs($fp_puts, $url . "\r\n");
            } 
        } 
    } while ($current_url = fgets($fp_gets, 1024)); //不断获得url
} 
main();
?>
```

# 2.使用crul lib
Curl是比较成熟的一个lib，异常处理、http header、POST之类都做得很好，重要的是PHP下操作MySQL进行入库操作比较省心。关于curl的说明具体可以查看PHP官方文档说明[http://php.net/manual/zh/book.curl.php](http://php.net/manual/zh/book.curl.php)
不过在多线程Curl（Curl_multi）方面比较麻烦。

**开启crul**
针对winow系统：
- php.in中修改（注释；去掉即可） 
> extension=php_curl.dll

- php文件夹下的libeay32.dll, ssleay32.dll, libssh2.dll 还有 php/ext下的php_curl4个文件移入windows/system32

使用crul爬虫的**步骤：**
- 使用cURL函数的基本思想是先使用curl_init()初始化一个cURL会话；
- 接着你可以通过curl_setopt()设置你需要的全部选项；
- 然后使用curl_exec()来执行会话；
- 当执行完会话后使用curl_close()关闭会话。

**例子**

```php
<?php
$ch = curl_init("http://www.example.com/");
$fp = fopen("example_homepage.txt", "w");

curl_setopt($ch, CURLOPT_FILE, $fp);
curl_setopt($ch, CURLOPT_HEADER, 0);

curl_exec($ch);
curl_close($ch);
fclose($fp);
?>
```
一个完整点的例子：

```php
<?php
/**
 * 将demo1-01换成curl爬虫
 * 爬虫程序 -- 原型
 * 从给定的url获取html内容
 * @param string $url 
 * @return string 
 */
function _getUrlContent($url) {
    $ch=curl_init();  //初始化一个cURL会话
    /*curl_setopt 设置一个cURL传输选项*/
    //设置需要获取的 URL 地址
    curl_setopt($ch,CURLOPT_URL,$url);
    //TRUE 将curl_exec()获取的信息以字符串返回，而不是直接输出
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
    //启用时会将头文件的信息作为数据流输出
    curl_setopt($ch,CURLOPT_HEADER,1);
    // 设置浏览器的特定header
    curl_setopt($ch, CURLOPT_HTTPHEADER, array(
        "Host: www.baidu.com",
        "Connection: keep-alive",
        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
        "Upgrade-Insecure-Requests: 1",
        "DNT:1",
        "Accept-Language: zh-CN,zh;q=0.8,en-GB;q=0.6,en;q=0.4,en-US;q=0.2",
        /*'Cookie:_za=4540d427-eee1-435a-a533-66ecd8676d7d; */    
        ));
    $result=curl_exec($ch);//执行一个cURL会话
    $code=curl_getinfo($ch,CURLINFO_HTTP_CODE);// 最后一个收到的HTTP代码
    if($code!='404' && $result){
       return $result;
    }
    curl_close($ch);//关闭cURL
} 
/**
 * 从html内容中筛选链接
 * @param string $web_content 
 * @return array 
 */
function _filterUrl($web_content) {
    $reg_tag_a = '/<[a|A].*?href=[\'\"]{0,1}([^>\'\"\ ]*).*?>/';
    $result = preg_match_all($reg_tag_a, $web_content, $match_result);
    if ($result) {
        return $match_result[1];
    } 
} 
/**
 * 修正相对路径
 * @param string $base_url 
 * @param array $url_list 
 * @return array 
 */
function _reviseUrl($base_url, $url_list) {
    $url_info = parse_url($base_url);//解析url
    $base_url = $url_info["scheme"] . '://';
    if ($url_info["user"] && $url_info["pass"]) {
        $base_url .= $url_info["user"] . ":" . $url_info["pass"] . "@";
    } 
    $base_url .= $url_info["host"];
    if ($url_info["port"]) {
        $base_url .= ":" . $url_info["port"];
    } 
    $base_url .= $url_info["path"];
    print_r($base_url);
    if (is_array($url_list)) {
        foreach ($url_list as $url_item) {
            if (preg_match('/^http/', $url_item)) {
                // 已经是完整的url
                $result[] = $url_item;
            } else {
                // 不完整的url
                $real_url = $base_url . '/' . $url_item;
                $result[] = $real_url;
            } 
        } 
        return $result;
    } else {
        return;
    } 
} 
/**
 * 爬虫
 * @param string $url 
 * @return array 
 */
function crawler($url) {
    $content = _getUrlContent($url);
    if ($content) {
        $url_list = _reviseUrl($url, _filterUrl($content));
        if ($url_list) {
            return $url_list;
        } else {
            return ;
        } 
    } else {
        return ;
    } 
} 
/**
 * 测试用主程序
 */
function main() {
    $file_path = "./url-03.txt";
    if(file_exists($file_path)){
        unlink($file_path);
    }
    $current_url = "http://www.baidu.com"; //初始url
    //记录url列表 　ab- 追加打开一个二进制文件，并在文件末尾写数据
    $fp_puts = fopen($file_path, "ab"); 
    //保存url列表 r-只读方式打开，将文件指针指向文件头
    $fp_gets = fopen($file_path, "r"); 
    do {
        $result_url_arr = crawler($current_url);
        echo "<p>$current_url</p>";
        if ($result_url_arr) {
            foreach ($result_url_arr as $url) {
                fputs($fp_puts, $url . "\r\n");
            } 
        } 
    } while ($current_url = fgets($fp_gets, 1024)); //不断获得url
} 
main();
?>
```

要对https支持，需要在`_getUrlContent`函数中加入下面的设置：
```php
curl_setopt($ch, CURLOPT_HTTPAUTH, CURLAUTH_BASIC ) ; 
curl_setopt($ch, CURLOPT_USERPWD, "username:password");    
curl_setopt($ch, CURLOPT_SSLVERSION,3); 
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE); 
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2);
```

**结果疑惑：**
我们通过1和2部分得到的结果差异很大，第1部分能得到四千多条url数据，而第2部分却一直是45条数据。

还有我们获得url数据可能会有重复的，这部分处理在我的[github](https://github.com/zrysmt/PHPSpider)上，对应demo2-01.php，或者demo2-02.php
# 3.file_get_contents/stream_get_contents与curl对比

## 3.1 file_get_contents/stream_get_contents对比
- stream_get_contents — 读取资源流到一个字符串
  与 [file_get_contents()]一样，但是 **stream_get_contents()** 是对一个已经打开的资源流进行操作，并将其内容写入一个字符串返回
  
```php
$handle = fopen($url, "r");
$content = stream_get_contents($handle, -1);//读取资源流到一个字符串,第二个参数需要读取的最大的字节数。默认是-1（读取全部的缓冲数据）
```
- file_get_contents — 将整个文件读入一个字符串

```php
$content = file_get_contents($url, 1024 * 1024);
```
> 【注】 如果要打开有特殊字符的 URL （比如说有空格），就需要使用进行 URL 编码。

## 3.2 file_get_contents/stream_get_contents与curl对比
[php中file_get_contents与curl性能比较分析](http://www.jb51.net/article/57238.htm)一文中有详细的对比分析，主要的对比现在列下来：
- fopen /file_get_contents 每次请求都会重新做DNS查询，并不对 DNS信息进行缓存。但是CURL会自动对DNS信息进行缓存。对同一域名下的网页或者图片的请求只需要一次DNS查询。这大大减少了DNS查询的次数。所以CURL的性能比fopen /file_get_contents 好很多。

- fopen /file_get_contents 在请求HTTP时，使用的是http_fopen_wrapper，不会keeplive。而curl却可以。这样在多次请求多个链接时，curl效率会好一些。

- fopen / file_get_contents 函数会受到php.ini文件中allow_url_open选项配置的影响。如果该配置关闭了，则该函数也就失效了。而curl不受该配置的影响。

- curl 可以模拟多种请求，例如：POST数据，表单提交等，用户可以按照自己的需求来定制请求。而fopen / file_get_contents只能使用get方式获取数据。

# 4.使用框架
使用框架这一块打算以后单独研究，并拿出来单写一篇博客


> 所有代码挂在我的[github](https://github.com/zrysmt/PHPSpider)上。


**参考阅读：**
- [我用爬虫一天时间“偷了”知乎一百万用户，只为证明PHP是世界上最好的语言](http://www.epooll.com/archives/806/)
- [知乎 -- PHP, Python, Node.js 哪个比较适合写爬虫？](https://www.zhihu.com/question/23643061)
- [最近关于对网络爬虫技术总结](http://qoofan.com/read/Pndwa54e8J.html)
- [PHP实现简单爬虫   (http://www.oschina.net/code/snippet_258733_12343)](https://www.oschina.net/code/snippet_258733_12343)]
- [一个PHP实现的轻量级简单爬虫](http://www.jb51.net/article/69108.htm)
- [php中file_get_contents与curl性能比较分析](http://www.jb51.net/article/57238.htm)
- [PHP curl之爬虫初步](http://www.cnblogs.com/freephp/p/4861184.html)
- [开源中国-PHP爬虫框架列表](http://www.oschina.net/project/lang/22?tag=64&show=news)
- [网页抓取：PHP实现网页爬虫方式小结，抓取爬虫](http://www.ido321.com/1158.html)
- [PHP爬虫框架--PHPCrawl](http://phpcrawl.cuab.de/)
- [php安装pcntl扩展实现多进程](http://www.chhua.com/web-note5277 "链接到 php安装pcntl扩展实现多进程")