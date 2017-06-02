---
title: 阿里巴巴校招2017前端笔试题目 -- 原生js/html5 实现一个路由
date: 2017-04-28 12:44:52
tags:
- FE
- 路由
- 笔试
categories: 前端技术
---

阿里巴巴校招2017前端笔试题目：
1）路由有什么缺点？
2）原生js/html5 实现一个路由

缺点:
*   使用浏览器的前进，后退键的时候会重新发送请求，没有合理地利用缓存
*   单页面无法记住之前滚动的位置，无法在前进，后退的时候记住滚动的位置

路由的概念：
*   路由是根据不同的 url 地址展示不同的内容或页面
*   前端路由就是把不同路由对应不同的内容或页面的任务交给前端来做，之前是通过服务端根据 url 的不同返回不同的页面实现的。

我们直接来看两个例子，一个是hash结构的，这是在Html5 的history api出现之前的解决方案；一个是基于history api实现的。
- hash

```
http://10.0.0.1/
http://10.0.0.1/#/about
http://10.0.0.1/#/concat
```
- history

```
http://10.0.0.1/
http://10.0.0.1/about
http://10.0.0.1/concat
```
前端的路由和后端的路由在实现技术上不一样，但是原理都是一样的。
# 1.hash
关键是监控两个事件，一个是页面加载进来的时候触发`load`,一个是hash改变的时候触发`hashchange`。
```javascript
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <ul>
        <li><a href="#/">turn white</a></li>
        <li><a href="#/blue">turn blue</a></li>
        <li><a href="#/green">turn green</a></li>
    </ul>
    <script>
    class Router {
        constructor() {
            this.routes = {};
            this.curUrl = "";
        }
        route(path, callback) {
            this.routes[path] = callback || function() {};
        }
        refresh() {
            this.curUrl = location.hash.slice(1) || '/';
            this.routes[this.curUrl]();
        }
        init() {
            window.addEventListener('load', this.refresh.bind(this), false);
            window.addEventListener('hashchange', this.refresh.bind(this), false);
        }

    }
    var router = new Router();
    router.init();
    var content = document.querySelector('body');
    // change Page anything
    function changeBgColor(color) {
        content.style.backgroundColor = color;
    }
    router.route('/', function() {
        changeBgColor('white');
    });
    router.route('/blue', function() {
        changeBgColor('blue');
    });
    router.route('/green', function() {
        changeBgColor('green');
    });
    </script>
</body>

</html>

```

# 2.history api
html5 增加了两个方法，分别是`pushState`，`replaceState`.

两个方法均有三个参数：一个状态对象、一个标题（现在会被忽略），一个可选的URL地址
**状态对象（state object）** — 一个JavaScript对象，与用pushState()方法创建的新历史记录条目关联。无论何时用户导航到新创建的状态，popstate事件都会被触发，并且事件对象的state属性都包含历史记录条目的状态对象的拷贝。

任何可序列化的对象都可以被当做状态对象。因为FireFox浏览器会把状态对象保存到用户的硬盘，这样它们就能在用户重启浏览器之后被还原，我们强行限制状态对象的大小为640k。如果你向pushState()方法传递了一个超过该限额的状态对象，该方法会抛出异常。如果你需要存储很大的数据，建议使用sessionStorage或localStorage。

pushState 用于向 history 添加当前页面的记录，而 replaceState 和 pushState 的用法完全一样，唯一的区别就是它用于修改当前页面在 history 中的记录。

**两者的一个表现的区别是**：在浏览器上点击后退键的时候，使用pushState的会正常按照点击的顺序依次返回，而使用replaceState的只是替换，不会返回，会直接返回到pushState的记录。

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Simple History</title>
</head>
<body>
    <a class="push" href="?push-one">Push One</a>
    <a class="push" href="?push-two">Push Two</a>
    <a class="push" href="?push-three">Push Three</a>
    <a class="replace" href="?replace-one">Replace One</a>
    <a class="replace" href="?replace-two">Replace Two</a>
    <a class="replace" href="?replace-three">Replace Three</a>
    <ul id="log"></ul>
    <script src="simple-history.js"></script>
    <script src="https://cdn.bootcss.com/jquery/1.7.1/jquery.min.js"></script>
    <script>
    (function() {
        if (!SimpleHistory.supported) {
            return;
        }
        SimpleHistory.start(function(path) {
            console.log("match", path);
            document.title = "Simple History - " + path;
            $("<li>").text("match: " + path).appendTo("#log");
        });
        $("a:not([href^=http])").click(function(event) {
            if (event.metaKey || event.shiftKey || event.ctrlKey) {
                return;
            }
            event.preventDefault();
            var path = $(event.target).attr("href");
            if ($(event.target).is(".push")) {
                SimpleHistory.pushState(event.target.href);
            } else {
                SimpleHistory.replaceState(event.target.href);
            }
        });
    }())
    </script>
</body>

</html>
```
**simple-history.js**
```javascript
(function(window, undefined) {

    var initial = location.href;

    window.SimpleHistory = {
        supported: !!(window.history && window.history.pushState),
        pushState: function(fragment, state) {
            state = state || {};
            history.pushState(state, null, fragment);
            this.notify(state);
        },
        replaceState: function(fragment, state) {
            state = state || {};
            history.replaceState(state, null, fragment);
        },
        notify: function(state) {
            console.log(location.pathname,location.search);
            this.matcher(location.pathname + location.search, state);
        },
        start: function(matcher) {
            this.matcher = matcher;
            window.addEventListener("popstate", function(event) {
                // workaround to always ignore first popstate event (Chrome)
                // a timeout isn't reliable enough
                if (initial && initial === location.href) {
                    initial = null;
                    return;
                }
                SimpleHistory.notify(event.state || {});
            }, false);
        }
    };

}(window));
```


**参考阅读：**
- [ 原生JS实现一个简单的前端路由（路由实现的原理）](http://blog.csdn.net/sunxinty/article/details/52586556)
- [从 React Router 谈谈路由的那些事](http://blog.csdn.net/u013063153/article/details/52513872)