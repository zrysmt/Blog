---
title: 一步一步DIY zepto库，研究zepto源码2
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---
我们接着上一篇博客继续完成`zepto.init`的其余内容。

基于Zepto 1.2.0版本。
> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，第一篇博客对应文件夹v0.2。
> https://github.com/zrysmt/DIY-zepto

整体的流程是：
*   有传入context，回调自身：`$(context).find(selector)`
*   selector参数为空，直接调用$.zepto.Z方法获取Z对象：`zepto.Z()`
*   selector参数为html片段，调用$.zepto.fragment方法获取对应DOM节点再调用$.zeptoZ方法获取Z对象
*   selector参数为css选择器，调用$.zepto.qsa方法获取对应DOM节点再调用$.zepto.Z方法获取Z对象
*   selector参数为DOM节点数组，去掉数组中值为null的项，调用$.zepto.Z方法获取Z对象
*   selector参数为单个DOM节点，`dom = [selector]`，然后调用$.zepto.Z方法获取Z对象
*   selector参数为Z对象，直接返回该Z对象
*   selector参数为函数，执行`$(document).ready(selector)`，在DOM加载完的时候调用该函数

整体代码：
```javascript
zepto.init = function(selector, context) {
    var dom;
    //未传参，返回空Zepto对象
    if (!selector) {
        console.log("未传参数");
        return zepto.Z();
    } else if (typeof selector == 'string') {
        selector = selector.trim();
        //如果是“<>”,基本的html代码时
        if (selector[0] == '<' && fragmentRE.test(selector)) {
            console.log(selector, RegExp.$1);
            //调用片段生成dom
            dom = zepto.fragment(selector, RegExp.$1, context), selector = null;
        } else if (context !== undefined) {
            return $(context).find(selector);
        } else {
            dom = zepto.qsa(document, selector)
        }
    } //如果selector是个函数
    else if (isFunction(selector)) {
        return $(document).ready(selector);
    } //如果selector是一个Zepto对象，返回它自己
    else if (zepto.isZ(selector)) {
        return selector;
    } else {
        if (isArray(selector)) {
            dom = compact(selector);
        } else if (isObject(selector)) {
            dom = [selector], selector = null;//单个DOM
        } else if (fragmentRE.test(selector)) {
            dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null;
        } else if (context !== undefined) {
            return $(context).find(selector);
        } else {
            dom = zepto.qsa(document, selector);
        }
    }
    return zepto.Z(dom, selector);
}
```
# 1.上下文查找
关键一步是`$(context).find(selector);`，通过find函数查找（`1.5`部分）。

```javascript
<div id="foo1">
    <div>1</div>
    <div>2</div>
</div>
<script src="_zepto.js"></script>
<script type="text/javascript">
console.log($('div', '#foo1'));//foo1下的两个div
</script>
```
## 1.1 工具及变量
```javascript
//清除包含的null undefined
function compact(array) {
    return filter.call(array, function(item) {
        return item != null;
    });
}
function isFunction(value) { return type(value) == "function"； }
$.isFunction = isFunction；
```
## 1.2 filter方法
```javascript
$.fn = {
    //fiter函数其实可以说是包装原生的filter方法
    filter: function(selector) {
        if (isFunction(selector)) {
           //this.not(selector)取到需要排除的集合，
           //第二次再取反(这个时候this.not的参数就是一个集合了)，得到想要的集合
            return this.not(this.not(selector));//见`1.3`部分
        } //下面一句的filter是原生的方法
        //过滤剩下this中有被selector选择的
        return $(filter.call(this, function(element) {
            return zepto.matches(element, selector);//见`1.4`部分
        }));
    }
}
}
```
## 1.3 not、forEach方法

```javascript
$.fn = {
    forEach: emptyArray.forEach,
    not: function(selector) {
        var nodes = [];
        //当selector为函数时，safari下的typeof NodeList也是function，
        //所以这里需要再加一个判断selector.call !== undefined
        if (isFunction(selector) && selector.call !== undefined)
            this.each(function(idx) {
                if (!selector.call(this, idx)) nodes.push(this);
            })
        else {
            var excludes = typeof selector == 'string' ? this.filter(selector) :
                //当selector为nodeList时执行slice.call(selector),
                //注意这里的isFunction(selector.item)是为了排除selector为数组的情况
                (likeArray(selector) && isFunction(selector.item)) ? slice.call(selector) : $(selector);
            this.forEach(function(el) {
                if (excludes.indexOf(el) < 0) nodes.push(el);
            })
        }
        //上面得到的结果是数组，需要转成zepto对象，以便继承其它方法，实现链写
        return $(nodes);
    }
}
```
## 1.4 matches方法

