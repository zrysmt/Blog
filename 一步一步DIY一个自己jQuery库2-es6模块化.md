---
title: 一步一步DIY一个自己jQuery库2-es6模块化
tags:    
- FE
- jQuery
- js原生实现库    
categories: 前端技术
---

有关rollup的安装使用可以查看我的另外一篇博客：[《rollup + es6最佳实践》](http://blog.csdn.net/future_todo/article/details/52725176),这里有提供了简明的搭建环境的说明，我们可以暂时忽略这篇博客，先参见第一部分`1.环境搭建`

我们首先把《一步一步DIY一个自己jQuery库1》的文件打包好

所有代码挂在我的[github](https://github.com/zrysmt/DIY-jQuery)中
# 1.搭建环境
## 1.1 目录结构
```
  - src
    + .babelrc
    + core.js
    + global.js
    + init.js
    + jquery.js
    + util.js
  bundle.js
  package.json
  rollup.config.dev.js
  test.html
```
- src是源代码文件夹，其中`jquery.js`是入口文件
- bundle是编译后的文件
- package.json是包管理文件
- ` rollup.config.dev.js`是rollup的配置文件
- `test.html`是测试文件,引入`<script src="bundle.js"></script>`即可测试

## 1.2 npm安装

```bash
npm i rollup rollup-plugin-babel babel-preset-es2015-rollup --save-dev
```
## 1.3 使用配置编译
新建文件，文件名为`rollup.config.dev.js`

```javascript
import babel from 'rollup-plugin-babel';

export default {
  entry: 'src/jquery.js',
  format: 'umd',
  moduleName: 'jQuery',
  plugins: [babel() ],
  dest: 'bundle.js',
};
```
src中`.babelrc`

```javascript
{
  presets: [
    ["es2015", { "modules": false }]
  ]
}
```
注意` { "modules": false }`一定要有，否则一直报错
**执行命令：**`rollup -c rollup.config.dev.js`,就能得到编译后的文件`bundle.js`。这里使用的是【umd】的形式，这是jquery的发布版本的格式，当然还有其他的一些格式，amd / es6 / cjs / iife

# 2.打包
`jquery.js`

```javascript
// 出口
import jQuery from './core';
import global from './global';
import init from './init';
 
global(jQuery);
init(jQuery);
 
export default jQuery;
```
`core.js`

```javascript
var version = "0.0.1",
    jQuery = function(selector, context) {
        return new jQuery.fn.init(selector, context);
    };

jQuery.fn = jQuery.prototype = {
    jquery: version,
    constructor: jQuery,
    setBackground: function() {
        this[0].style.background = 'yellow';
        return this;
    },
    setColor: function() {
        this[0].style.color = 'blue';
        return this;
    }
};

jQuery.extend = jQuery.fn.extend = function() {
    var isObject = function(obj) {
        return Object.prototype.toString.call(obj) === "[object Object]";
    };
    var isArray = function(obj) {
        return Object.prototype.toString.call(obj) === "[object Array]";
    };
    var name, clone, copy, copyIsArray ,options, i = 1,
        length = arguments.length,
        target = arguments[0] || {},
        deep = false; //默认为浅复制

    if (typeof target === "boolean") {
        deep = target;
        target = arguments[i] || {};
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
            for (name in options) {
                src = target[name]; //jQuery是否已经有该属性
                copy = options[name];
                if (target === copy) {
                    continue;
                }
                //深拷贝，且确保被拷属性为对象/数组
                if (deep && copy && isObject(copy) || (copyIsArray = isArray(copy))) {
                    //被拷贝属性为数组
                    if (copyIsArray) {
                        copyIsArray = false;
                        //被合并属性
                        clone = src && isArray(src) ? src : [];
                    } else { //被拷贝属性为对象
                        clone = src && isObject(src) ? src : {};
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
export default jQuery;
```
`init.js`

```javascript
var init = function(jQuery){
    jQuery.fn.init = function (selector, context, root) {
        if (!selector) {
            return this;
        } else {
            var elem = document.querySelector(selector);
            if (elem) {
                this[0] = elem;
                this.length = 1;
            }
            return this;
        }
    };
 
    jQuery.fn.init.prototype = jQuery.fn;
};
export default init;
```
`global.js`

```javascript
var global = function(jQuery){
    //走模块化形式的直接绕过
    if(typeof exports === 'object'&&typeof module !== 'undefined') return;
    var _jQuery = window.jQuery,
        _$ = window.$; 
    jQuery.noConflict = function( deep ) {
        //确保window.$没有再次被改写
        if ( window.$ === jQuery ) {
            window.$ = _$;
        }
        //确保window.jQuery没有再次被改写
        if ( deep&&window.jQuery === jQuery ) {
            window.jQuery = _jQuery;
        }
        return jQuery;  //返回 jQuery 接口引用
    };
 
    window.jQuery = window.$ = jQuery;
};
 
export default global;
```
打包后`bundle.js`

```javascript
(function (global, factory) {
    typeof exports === 'object' && typeof module !== 'undefined' ? module.exports = factory() :
    typeof define === 'function' && define.amd ? define(factory) :
    (global.jQuery = factory());
}(this, (function () { 'use strict';

var _typeof = typeof Symbol === "function" && typeof Symbol.iterator === "symbol" ? function (obj) { return typeof obj; } : function (obj) { return obj && typeof Symbol === "function" && obj.constructor === Symbol ? "symbol" : typeof obj; };

var version = "0.0.1";
var jQuery$1 = function jQuery$1(selector, context) {

    return new jQuery$1.fn.init(selector, context);
};

jQuery$1.fn = jQuery$1.prototype = {
    jquery: version,
    constructor: jQuery$1,
    setBackground: function setBackground() {
        this[0].style.background = 'yellow';
        return this;
    },
    setColor: function setColor() {
        this[0].style.color = 'blue';
        return this;
    }
};

jQuery$1.extend = jQuery$1.fn.extend = function () {
    var isObject = function isObject(obj) {
        return Object.prototype.toString.call(obj) === "[object Object]";
    };
    var isArray = function isArray(obj) {
        return Object.prototype.toString.call(obj) === "[object Array]";
    };
    var name, clone, copy, copyIsArray, options,
        i = 1,
        length = arguments.length,
        target = arguments[0] || {},
        deep = false; //默认为浅复制

    if (typeof target === "boolean") {
        deep = target;
        target = arguments[i] || {};
        i++;
    }
    if ((typeof target === 'undefined' ? 'undefined' : _typeof(target)) !== "object" && typeof target !== "function") {
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
                if (deep && copy && isObject(copy) || (copyIsArray = isArray(copy))) {
                    //被拷贝属性为数组
                    if (copyIsArray) {
                        copyIsArray = false;
                        //被合并属性
                        clone = src && isArray(src) ? src : [];
                    } else {
                        //被拷贝属性为对象
                        clone = src && isObject(src) ? src : {};
                    }
                    //右侧递归，直到内部属性值是非对象
                    target[name] = jQuery$1.extend(deep, clone, copy);
                } else if (copy !== undefined) {
                    //非对象/数组，或者浅复制的情况
                    target[name] = copy; //递归结束
                }
            }
        }
    }

    //返回修改后的target

    return target;
};

var _typeof$1 = typeof Symbol === "function" && typeof Symbol.iterator === "symbol" ? function (obj) { return typeof obj; } : function (obj) { return obj && typeof Symbol === "function" && obj.constructor === Symbol ? "symbol" : typeof obj; };

var global = function global(jQuery) {
    //走模块化形式的直接绕过
    if ((typeof exports === 'undefined' ? 'undefined' : _typeof$1(exports)) === 'object' && typeof module !== 'undefined') return;

    var _jQuery = window.jQuery,
        _$ = window.$;

    jQuery.noConflict = function (deep) {
        //确保window.$没有再次被改写
        if (window.$ === jQuery) {
            window.$ = _$;
        }

        //确保window.jQuery没有再次被改写
        if (deep && window.jQuery === jQuery) {
            window.jQuery = _jQuery;
        }

        return jQuery; //返回 jQuery 接口引用
    };

    window.jQuery = window.$ = jQuery;
};

var init = function init(jQuery) {
    jQuery.fn.init = function (selector, context, root) {
        if (!selector) {
            return this;
        } else {
            var elem = document.querySelector(selector);
            if (elem) {
                this[0] = elem;
                this.length = 1;
            }
            return this;
        }
    };

    jQuery.fn.init.prototype = jQuery.fn;
};

// 出口
global(jQuery$1);
init(jQuery$1);

return jQuery$1;

})));
```
# 3.增加基础工具模块&完善extend方法
我们在《一步一步DIY一个自己jQuery库1》中说过extend方法的不完善的地方
- 使用`isObject`,`isArray`并不严谨。在某些浏览器中，像 document 在 Object.toSting 调用时也会返回和 Object 相同结果；

新增一个`util.js`

```javascript
export var class2type = {}; //在core.js中会被赋予各类型属性值
export const toString = class2type.toString; //等同于 Object.prototype.toString
export const getProto = Object.getPrototypeOf;
export const hasOwn = class2type.hasOwnProperty;
export const fnToString = hasOwn.toString; //等同于 Object.toString/Function.toString
export const ObjectFunctionString = fnToString.call(Object); //顶层Object构造函数字符串"function Object() { [native code] }"，用于判断 plainObj
```
在`core.js`中修改/新增代码
- 导入

```javascript
import { class2type, toString, getProto, hasOwn, fnToString, ObjectFunctionString } from './util.js';
```
- 修改

```javascript
typeof target !== "function"   //修改为!jQuery.isFunction(target)
isArray                        //均修改为jQuery.isArray
isObject                       //均修改为jQuery.isObject
```
- 新增

```javascript
//新增修改点1，class2type注入各JS类型键值对，配合 jQuery.type 使用，后面会用上
"Boolean Number String Function Array Date RegExp Object Error Symbol".split(" ").forEach(function(name) {
    class2type["[object " + name + "]"] = name.toLowerCase();
});
//新增修改点2
jQuery.extend({
    isArray: Array.isArray || function( obj ) {
        return jQuery.type(obj) === "array";
    },
    isFunction: function( obj ) {
        return jQuery.type(obj) === "function";
    },
    isPainObject: function(obj) {
        var proto, Ctor;

        if (!obj || toString.call(obj) !== "[object Object]") {
            return false;
        }

        proto = getProto(obj);
        // 通过 Object.create( null ) 形式创建的 {} 是没有prototype的
        if (!proto) {
            return true;
        }

        //简单对象的构造函数等于最顶层Object构造
        Ctor = hasOwn.call(proto, "constructor") && proto.constructor;
        return typeof Ctor === "function" && fnToString.call(Ctor) === ObjectFunctionString;
    },

    type: function(obj) {
        if (obj == null) { //不能用 === 
            return obj + ""; //undefined or null
        }

        return typeof obj === "object" || typeof obj === "function" ?
            //兼容安卓2.3- 函数表达式类型不正确情况
            class2type[toString.call(obj)] || "object" :
            typeof obj;
    }
});
```
修改后的文件我已经挂在了我的[github](https://github.com/zrysmt/DIY-jQuery)中，对应文件夹是`v2`.

参考阅读：
- [从零开始，DIY一个jQuery（1）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551118&idx=1&sn=be4df567418db97c9b3d8a7ab5314e01&scene=1&srcid=0810SD0bcpyNFVUcmgsy5kh7#rd)
- [从零开始，DIY一个jQuery（2）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551123&idx=2&sn=26ddfeb73928eded3a63f05ca5273d66&scene=1&srcid=0811ZOMbQiHHtIIHRBceFbp7#rd)
- [从零开始，DIY一个jQuery（3）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551144&idx=2&sn=78d8eec17bcdfa5bf51b5ec14c15c474&scene=1&srcid=0817O7pEiReCTSKGByB8SSIq#rd)




