---
title: 一步一步DIY一个自己jQuery库3-jquery+sizzle引擎
tags:    
- FE
- jQuery
- js原生实现库    
categories: 前端技术
---

所有代码挂在我的[github](https://github.com/zrysmt/DIY-jQuery)中

在前两篇的基础上，正式引入sizzle引擎，我们不详细介绍该引擎，会使用即可。

我们在前两篇的`jQuery.fn.init`的方法是

```javascript
var init = function(jQuery){
    jQuery.fn.init = function (selector, context) {
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
```
这里得到的结果是个数组，而不是我们需要的类数组结构的DOM集合**HTMLCollection**

# 1.引入Sizzle引擎
下载Sizzle，将`sizzle.js`文件复制在`src/sizzle`中，并且改造Sizzle成模块化的
```javascript
//1-头部注释
//(function( window ) { 

//2-最后尾部注释
/*if ( typeof define === "function" && define.amd ) {
    define(function() { return Sizzle; });
// Sizzle requires that there be a global window in Common-JS like environments
} else if ( typeof module !== "undefined" && module.exports ) {
    module.exports = Sizzle;
} else {
    window.Sizzle = Sizzle;
}
// EXPOSE

})( window );*///修改点2

//3-增加
export default Sizzle;
```
同时增加一个初始化文件`src/sizzle/init.js`
```javascript
import Sizzle from './sizzle.js';

var selectorInit  = function(jQuery){
    jQuery.find = Sizzle;  // Sizzle 赋予静态接口 jQuery.find
}

export default selectorInit;
```
我们可以在jquery源码中找到全部将sizzle赋值的语句，这些我们暂时先不管

```javascript
jQuery.find = Sizzle;
jQuery.expr = Sizzle.selectors;
jQuery.expr[":"] = jQuery.expr.pseudos;
jQuery.unique = Sizzle.uniqueSort;
jQuery.text = Sizzle.getText;
jQuery.isXMLDoc = Sizzle.isXML;
jQuery.contains = Sizzle.contains;
```
修改`jquery.js`

```javascript
import jQuery from './core';
import global from './global';
import init from './init';
import sizzleInit from './sizzle/init';  //新增
 
global(jQuery);
init(jQuery);
sizzleInit(jQuery);  //新增
 
export default jQuery;
```
测试：
```html
<div><span>1</span></div>
<div><span>2</span></div>
<div>3</div>
```
```javascript
var div = $('div');
console.log(div);
```
最后的结果仍然是个DOM集合数组

# 2.`$.merger`方法

```javascript
jQuery.fn = jQuery.prototype = {
    length: 0,  // 修改点1，jQuery实例.length 默认为0,这句一定要有，否则length会NAN
```
```javascript
jQuery.extend({
    merge: function(first, second) {//新增
        var len = +second.length,
            j = 0,
            i = first.length;
        for (; j < len; j++) {
            first[i++] = second[j];
        }
        first.length = i;

        return first;
    }
});
```
测试
```javascript
    var divs = $.find('div'); //纯数组
    var $div1 = $.merge(['hi'], divs); //右边的数组合并到左边的数组，形成一个新数组
    var $div2 = $.merge({
        0: 'hi',
        length: 1
    }, divs); //右边的数组合并到左边的对象，形成一个新的类数组对象

    console.log($div1);
    console.log($div2);
```
我们发现，只需要将`$.merger`的第一个参数first设置为this（jQuery的示例对象，length已经默认设置为0），第二个参数second设置为搜索到的DOM集合就可以得到DOM集合类数组对象。
修改`src/init.js`
```javascript
var init = function(jQuery){
    jQuery.fn.init = function (selector, context) {
        if (!selector) {
            return this;
        } else {
            var elemList = jQuery.find(selector);
            if (elemList.length) {
                jQuery.merge( this, elemList );  //this是jQuery实例，默认实例属性 .length 为0
            }
            return this;
        }
    };
 
    jQuery.fn.init.prototype = jQuery.fn;
};
export default init;
```
# 3.扩展 `$.fn.find`

我们虽然能使用$.find方法，但是它并不支持链式，所以我们需要扩展之。

```javascript
jQuery.fn.extend({
    find: function(selector) {
        var i, ret,
            len = this.length,
            self = this;
        ret = [];

        for (i = 0; i < len; i++) {
            jQuery.find(selector, self[i], ret); // //直接利用 Sizzle 接口，把结果注入到 ret 数组中去
        }
        return jQuery.merger(this,ret);
    }
}
```
# 4.记录栈-pushStack方法

参考浏览器的历史记录栈，将检索到的jQuery实例放入到栈中，方便存取数据，其中jquery中有一个方法`$.end`放回上次检索的jQuery对象，使用记录栈能够很方便的实现。

```javascript
jQuery.fn = jQuery.prototype = {
    /**新增：
     * [pushStack 入栈操作]
     * @param  {[Array]} elems 
     * @return {[*]}  
     */
    pushStack: function(elems) {
        var ret = jQuery.merge(this.constructor(), elems); //this.constructor() 返回了一个 length 为0的jQuery对象
        ret.prevObject = this;
        return ret;
    }
};
```
重新修改第3部分的代码

```javascript
jQuery.fn.extend({
    find: function(selector) {
        var i, ret,
            len = this.length,
            self = this;
        ret =[];
        for (i = 0; i < len; i++) {
            jQuery.find(selector, self[i], ret); // //直接利用 Sizzle 接口，把结果注入到 ret 数组中去
        }
        return  this.pushStack(ret);
    },
});
```
从性能上考虑，改为这样，建设merge里面的遍历

```javascript
jQuery.fn.extend({
    find: function(selector) {
        var i, ret,
            len = this.length,
            self = this;
        ret = this.pushStack([]);
        for (i = 0; i < len; i++) {
            jQuery.find(selector, self[i], ret); 
        }
        return ret;
    }
});
```
# 5.`$.fn.end`、`$.fn.eq` 和 `$.fn.get`

```javascript
jQuery.fn.extend({
    end: function() {
        return this.prevObject || this.constructor();//this.prevObject记录栈中存在
    },
    eq: function(i) {
        var len = this.length;
        var j = +i + (i < 0 ? len : 0);
        return this.pushStack(j >= 0 && j < len ? [this[j]] : []);
    },
    get: function(num) {
        return num != null ?
            // 支持倒序搜索，num可以是负数
            (num < 0 ? this[num + this.length] : this[num]) :
            // 克隆一个新数组，避免指向相同
            [].slice.call(this); 
    },
    first: function() {
        return this.eq(0);
    },
    last: function() {
        return this.eq(-1);
    }
});
```

# 6.todolist

我们花了三篇博客写到这里，其实还是有很多没有完全实现，后面的部分也是参照jquery的源码，DIY一个自己的jquery，还有一些没有实现的点
- `$.fn.init`第二个参数context上下文还没实现
- `$.fn.find`返回结果中可能带着重复的DOM
  例如：    
  
  ```html
  <div><div><span>hi</span></div></div>
  <script>
    var  $span = $('div').find('span');
    console.log($span);  //返回两个span
  </script>
  ```
下面的部分留作再写几篇博客

参考阅读：
- [从零开始，DIY一个jQuery（1）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551118&idx=1&sn=be4df567418db97c9b3d8a7ab5314e01&scene=1&srcid=0810SD0bcpyNFVUcmgsy5kh7#rd)
- [从零开始，DIY一个jQuery（2）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551123&idx=2&sn=26ddfeb73928eded3a63f05ca5273d66&scene=1&srcid=0811ZOMbQiHHtIIHRBceFbp7#rd)
- [从零开始，DIY一个jQuery（3）](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551144&idx=2&sn=78d8eec17bcdfa5bf51b5ec14c15c474&scene=1&srcid=0817O7pEiReCTSKGByB8SSIq#rd)
