---
title: domReady机制探究及DOMContentLoaded研究
tags: 
- FE
- DOMContentLoaded
- domReady
categories: 前端技术
---
domReady机制是很多框架和库都具有的种子模块，使用了在DOM树解析完成后就立即响应，不用等待图片等资源下载完成（onload执行时候表示这些资源完全下载完成）的一种机制，那怎么实现呢。


1）支持DOMContentLoaded事件的，就使用DOMContentLoaded事件；
2）不支持的，就用来自Diego Perini发现的著名Hack兼容。兼容原理大概就是，通过IE中的document.documentElement.doScroll(‘left’)来判断DOM树是否创建完毕或者使用监控script标签的onreadystatechange得到它的readyState属性判断【遗憾的是经过我们的实验，在IE下domReady机制总会在onload后执行】
# 1. domReady机制在IE7-8下
## 1.1 domReady机制源码（包括IE/非IE）
demo1.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title> DOMContentLoaded Demo</title>
</head>
<body>
    <div id="div1"></div>
    <script defer='defer' src="deferjs.js"></script>
    <script src="http://cdn.bootcss.com/jquery/1.12.4/jquery.js"></script>
    <script type="text/javascript" src="demo1.js"></script>
    <script>
    dom.Ready(function() {
        console.info("我的domReady1");
    });
    </script>
</body>


</html>
```
deferjs.js
```javascript
console.log("defer script");
```
demo1.js
```javascript
dom = [];
dom.isReady = false;
dom.isFunction = function(obj) {
    return Object.prototype.toString.call(obj) === "[object Function]";
};
dom.Ready = function(fn) {
    dom.initReady(); //如果没有建成DOM树，则走第二步，存储起来一起杀
    if (dom.isFunction(fn)) {
        if (dom.isReady) {
            fn(); //如果已经建成DOM，则来一个杀一个
        } else {
            dom.push(fn); //存储加载事件
        }
    }
};
dom.fireReady = function() {
    if (dom.isReady) return;
    dom.isReady = true;
    for (var i = 0, n = dom.length; i < n; i++) {
        var fn = dom[i];
        fn();
    }
    dom.length = 0; //清空事件
};
dom.initReady = function() {
    if (document.addEventListener) {//非IE
        document.addEventListener("DOMContentLoaded", function() {
            console.log("DOMContentLoaded");
            document.removeEventListener("DOMContentLoaded", arguments.callee, false); //清除加载函数
            dom.fireReady();
        }, false);
    } else {//IE走这条线
        if (document.getElementById) {
            document.write("<script id=\"ie-domReady\" defer='defer'src=\"//:\"><\/script>");
            document.getElementById("ie-domReady").onreadystatechange = function() {
                console.log(this.readyState);
                if (this.readyState === "complete") {
                 //只针对IE readyState 的值 complete--脚本执行完成。
                 //这个时候DOM树肯定已经解析完成了，不支持defer属性
                 //会在onload函数之后执行。
                    dom.fireReady();
                    console.log('this.readyState === "complete"');
                    this.onreadystatechange = null;
                    this.parentNode.removeChild(this);
                }
            };
        }
    }
};
/**********测试**************************************************/
dom.Ready(function() {
    console.info("我的domReady2");
});
/*$(document).ready(function() {
    dom.Ready(function() {
        console.info("我的domReady4在jquery的ready函数中");
    });
    console.log('jquery中的ready函数');
});*/
dom.Ready(function() {
    console.info("我的domReady3");
});


