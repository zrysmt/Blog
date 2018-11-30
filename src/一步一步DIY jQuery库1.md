---
title: 一步一步DIY jQuery库1
tags:    
- FE
- jQuery
- js原生实现库    
categories: 前端技术
date: 2016-10-02 00:00:00
---

前一段时间，看到一篇系列文章《从零开始，DIY一个jQuery 1-3》三篇文章从头讲解如何DIY一个jQuery，感觉挺有意思，今天想试一试看看。
  
我在前端已经有了一年的经验，jquery几乎是每天都会使用到，但是现在还没时间去研究一下它的源码，即使这样我也想动手尝试下DIY一个JQuery库，我相信这可以加深我对jquery的理解，当然我也试着去学习jquery的源码，这很可能会写成一个系列的文章，这个系列文章以一个入门的jquery原理探索者的视角。当然，这篇文章作为入门的入门。
  
【注】所有代码挂在我的[github](https://github.com/zrysmt/DIY-jQuery)上
# 1.实现一个基本的框架
## 1.1 整体
- 整体是个闭包，独立作用域，避免污染全局变量
- 不用new，是因为使用了工厂模式，new放在了内部`return new jQuery.fn.init(selector)`
- jQuery是最基础的对象，方法放在了`jQuery.prototype`中，即是`jQuery.fn`
- $.extend / $.fn.extend 来扩展静态方法和原型方法
- 使用全局变量`window.$`, `window.jQuery`即可调用
- `return this`为了链式调用

```javascript
//IIFE 独立作用域
    (function() {
        var version = '0.0.1';

        jQuery = function(selector) {
            return new jQuery.fn.init(selector); //jQuery实例的构造函数已经变成了 jQuery.fn.init 
        };

        /*jQuery.fn   主要方法*/
        jQuery.fn = jQuery.prototype = {
            jquery: version,
            construct: jQuery,
            //方法
            setBackground: function(color) {
                this[0].style.background = color;
                return this;  //链式调用
            },
        };

        var init = jQuery.fn.init = function(selector) {
            if (!selector) {
                return this;
            } else {
                var elem = document.querySelector(selector);
                if (elem) {
                    this[0] = elem;
                    this.length = 1;
                }
                console.info(this);
                return this;
            }
        };
        init.prototype = jQuery.prototype; //把 jQuery.fn.init 的原型指向 jQuery 的原型（jQuery.prototype / jQuery.fn）即可

        window.$ = window.jQuery = jQuery;
    })();
```

**测试**
```
var $div = $('div');
console.log($div);
$div.setBackground('blue');
console.log($div.jquery); //0.0.1
console.log($.fn.jquery); //0.0.1
```
$div的结果

```
- j…y.fn.init {0: div, length: 1}
  + 0:div
  + length:1
```
## 1.2 冲突处理

```javascript
/*冲突处理*/
var _jQuery = window.jQuery,
_$ = window.$;
//deep 参数类型为 Boolean，若为真，表示要求连window.jQuery 变量都需要吐回去
jQuery.noConflict = function(deep) {
    if (window.$ === jQuery) {
    window.$ = _$;
}
//确保window.jQuery没有再次被改写
if (deep && window.jQuery === jQuery) {
    window.jQuery = _jQuery;
}
return jQuery; //返回 jQuery 接口引用
};
```
**测试**
```
var $$$ = jQuery.noConflict();
$$$('div').setBackground('red');
```
## 1.3 `$.extend` / `$.fn.extend` 来扩展静态方法和原型方法

```javascript
jQuery.extend = jQuery.fn.extend = function() {
     var target = arguments[0] || {};
     for(var key in target){
         jQuery.fn[key] = target[key];
     }
};
```
但是调用必须是`$().min`而不能是`$.min`
```
jQuery.extend({
    min: function(a, b) {
        return a < b ? a : b;
    },
    max: function(a, b) {
        return a > b ? a : b;
    }
});
console.log($().min(3,5));
```
当然这只是个人的一些想法，我们来仿照jquery源码实现

```javascript
var isObject = function(obj) {
    return Object.prototype.toString.call(obj) === "[object Object]";
};
var isArray = function(obj) {
    return Object.prototype.toString.call(obj) === "[object Array]";
};
var options, i = 1,
length = arguments.length,
target = arguments[0] || {},
deep = false; //默认为浅复制

if (typeof target === "boolean") {
    deep = target;
    taeget = arguments[i] || {};
    i++;
}
if (typeof target !== "object" && typeof target !== "function") {
    target = {};
}

//target后面没有其他参数了(要拷贝的对象)，直接扩展jQuery自身，target并入jQuery
if (i === length) {
    target = this;
    i--;
}
for (; i < length; i++) {
    if ((options = arguments[i]) != null) {
    var name, clone, copy;
    for (name in options) {
        src = target[name]; //jQuery是否已经有该属性
        copy = options[name];
        if (target === copy) {
           continue;
         }
       //1-深拷贝，且确保被拷属性为对象/数组
       if (deep && copy && (isObject(copy)||(copyIsArray = isArray(copy)))){
       //被拷贝属性为数组
          if(copyIsArray){
              copyIsArray = false;
              //被合并属性
              clone = src && isArray(src)?src:[];
          }else{//被拷贝属性为对象
              clone = src && isArray(src)?src:{};
          }
           //右侧递归，直到内部属性值是非对象
         target[name] = jQuery.extend(deep,clone,copy);
      }else if(copy!==undefined){//2-非对象/数组，或者浅复制的情况
         target[name] = copy;//递归结束
       }
    }
  }
}
//返回修改后的target
return target;
```
拷贝主要分为两个部分：

```javascript
//1-深拷贝，且确保被拷属性为对象/数组
if (deep && copy && isObject(copy)||(copyIsArray = isArray(copy))){
    /*...*/}
//2-非对象/数组，或者浅复制的情况
else if(copy!==undefined){
}
```
使用的几种情况

```javascript
$.extend(  targetObj,  copyObj1[,  copyObj2...]  )
$.extend(  true,  targetObj,  copyObj1[,  copyObj2...]  )
$.extend(  copyObj  )
$.extend(  true,  copyObj
```
测试

```javascript
 jQuery.extend({
     min: function(a, b) {
         return a < b ? a : b;
     },
     max: function(a, b) {
         return a > b ? a : b;
     }
});
console.log($.min(3, 5));
```
注意：这个时候只是使用`isObject`,`isArray`并不严谨
在某些浏览器中，像 document 在 Object.toSting 调用时也会返回和 Object 相同结果；
这些我们将在《一步一步DIY一个自己jQuery库2》中进行补充。

# 2.全部代码

```javascript
//IIFE 独立作用域
    (function() {
        var version = '0.0.1';

        jQuery = function(selector) {
            return new jQuery.fn.init(selector); //jQuery实例的构造函数已经变成了 jQuery.fn.init 
        };

        /*冲突处理*/
        var _jQuery = window.jQuery,
            _$ = window.$;
        //deep 参数类型为 Boolean，若为真，表示要求连window.jQuery 变量都需要吐回去
        jQuery.noConflict = function(deep) {
            if (window.$ === jQuery) {
                window.$ = _$;
            }
            //确保window.jQuery没有再次被改写
            if (deep && window.jQuery === jQuery) {
                window.jQuery = _jQuery;
            }

            return jQuery; //返回 jQuery 接口引用
        };

        /*jQuery.fn   主要方法*/
        jQuery.fn = jQuery.prototype = {
            jquery: version,
            construct: jQuery,
            //方法
            setBackground: function(color) {
                this[0].style.background = color;
                console.warn(this);
                return this;
            },
        };

        var init = jQuery.fn.init = function(selector) {
            if (!selector) {
                return this;
            } else {
                var elem = document.querySelector(selector);
                if (elem) {
                    this[0] = elem;
                    this.length = 1;
                }
                console.info(this);
                return this;
            }
        };
        init.prototype = jQuery.prototype; //把 jQuery.fn.init 的原型指向 jQuery 的原型（jQuery.prototype / jQuery.fn）即可

        jQuery.extend = jQuery.fn.extend = function() {
            var isObject = function(obj) {
                return Object.prototype.toString.call(obj) === "[object Object]";
            };
            var isArray = function(obj) {
                return Object.prototype.toString.call(obj) === "[object Array]";
            };
            var options, i = 1,
                length = arguments.length,
                target = arguments[0] || {},
                deep = false; //默认为浅复制

            if (typeof target === "boolean") {
                deep = target;
                taeget = arguments[i] || {};
                i++;
            }
            if (typeof target !== "object" && typeof target !== "function") {
                target = {};
            }

            //target后面没有其他参数了(要拷贝的对象)，直接扩展jQuery自身，target并入jQuery
            if (i === length) {
                target = this;
                i--;
            }
            for (; i < length; i++) {
                if ((options = arguments[i]) != null) {
                    var name, clone, copy;
                    for (name in options) {
                        src = target[name]; //jQuery是否已经有该属性
                        copy = options[name];
                        if (target === copy) {
                            continue;
                        }
                        //深拷贝，且确保被拷属性为对象/数组
                        if (deep && copy && (isObject(copy) || (copyIsArray = isArray(copy)))) {
                            //被拷贝属性为数组
                            if (copyIsArray) {
                                copyIsArray = false;
                                //被合并属性
                                clone = src && isArray(src) ? src : [];
                            } else { //被拷贝属性为对象
                                clone = src && isArray(src) ? src : {};
                            }
                            //右侧递归，直到内部属性值是非对象
                            target[name] = jQuery.extend(deep, clone, copy);
                        } else if (copy !== undefined) { //非对象/数组，或者浅复制的情况
                            target[name] = copy; //递归结束
                        }
                    }
                }
            }

            //返回修改后的target

            return target;

        };

        window.$ = window.jQuery = jQuery;
    })();

    //测试
    var $div = $('div');
    console.log($div);
    $div.setBackground('blue');
    console.log($div.jquery); //0.0.1
    console.log($.fn.jquery); //0.0.1
    console.log(jQuery.extend());
    console.log($.extend.jquery);
    //冲突
    /*var $$$ = jQuery.noConflict();
    $$$('div').setBackground('red');*/
    //
    jQuery.extend({
        min: function(a, b) {
            return a < b ? a : b;
        },
        max: function(a, b) {
            return a > b ? a : b;
        }

    });
    console.info(jQuery.prototype);
    console.info(jQuery);
    // console.log($().min(3,5));
    // console.log($.prototype.min(3, 5));
    console.log($.min(3, 5));
```
参考阅读：
- [从零开始，DIY一个jQuery（1）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551118&idx=1&sn=be4df567418db97c9b3d8a7ab5314e01&scene=1&srcid=0810SD0bcpyNFVUcmgsy5kh7#rd)
- [从零开始，DIY一个jQuery（2）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551123&idx=2&sn=26ddfeb73928eded3a63f05ca5273d66&scene=1&srcid=0811ZOMbQiHHtIIHRBceFbp7#rd)
- [从零开始，DIY一个jQuery（3）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551144&idx=2&sn=78d8eec17bcdfa5bf51b5ec14c15c474&scene=1&srcid=0817O7pEiReCTSKGByB8SSIq#rd)