---
title: 前端CSS&JS动画总结
tags:    
- FE
- CSS3动画
- JS动画    
categories: 前端技术
---
使用CSS3，我们可以很方便快捷的改变元素的宽度、高度，方位，角度，透明度等基本信息，但是这些不能满足我们的需求，而且浏览器对于CSS3的兼容性不好，所以这时候就需要拓展更多的js动画。
# 1.CSS3动画

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CSS3 动画</title>
    <style type="text/css">
    #taxiway {
        width: 800px;
        height: 100px;
        background: #E8E8FF;
        position: relative;
    }
    #move,
    #move2{
        position: absolute;
        left: 0px;
        width: 100px;
        height: 100px;
        background: #a9ea00;
    }
    #move3 {
        position: absolute;
        left: 0px;
        width: 50px;
        height: 50px;
        background: #a9ea00;
    }
    .animate {
        animation-duration: 3s;
        animation-name: slidein;
        animation-timing-function: ease-in-out;
        animation-iteration-count: 2;        /* 几次 */
        animation-fill-mode: forwards;
    }
    @keyframes slidein {
        from {
            left: 0%;
            background: white;
        }
        to {
            left: 700px;
            background: red;
        }
    }    
    .animate2 {
        animation-duration: 3s;
        animation-name: cycle;
        animation-iteration-count: 2;
        animation-direction: alternate;
    }   
    @keyframes cycle {
        to {
            width: 200px;
            height: 200px;
        }
    }
    </style>
</head>
<body>
    <div id="taxiway">
        <div id="move"></div>
        <div id="move2" class="animate"></div>
        <div id="move3" class="animate2"></div>
    </div>
</body>
</html>
```
有两个动画，`class="animate"`,`class="animate2"`
第一种动画是：从左边（0%）到右边（700px)处，背景颜色从white变成red，并且来回变换两次（`animation-iteration-count: 2`）；
第二种动画是：`id="move3"`元素从大小为50px，50px,变为200px,200px
# 2.JS动画
最基础的动画刚开始就是利用`setTimeout`和`setInterval`实现的。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>动画</title>
    <style type="text/css">
    #taxiway {
        width: 800px;
        height: 100px;
        background: #E8E8FF;
        position: relative;
    } 
    #move {
        position: absolute;
        left: 0px;
        width: 100px;
        height: 100px;
        background: #a9ea00;
    }
    </style>
</head>

<body>
    <div id="taxiway">
        <div id="move"></div>
    </div>
    <script type="text/javascript" src="startAnimate.js"> 
    </script>
</body>
</html>
```
startAnimate.js

```javascript
window.onload = function() {
    var el = document.getElementById('move');
    var parent = document.getElementById('taxiway');
    var distance = parent.offsetWidth - el.offsetWidth; //总距离
    var begin = parseFloat(window.getComputedStyle(el, null).left); //开始位置
    var end = begin + distance; //结束位置
    var fps = 30;
    var interval = 1000 / fps; //每隔多少ms刷新一次
    var duration = 2000; //时长
    var times = duration / 1000 * fps; //一共刷新这么多次
    var step = distance / times; //每次移距离
    console.log(distance, begin, end);

    el.onclick = function() {
        startAnimate(this);
    }

    function startAnimate(el) {
        var beginTime = new Date();
        var id = setInterval(function() {
            var t = new Date - beginTime;
            if (t >= duration) {
                el.style.left = end + "px";
                clearInterval(id);
                console.info(t);
            } else {
                var per = t / duration; //当前进度  控制per就可以控制加减速
                el.style.left = begin + per * distance + "px";
            }
        }, interval);
    }
}

```
通过控制per的大小变化可以控制加减速，这里我们参照`jquery.easing.js`的曲线函数