console.log('在js中');
window.onload = function(){
  console.log("onload函数");
};
```
## 1.2 背景知识介绍
- `document.readystate`


> **readyState** 属性返回当前文档的状态（载入中……）。
该属性返回以下值:
- uninitialized - 还未开始载入
- loading - 载入中
- interactive - 已加载，文档与用户可以开始交互并引发[DOMContentLoaded](https://developer.mozilla.org/zh-CN/docs/Web/Reference/Events/DOMContentLoaded "/zh-CN/docs/Web/Reference/Events/DOMContentLoaded")事件
- complete - 载入完成




- IE的 script的readyState
FireFox的script 元素不支持onreadystatechange事件，只支持onload事件
IE的 script 元素支持onreadystatechange事件，不支持onload事件




>只针对IE readyState 的值  可能为 以下几个 :
-   “uninitialized” – 原始状态 
-   “loading” – 下载数据中..
-   “loaded” – 下载完成
-   “interactive” – 还未执行完毕.
-   “complete” – 脚本执行完毕




- **defer和onload函数**


```javascript
<script defer='defer' src="deferjs.js"></script>
```




> defer 属性仅适用于外部脚本（只有在使用 src 属性时）
> *   如果 async="async"：脚本相对于页面的其余部分异步地执行（当页面继续进行解析时，脚本将被执行）
> *   如果不使用 async 且 defer="defer"：脚本将在页面完成解析时执行
> *   如果既不使用 async 也不使用 defer：在浏览器继续解析页面之前，立即读取并执行脚本


## 1.3 结果分析
在IE7/8打印的结果是：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/domReady1.png)
dom.fireReady函数在onload函数之后执行


## 1.4 IE下监控DOM树是否解析完成的其他做法
除了使用`document.write("<script id=\"ie-domReady\" defer='defer'src=\"//:\"><\/script>")`还可以监控DOM树是否解析完成
在更早的IE版本中,可以通过每隔一段时间执行一次`document.documentElement.doScroll("left")来检测这一状态，`因为这条代码在DOM加载完毕之前执行时会抛出错误(throw an error)
```javascript
    (function() {
        try { //在DOM未建完之前调用元素的doScroll抛出错误
            document.documentElement.doScroll('left');
        } catch (e) { //延迟再试
            setTimeout(arguments.callee, 50);
            return;
        }
        init(); //没有错误则执行用户回调
    })();
```
# 2. domReady机制在chrome中
将上面的demo2.js文件下注释的jquery的ready函数取消注释进行执行，得到结果是：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/domReady2.png)
demo1.html中将script标签放入到`<head>`得到的结果是一样的。


**在chrome中的顺序是**：
+ `document.readyState` 为`loading`
  - jquery的ready函数外 (打印结果：在js中)
+ `document.readyState` 为`interactive`【DOM解析完成】
  - 带defer的script (打印结果：defer script)
  - jquery的ready函数里面 (打印结果：jquery中的ready函数)
  - 监听DOMContentLoaded要执行的函数 dom.fireReady(打印结果：DOMContentLoaded和我是domReady系列)
+ `document.readyState` 为`compelete`
 - onload函数（打印结果：onload函数）


# 3.总结
1，2区别：
- 带defer的script标签，IE8以下中不支持defer属性
- dom.fireReady在IE中的逻辑是在`document.readyState=="compelte"`后，会在onload函数之后紧接着执行,在chrome/Firfox的逻辑是在`document.addEventListener("DOMContentLoaded",function(){})`的回掉函数中。


综上所诉，执行的顺序应该为：
+ `document.readyState` 为`loading`
  - jquery的ready函数外
+ 【非IE下】`document.readyState` 为`interactive`【DOM解析完成】
  - 带defer的script
  - jquery的ready函数里面 
  - 触发`DOMContentLoaded`事件，监听DOMContentLoaded要执行的函数
+ 【IE常用来判断DOM树是否解析完成】document.documentElement.doScroll 这时可以让HTML元素使用doScroll方法，抛出错误就是DOM树未解析完成
+ `document.readyState` 为`compelete`
 - onload函数（打印结果：onload函数）【图片flash等资源都加载完毕】




最后附上监测IE，在IE的onload函数后面执行执行的另外一种实现方式
```javascript
//http://javascript.nwbox.com/IEContentLoaded/
//by Diego Perini 2007.10.5
function IEContentLoaded(w, fn) {
    var d = w.document||document,
        done = false,
        init = function() {
            if (!done) { //只执行一次
                done = true;
                fn();
            }
        };


    (function() {
        try { //在DOM未建完之前调用元素的doScroll抛出错误
            d.documentElement.doScroll('left');
        } catch (e) { //延迟再试
            setTimeout(arguments.callee, 50);
            return;
        }
        init(); //没有错误则执行用户回调
    })();
    // 如果用户是在domReady之后绑定这个函数呢？立即执行它
    d.onreadystatechange = function() {
        if (d.readyState == 'complete') {
            d.onreadystatechange = null;
            init();
        }
    };
}
```


参考阅读：
- 司徒正美 - 《javascript框架设计》
- [司徒正美-javascript的事件加载](http://www.cnblogs.com/rubylouvre/archive/2009/08/26/1554204.html)
- [主流JS框架中DOMReady事件的实现](http://www.cnblogs.com/JulyZhang/archive/2011/02/12/1952484.html)
- [javascript的domReady](http://www.cnblogs.com/rubylouvre/archive/2009/12/30/1635645.html)
- [document.readyState的属性](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/readyState)
- [DOMContentLoaded介绍](https://developer.mozilla.org/zh-CN/docs/Web/Events/DOMContentLoaded)
- [又说 动态加载 script. ie 下 script Element 的 readyState状态](http://www.cnblogs.com/_franky/archive/2010/06/20/1761370.html)


