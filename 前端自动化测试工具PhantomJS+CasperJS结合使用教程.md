---
title: 前端自动化测试工具PhantomJS+CasperJS结合使用教程
tags: 
- FE
- PhantomJS
- CasperJS
- 自动化测试
categories: 前端技术
---
> 下面的安装测试基于window系统（win10）

# 1.PhantomJS
[**PhantomJS** ](http://phantomjs.org/ " phantomjs ")是一个基于 **WebKit** 的服务器端JavaScript API,它全面支持web而不需浏览器支持，其快速，原生支持各种Web标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG。 PhantomJS 可以用于 页面自动化， 网络监测 ， 网页截屏 ，以及 无界面测试 等
## 1.1 安装
下载地址为：http://phantomjs.org/download.html 解压之后，可以加到环境变量中
## 1.2 使用
- 示例demo1.js--截图：

```javascript
var page = require('webpage').create();
page.viewportSize = {
    width: 1366,
    height: 800
};
var urls = ["https://www.baidu.com/", "https://zrysmt.github.io/"];
page.open(urls[0], function() {
    console.log('welcome!');
    page.render('screen.png');
    phantom.exit();
});
```
命令行输入：
```
phantomjs demo1.js
```

- 示例demo2.js--DOM操作

```javascript
var page = require('webpage').create();
phantom.outputEncoding = "gbk"; //解决中文乱码
page.open("https://www.baidu.com/", function(status) {
    console.log(status);
    page.render('screen.png');
    var title = page.evaluate(function() {
        return document.title;
    });
    console.log('Page title: ' + title);
    phantom.exit();
});
```
执行命令同上，得到结果是：
```
success
Page title: 百度一下，你就知道
```
所执行的DOM操作要在`page.evaluate`中，

- 示例demo3.js--读取文件

```javascript
var system = require('system');
var fs = require('fs');

phantom.outputEncoding = "gbk"; //解决中文乱码
var filePath = "url-02.txt";
var content = fs.read(filePath);
console.log(content);
```

url-02.txt中如果是很多url，一个一个访问url的话，可能会这样实现
```javascript
page.open(urlArr[0], function(status) {
    page.open(urlArr[1], function(status) {
       page.open(urlArr[2], function(status) {
            //... ...
        }); 
    });
});
```
这样写法就有很大的不方便，于是我们就引入了CasperJS
# 2.CasperJS
## 2.1 安装
```
npm i casperjs --save-dev
```
为方便使用也可以加入到环境变量中
## 2.2 使用
- 示例demo:casper-test.js--打开网页截图

```javascript
var casper = require('casper').create();
casper.start();
casper.thenOpen('http://www.baidu.com/', function () {
    casper.captureSelector('baidu.png', 'html');
});
casper.run();
```
执行命令如下
```
casperjs casper-test.js
```

- 示例demo:casper-test2.js--操作DOM，访问网页

```javascript
var casper = require('casper').create();
var links;

function getLinks() {
// Scrape the links from top-right nav of the website
    var links = document.querySelectorAll('ul.navigation li a');
    return Array.prototype.map.call(links, function (e) {
        return e.getAttribute('href')
    });
}
// Opens casperjs homepage
casper.start('http://casperjs.org/');

casper.then(function () {
    links = this.evaluate(getLinks);
});

casper.run(function () {
    for(var i in links) {
        console.log(links[i]);
    }
    casper.done();
});
```
执行命令类比同上

- 示例demo:casper-test3.js--单元测试

```javascript
function Cow() {
    this.mowed = false;
    this.moo = function moo() {
        this.mowed = true; // mootable state: don't do that at home
        return 'moo!';
    };
}

casper.test.begin('Cow can moo', 2, function suite(test) {
    var cow = new Cow();
    test.assertEquals(cow.moo(), 'moo!');
    test.assert(cow.mowed);
    test.done();
});
```

执行命令
```
casperjs test casper-test3.js
```
结果是：
```
Test file: casper-test3.js
# Cow can moo
PASS Subject equals the expected value
PASS Subject is strictly true
PASS Cow can moo (2 tests)
PASS 2 tests executed in 0.031s, 2 passed, 0 failed, 0 dubious, 0 skipped.
```
- 示例demo:casper-test4.js--浏览器测试

```javascript
casper.test.begin('Google search retrieves 10 or more results', 5, function suite(test) {
    casper.start("http://www.google.fr/", function() {
        test.assertTitle("Google", "google homepage title is the one expected");
        test.assertExists('form[action="/search"]', "main form is found");
        this.fill('form[action="/search"]', {
            q: "casperjs"
        }, true);
    });

    casper.then(function() {
        test.assertTitle("casperjs - Recherche Google", "google title is ok");
        test.assertUrlMatch(/q=casperjs/, "search term has been submitted");
        test.assertEval(function() {
            return __utils__.findAll("h3.r").length >= 10;
        }, "google search for \"casperjs\" retrieves 10 or more results");
    });

    casper.run(function() {
        test.done();
    });
});
```
执行命令如demo3类比

# 3.PhantomJs+CasperJs
实现异步操作

```javascript
var casper = require('casper').create(); //新建一个页面

casper.start(url1); //添加第一个URL
casper.thenOpen(url2); //添加第二个URL,依次类推
casper.thenOpen(url3);
casper.thenOpen(url4);

casper.run(); //开始导航
```
demo(casper-phantomjs.js)如下--一次访问三十几个url：
```javascript
var fs = require('fs');
var casper = require('casper').create();
phantom.outputEncoding = "gbk"; //解决中文乱码

var filePath = "url-02.txt";
var content = fs.read(filePath);
var urlArr = content.split('\n');
casper.start();
for (var i = 0; i < urlArr.length; i++) {
    casper.thenOpen(urlArr[i], function() {
        this.echo('Page title: ' + this.getTitle());
    });
}
casper.run();
// phantom.exit();
```
执行命令
```
casperjs casper-phantomjs.js
```

**参考阅读：**
- [http://phantomjs.org/](http://phantomjs.org/)
- [PhantomJS快速入门教程](http://www.tuicool.com/articles/beeMNj/)
- [http://casperjs.org/](http://casperjs.org/)
- [浏览器自动化测试初探 - 使用phantomjs与casperjs](http://imweb.io/topic/55e46d8d771670e207a16bdc)
- [CasperJS,基于PhantomJS的工具包](http://www.cnblogs.com/ziyunfei/archive/2012/09/27/2706254.html)