```javascript
function bounceOut(x) {
    var n1 = 7.5625,
        d1 = 2.75;
    if (x < 1 / d1) {
        return n1 * x * x;
    } else if (x < 2 / d1) {
        return n1 * (x -= (1.5 / d1)) * x + .75;
    } else if (x < 2.5 / d1) {
        return n1 * (x -= (2.25 / d1)) * x + .9375;
    } else {
        return n1 * (x -= (2.625 / d1)) * x + .984375;
    }
}
var pow = Math.pow,
    sqrt = Math.sqrt,
    sin = Math.sin,
    cos = Math.cos,
    PI = Math.PI,
    c1 = 1.70158,
    c2 = c1 * 1.525,
    c3 = c1 + 1,
    c4 = (2 * PI) / 3,
    c5 = (2 * PI) / 4.5;
var easeSetting = {
    liner: function(x) {
        return x;
    },
    swing: function(x) {
        return 0.5 - cos(x * Math.PI) / 2;
    },
    easeInQuad: function(x) {
        return x * x;
    },
    easeOutQuad: function(x) {
        return 1 - (1 - x) * (1 - x);
    },
    easeInOutQuad: function(x) {
        return x < 0.5 ?
            2 * x * x :
            1 - pow(-2 * x + 2, 2) / 2;
    },
    easeInCubic: function(x) {
        return x * x * x;
    },
    easeOutCubic: function(x) {
        return 1 - pow(1 - x, 3);
    },
    easeInOutCubic: function(x) {
        return x < 0.5 ?
            4 * x * x * x :
            1 - pow(-2 * x + 2, 3) / 2;
    },
    easeInQuart: function(x) {
        return x * x * x * x;
    },
    easeOutQuart: function(x) {
        return 1 - pow(1 - x, 4);
    },
    easeInOutQuart: function(x) {
        return x < 0.5 ?
            8 * x * x * x * x :
            1 - pow(-2 * x + 2, 4) / 2;
    },
    easeInQuint: function(x) {
        return x * x * x * x * x;
    },
    easeOutQuint: function(x) {
        return 1 - pow(1 - x, 5);
    },
    easeInOutQuint: function(x) {
        return x < 0.5 ?
            16 * x * x * x * x * x :
            1 - pow(-2 * x + 2, 5) / 2;
    },
    easeInSine: function(x) {
        return 1 - cos(x * PI / 2);
    },
    easeOutSine: function(x) {
        return sin(x * PI / 2);
    },
    easeInOutSine: function(x) {
        return -(cos(PI * x) - 1) / 2;
    },
    easeInExpo: function(x) {
        return x === 0 ? 0 : pow(2, 10 * x - 10);
    },
    easeOutExpo: function(x) {
        return x === 1 ? 1 : 1 - pow(2, -10 * x);
    },
    easeInOutExpo: function(x) {
        return x === 0 ? 0 : x === 1 ? 1 : x < 0.5 ?
            pow(2, 20 * x - 10) / 2 :
            (2 - pow(2, -20 * x + 10)) / 2;
    },
    easeInCirc: function(x) {
        return 1 - sqrt(1 - pow(x, 2));
    },
    easeOutCirc: function(x) {
        return sqrt(1 - pow(x - 1, 2));
    },
    easeInOutCirc: function(x) {
        return x < 0.5 ?
            (1 - sqrt(1 - pow(2 * x, 2))) / 2 :
            (sqrt(1 - pow(-2 * x + 2, 2)) + 1) / 2;
    },
    easeInElastic: function(x) {
        return x === 0 ? 0 : x === 1 ? 1 :
            -pow(2, 10 * x - 10) * sin((x * 10 - 10.75) * c4);
    },
    easeOutElastic: function(x) {
        return x === 0 ? 0 : x === 1 ? 1 :
            pow(2, -10 * x) * sin((x * 10 - 0.75) * c4) + 1;
    },
    easeInOutElastic: function(x) {
        return x === 0 ? 0 : x === 1 ? 1 : x < 0.5 ?
            -(pow(2, 20 * x - 10) * sin((20 * x - 11.125) * c5)) / 2 :
            pow(2, -20 * x + 10) * sin((20 * x - 11.125) * c5) / 2 + 1;
    },
    easeInBack: function(x) {
        return c3 * x * x * x - c1 * x * x;
    },
    easeOutBack: function(x) {
        return 1 + c3 * pow(x - 1, 3) + c1 * pow(x - 1, 2);
    },
    easeInOutBack: function(x) {
        return x < 0.5 ?
            (pow(2 * x, 2) * ((c2 + 1) * 2 * x - c2)) / 2 :
            (pow(2 * x - 2, 2) * ((c2 + 1) * (x * 2 - 2) + c2) + 2) / 2;
    },
    easeInBounce: function(x) {
        return 1 - bounceOut(1 - x);
    },
    easeOutBounce: bounceOut,
    easeInOutBounce: function(x) {
        return x < 0.5 ?
            (1 - bounceOut(1 - 2 * x)) / 2 :
            (1 + bounceOut(2 * x - 1)) / 2;
    }
};
```
使用很简单

