---
title: 一步一步DIY zepto库，研究zepto源码1
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---

我在之前写了《一步一步DIY jQuery库》系列文章，然后发现再往下进行研究jQuery库的时候，由于jQuery库做了很多兼容IE6-8的内容，使其看起来比较繁琐，这也造成了jQuery源码的不宜读性。所幸作为移动端的jQuery库替代品-[Zepto](http://zeptojs.com/),是一个轻量级的**针对现代高级浏览器的JavaScript库， **它与jquery有着类似的api。**Zepto**的设计目的是**提供 jQuery 的类似的API**，但并不是100%覆盖 jQuery。

接下来我们会用一系列博客一边研究Zepto源码，一边DIY一个Zepto库。

基于Zepto 1.2.0版本。

> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，第一篇博客对应文件夹v0.1。
> https://github.com/zrysmt/DIY-zepto

# 1.下载源码并且编译
在github中：[https://github.com/madrobby/zepto](https://github.com/madrobby/zepto)clone下源码，使用下面的命令在命令行编译：
```bash
npm install
npm run dist
```
生成源码文件
```
`dist/zepto.js`
`dist/zepto.min.js`
```
# 2.整体结构

```javascript
var Zepto = (function() {
})();
window.Zepto = Zepto;
window.$ === undefined && (window.$ = Zepto);
```
**Zepto**没有提供`noConflict`命名冲突处理机制，`$`被占用后，就只能用`Zepto`。

# 3.无new化处理结构
使用
```javascript
console.log($('<p></p>'));//生成dom（<p></p>）
```
```javascript
$ = function(selector, context) {
   return zepto.init(selector, context)
}
```
实际上是调用`zepto.init`
```javascript
zepto.init = function(selector, context) {
    //... ...
    return zepto.Z(dom, selector);
}
```
调用`zepto.Z`
```javascript
function Z(dom, selector) {
    var i, len = dom ? dom.length : 0;
    for (i = 0; i < len; i++) {
        this[i] = dom[i];
        this.length = len;//NodeList对象一定要有length属性
        this.selector = selector || '';//选择符
    }
}
zepto.Z = function(dom, selector) {
    return new Z(dom, selector);//在这里使用new实例化
}
```
# 4.传参 形如`$('<p></p>')`
## 4.1 定义要使用的变量
```javascript
var emptyArray = [],
    concat = emptyArray.concat,
    filter = emptyArray.filter,
    slice = emptyArray.slice,
    fragmentRE = /^\s*<(\w+|!)[^>]*>/,
    singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/,
    tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig,
    rootNodeRE = /^(?:body|html)$/i,
    table = document.createElement('table'),
    tableRow = document.createElement('tr'),
    containers = {
        'tr': document.createElement('tbody'),
        'tbody': table,
        'thead': table,
        'tfoot': table,
        'td': tableRow,
        'th': tableRow,
        '*': document.createElement('div')
    },
    class2type = {},
    toString = class2type.toString,
    zepto = {};
```
## 4.2 要使用的工具函数
- 判断类型模块

这些函数都比较好理解
```javascript
function type(obj) {
    return obj == null ? String(obj) :
        class2type[toString.call(obj)] || "object"；
}
function isWindow(obj) {
    return obj != null && obj == obj.window
}
function isObject(obj) {
    return type(obj) == "object";
}
isArray = Array.isArray ||
      function(object){ return object instanceof Array }
function likeArray(obj) {
    var length = !!obj && 'length' in obj && obj.length,
        type = $.type(obj)

    return 'function' != type && !isWindow(obj) && (
        'array' == type || length === 0 ||
        (typeof length == 'number' && length > 0 && (length - 1) in obj)
    )
}
//加上下面这些就可以使用类型判断了
$.each("Boolean Number String Function Array Date RegExp Object Error".split(" "), function(i, name) {
    class2type[ "[object " + name + "]" ] = name.toLowerCase();
});
```
- 在`$`后面定义

```javascript
$.trim = function(str) {
    return str == null ? "" : String.prototype.trim.call(str);
}
$.each = function(elements, callback) {
    var i, key;
    if (likeArray(elements)) {
        for (i = 0; i < elements.length; i++)
            if (callback.call(elements[i], i, elements[i]) === false) return elements;
    } else {
        for (key in elements)
            if (callback.call(elements[key], key, elements[key]) === false) return elements;
    }

    return elements
}
$.type = type;
$.isArray = isArray；
```
## 4.4 `zepto.init`
```javascript
zepto.init = function(selector, context) {
    var dom;
    //未传参，返回空Zepto对象
    if (!selector) {
        return zepto.Z();
    } else if (typeof selector == 'string') {
        selector = selector.trim();
        //如果是“<>”,基本的html代码时
        if (selector[0] == '<' && fragmentRE.test(selector)) {
            //调用片段生成dom
            dom = zepto.fragment(selector, RegExp.$1, context), selector = null;//@1
        }
        //TODO:带有上下文和css查询
    } //如果selector是一个Zepto对象，返回它自己
    else if (zepto.isZ(selector)) {
        return selector;
    } else {
        if (isObject(selector)) {
            dom = [selector], selector = null;//@3
        }
    }
    return zepto.Z(dom, selector);
}
```
## 4.5 `zepto.fragment`
```javascript
 /**
  * [fragment 内部函数 HTML 转换成 DOM]
  * @param  {[String]} html       [html片段]
  * @param  {[String]} name       [容器标签名]
  * @param  {[Object]} properties [附加的属性对象]
  * @return {[*]}           
  */
 zepto.fragment = function(html, name, properties) {
     var dom, nodes, container;
     if (singleTagRE.test(html)) {
         dom = $(document.createElement(RegExp.$1));//@2
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
         //TODO 第三个参数properties带有属性
     }
     return dom;
 }
```
整体流程：
```javascript
$('<p></p>')
```
`zepto.init`--> @1 -->`zepto.fragment`--> @2 --> 函数 $ --> `zepto.init`
--> @3 --> `return zepto.Z(dom, selector)` --> 函数 Z 返回结果。
# 5.`$.fn`扩展
```javascript
$.fn = {
    log: function(test) {
        return '测试';
    }
};
```
使用时候：
```javascript
console.log($.fn.log());//测试
console.log($('div').log());//undefined
```
`$('div')`返回的是Z对象（isZ返回true）。
注意，我们在外部只暴露了`Zepto`,`zepto`，`Z`，`zepto.Z`是内部的变量。
我们缺少一步，将`Z.prototype`指向`$.fn`
```javascript
zepto.Z.prototype = Z.prototype = $.fn;
```
这个时候我们的例子都能正确使用了。
# 6.链式调用
其实原理很简单，只要`return this;`即可。
在`$.fn`中需要
```javascript
$.fn = {
   constructor: zepto.Z,
   length: 0,//为了链式调用能够return this;
};
```
`constructor`和`length`都是为了指定this的constructor,增加默认length属性。而且我们在设置`zepto.Z.prototype = Z.prototype = $.fn;`等于重写了`zepto.Z`和`Z`的原型链，需要使用`constructor: zepto.Z`重新使原型链连接上。
# 7.`$.extend`扩展
- 工具

```javascript
function isPlainObject(obj) {
     return isObject(obj) && !isWindow(obj) && Object.getPrototypeOf(obj) == Object.prototype;
}
```
- `$.extend`

```javascript
$.extend = function(target) {
    var deep, args = slice.call(arguments, 1);
    if (typeof target == 'boolean') {
        deep = target;
        target = args.shift();
    }
    args.forEach(function(arg) { extend(target, arg, deep); });
    return target
}
```
- extend函数

```javascript
function extend(target, source, deep) {
    for (key in source)
    //  deep=true深拷贝 source[key]是数组，一层一层剥开
        if (deep && (isPlainObject(source[key]) || isArray(source[key]))) {
        if (isPlainObject(source[key]) && !isPlainObject(target[key]))
            target[key] = {}; //target[key]不是对象的时候，返回空
        // source[key]是数组，target[key]不是数组
        if (isArray(source[key]) && !isArray(target[key]))
            target[key] = [];
        extend(target[key], source[key], deep); //递归
    } else if (source[key] !== undefined) { //递归结束，source[key]不是数组
        target[key] = source[key];
    }
}
```
例子
```javascript
var target = {
        one: 'patridge',
        three: ["apple", "patato"],
        five: { w: "10", a: "20" }
    },
    source2 = {
        three: ["apple1", "patato1", "abc"],
        four: "orange",
        five: { w: "100", h: "200" }
    };
// console.log($.extend(target, source2));//差别：five:{w:"100",h:"200"}
console.log($.extend(true,target, source2));//差别：five:{a:"20" h:"200" w:"100"}
```
# 8.CSS选择器查询
完成`zepto.init`中TODO:带有上下文和css查询。毕竟这些才是我们经常使用的。
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
            //TODO:带有上下文和css查询
    /********增加代码*****************************************/
        } else if (context !== undefined) {
            return $(context).find(selector);
        } else {
            dom = zepto.qsa(document, selector)
        }
    /******************************************************/
    } //如果selector是一个Zepto对象，返回它自己
    else if (zepto.isZ(selector)) {
        return selector;
    } else {
        if (isObject(selector)) {
            dom = [selector], selector = null;
        }
    }
    return zepto.Z(dom, selector);
}
```
- qsa函数定义

- 需要的工具和变量

```javascript
simpleSelectorRE = /^[\w-]*$/；//全局变量
//匹配包括下划线的任何单词字符或者 - 
```
- qsa函数

```javascript
/**
 * [qsa CSS选择器]
 * @param  {[ELEMENT_NODE]} element  [上下文，常用document]
 * @param  {[String]} selector [选择器]
 * @return {[NodeList ]}   [查询结果]
 */