```javascript
     /**
     * [matches 元素element是否在selector中]
     * @param  {[元素]} element  [元素，被查询的元素]
     * @param  {[String]} selector [CSS选择器]
     * @return {[Boolean]}          [/true/false]
     */
 zepto.matches = function(element, selector) {
     if (!selector || !element || element.nodeType !== 1) return false
     var matchesSelector = element.matches || element.webkitMatchesSelector ||
         element.mozMatchesSelector || element.oMatchesSelector ||
         element.matchesSelector;
     //如果当前元素能被指定的css选择器查找到,则返回true,否则返回false.
     //https://developer.mozilla.org/zh-CN/docs/Web/API/Element/matches
     if (matchesSelector) return matchesSelector.call(element, selector);
     //如果浏览器不支持MatchesSelector方法，则将节点放入一个临时div节点
     var match, parent = element.parentNode,
         temp = !parent;
     //当element没有父节点(temp)，那么将其插入到一个临时的div里面
     //目的就是为了使用qsa函数
     if (temp)(parent = tempParent).appendChild(element);
     ///将parent作为上下文，来查找selector的匹配结果，并获取element在结果集的索引
     //不存在时为－1,再通过~-1转成0，存在时返回一个非零的值
     match = ~zepto.qsa(parent, selector).indexOf(element);
     //将插入的节点删掉(&&如果第一个表达式为false,则不再计算第二个表达式)
     temp && tempParent.removeChild(element);
     return match; //true/false即可
 }
```
## 1.5 find方法
```javascript
$.fn = {
    find: function(selector) {
        var result, $this = this;
        if (!selector) {
            result = $();
        } //1-如果selector为node或者zepto集合时
        else if (typeof selector == 'object') {
            //遍历selector，筛选出父级为集合中记录的selector
            result = $(selector).filter(function() {
                var node = this;
                //如果$.contains(parent, node)返回true，则emptyArray.some
                //也会返回true,外层的filter则会收录该条记录
                return emptyArray.some.call($this, function(parent) {
                    return $.contains(parent, node);
                })
            })
        } else if (this.length == 1) { //2-NodeList对象，且length=1
            result = $(zepto.qsa(this[0], selector));
        } else {
            result = this.map(function() { //3-NodeList对象，且length>1
                return zepto.qsa(this, selector);
            });
        }
        return result;
    }
};
```
这部分会使用发dao`map`函数，这里给出代码
`$.fn`中：
```javascript
map: function(fn) {
    return $($.map(this, function(el, i) {
        return fn.call(el, i, el);
    }));
}
```

```javascript
$.map = function(elements, callback) {
    var value, values = [],
        i, key;
    if (likeArray(elements)) {
        for (var i = 0; i < elements.length; i++) {
            value = callback(elements[i], i)
            if (value != null) values.push(value);
        }
    } else {
        for (key in elements) {
            value = callback(elements[key], key);
            if (value != null) values.push(value);
        }
    }
    return flatten(values);
}
```
```javascript
//得到一个数组的副本
function flatten(array) {
    return array.length > 0 ? $.fn.concat.apply([], array) : array;
}
```
# 2.selector是函数（DOMReady机制）
`zepto.init`中

```javascript
if (isFunction(selector)) {
    return $(document).ready(selector);
}
```

```javascript
var readyRE = /complete|loaded|interactive/；
```
`$.fn`函数里面

```javascript
ready: function(callback) {
    if (readyRE.test(document.readyState) && document.body) {
        callback($);
    } else {
        document.addEventListener('DOMContentLoaded', function() { callback($) }, false);
    }
    return this;
}
```
其实这里由于不考虑兼容IE10以下版本，所以写的比较简单。详细的介绍可以看我的另外的一片博客[【domReady机制探究及DOMContentLoaded研究】](http://blog.csdn.net/future_todo/article/details/53120569)。

这里的`document.readyState`顺序是：loading-->interactive(触发监听DOMContentLoaded的函数)【DOM解析完成】-->compelete【资源完成】。

`ready`就是为了保证回掉函数`callback`在`onload`之前执行。

第一篇博客选择符是html片段的时候，还剩下当第三个参数properties带有属性的时候没有处理。
# 3. 创建DOM时候带有属性

用法例如
```javascript
$('<div></div>',{width:'100px'})
```
- 工具函数/变量

```javascript
var methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset']；
```
在`zepto.fragment`中

```javascript
zepto.fragment = function(html, name, properties) {
    var dom, nodes, container;
    if (singleTagRE.test(html)) {
        dom = $(document.createElement(RegExp.$1));
    }
    if (!dom) {
        //修正自闭合标签<input/>转换为<input></input>
        if (html.replace) html = html.replace(tagExpanderRE, "<$1></$2>");
        if (name === undefined) name = fragmentRE.test(html) && RegExp.$1;
        //设置容器名，如果不是tr,tbody,thead,tfoot,td,th，则容器名为div
        if (!(name in containers)) name = "*";
        container = containers[name]; //创建容器
        container.innerHTML = '' + html; //生成DOM
        //取容器的子节点
        dom = $.each(slice.call(container.childNodes), function() {
            container.removeChild(this);
        });
    }
    //TODO 第三个参数properties带有属性
    if (isPlainObject(properties)) {
        nodes = $(dom);
        $.each(properties, function(key, value) {
            // 优先获取属性修正对象，通过修正对象读写值
            // methodAttributes包含'val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'，这些方法都应在`$.fn`中重写。
            if (methodAttributes.indexOf(key) > -1) {
                nodes[key](value);
            } else {
                nodes.attr(key, value)
            }
        });
    }
    return dom;
}
```
其中methodAttributes方法的源码实现，我们以后再具体介绍，到此为止`zepto.init`已经基本介绍结束了，接下来我们将另外写一篇博客看看zepto里面操作DOM的具体方法。

参考阅读：
- [Zepto核心模块源码分析](https://github.com/oadaM92/zepto/blob/master/oadaM92/zepto/README.md)
- [zepto.js 源码解析](http://www.runoob.com/w3cnote/zepto-js-source-analysis.html)