```javascript
el.style.left = begin + easeSetting.easeInOutElastic(per) * distance + "px";
```
但是使用setInterval或setTimeout定时修改DOM、CSS实现动画比较消耗资源，照成页面比较卡顿，所以我们选择使用`requestAnimationFrame`得到连贯的逐帧动画。
# 3.requestAnimationFrame

```javascript
function draw() {
    var per = (new Date - startTime) / duration;
    var left = begin + easeSetting.easeInOutElastic(per) * distance;
    el.style.left = left + "px";
    if (progress < end) {
        requestAnimationFrame(draw);//重绘UI
    }
}
var requestAnimationFrame = window.requestAnimationFrame ||
    window.webkitRequestAnimationFrame ||
    window.mozRequestAnimationFrame || window.msRequestAnimationFrame || function(callback) {
        window.setTimeout(callback, 1000 / 60);
    },
    startTime = window.mozAnimationStartTime || Date.now(),
    progress = 0;
requestAnimationFrame(draw);
```
我们在上面的例子中去兼容所有的浏览器，但是这个还不是很完美，司徒正美给出了几个解决方案，点击[这里](https://github.com/RubyLouvre/jsbook/blob/master/ch14fx.js)进行查看。 [requestAnimationFrame动画控制详解](http://blog.csdn.net/whqet/article/details/42911059?utm_source=tuicool&utm_medium=referral)一文中也提供了几中解决方案，我把支持包括兼容ios6的例子写在这里，以供参考：

```javascript
// requestAnimationFrame polyfill by Erik Möller.
// Fixes from Paul Irish, Tino Zijdel, Andrew Mao, Klemen Slavič, Darius Bacon

// MIT license
if (!Date.now)
    Date.now = function() { return new Date().getTime(); };

(function() {
    'use strict';   
    var vendors = ['webkit', 'moz'];
    for (var i = 0; i < vendors.length && !window.requestAnimationFrame; ++i) {
        var vp = vendors[i];
        window.requestAnimationFrame = window[vp+'RequestAnimationFrame'];
        window.cancelAnimationFrame = (window[vp+'CancelAnimationFrame']
                                   || window[vp+'CancelRequestAnimationFrame']);
    }
    if (/iP(ad|hone|od).*OS 6/.test(window.navigator.userAgent) // iOS6 is buggy
        || !window.requestAnimationFrame || !window.cancelAnimationFrame) {
        var lastTime = 0;
        window.requestAnimationFrame = function(callback) {
            var now = Date.now();
            var nextTime = Math.max(lastTime + 16, now);
            return setTimeout(function() { callback(lastTime = nextTime); },
                              nextTime - now);
        };
        window.cancelAnimationFrame = clearTimeout;
    }
}());
```
# 4.动画库简单介绍
- css库 -- [animate.css](https://github.com/daneden/animate.css)
使用很简单，写在类名中即可

```html
<link rel="stylesheet" href="animate.min.css">
<h1 class="animated infinite bounce">Example</h1>
```
- js库[velocity.js](http://velocityjs.org/)

```javascript
<script src="http://cdn.bootcss.com/jquery/1.12.4/jquery.js"></script>
<script src="velocity.js"></script>
<script>
       $('#move').velocity({opcity: 0.5})
       .delay(1000).velocity({left:"+=400px"})
       .velocity({rotateY:"360deg"},1000)
       .fadeOut('slow', function() {
           console.log("fadeout")
       });;
</script>
```


参考阅读：
- [CSS vs JS动画：谁更快？](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651550857&idx=1&sn=60049d0ec60aaa118fdc48691a659f0b&scene=1&srcid=0516KJfJZgtSAwVdmiwUOBGM#rd)
- [https://github.com/gdsmith/jquery.easing](https://github.com/gdsmith/jquery.easing)
- [http://gsgd.co.uk/sandbox/jquery/easing/](http://gsgd.co.uk/sandbox/jquery/easing/)
- [window.requestAnimationFrame--MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)
- 《司徒正美-javascript框架设计--第十四章 动画引擎》
- [requestAnimationFrame动画控制详解](http://blog.csdn.net/whqet/article/details/42911059?utm_source=tuicool&utm_medium=referral)
- [CSS3动画那么强，requestAnimationFrame还有毛线用？](http://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-%E5%8A%A8%E7%94%BB%E7%AE%97%E6%B3%95/)
- [玩转HTML5移动页面(动效篇)](http://isux.tencent.com/play-with-html5-animate.html)