zepto.qsa = function(element, selector) {
    var found,
        maybeID = selector[0] == '#',
        maybeClass = !maybeID && selector[0] == '.',
        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // Ensure that a 1 char tag name still gets checked
        isSimple = simpleSelectorRE.test(nameOnly); //匹配包括下划线的任何单词字符或者 - 
    return (element.getElementById && isSimple && maybeID) ? //Safari DocumentFragment 没有 getElementById
        //根据id号去查，有返回[found],无返回[]
        ((found = element.getElementById(nameOnly)) ? [found] : []) :
        //不是元素(ELEMENT_NODE)，DOCUMENT_NODE,DOCUMENT_FRAGMENT_NODE,返回空[]
        (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :
        //是上述类型，转化为数组
        slice.call(
            //DocumentFragment 没有getElementsByClassName/TagName
            isSimple && !maybeID && element.getElementsByClassName ?
            maybeClass ? element.getElementsByClassName(nameOnly) : //通过类名获得
            element.getElementsByTagName(selector) : //通过tag标签名获得
            element.querySelectorAll(selector) //不支持getElementsByClassName/TagName的
        );
};
```


这篇博文已经把Zepto的基本架构构建出来了，当然这远远不够，甚至`zepto.init`都没完全实现，下一篇博文将首先完全实现`zepto.init`

> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，第一篇博客对应文件夹v0.1。

参考阅读：
- [Zepto.js API中文版](http://www.runoob.com/manual/zeptojs.html#)
- [https://github.com/madrobby/zepto](https://github.com/madrobby/zepto)