---
title: 动手DIY一个underscorejs库及underscorejs源码分析2
tags:
- FE
- underscorejs
- 源码
- js原生实现库
categories: 前端技术
---

接着第一篇《动手DIY一个underscorejs库及underscorejs源码分析1》

> 所有代码挂在我的[github](https://github.com/zrysmt/DIY-underscorejs)上。

# 1.兼容requirejs和seajs模块化
- requirejs
在代码的尾部加上

```javascript
    if (typeof define == 'function' && define.amd) {
        //定义一个模块并且起个名字
        define('_underscore', [], function() {
            return _;
        });
    }
```
使用测试：[代码请点我](https://github.com/zrysmt/DIY-underscorejs/tree/master/demo)
demo3.html

```html
<body>
    <script data-main="demo3" src="lib/require.js"></script>
</body>
```
demo3.js
```javascript
define(function(require) {
    require(['../DIY/2/_underscore'], function() {
        console.log(_);
    });
});
```
- 加上支持seajs的代码

```javascript
    if (typeof define == 'function' && define.amd) {
        define('_underscore', [], function() {
            return _;
        });
    } else if (typeof define == 'function' && define.cmd) { //seajs
        define(function(require, exports, module) {
            module.exports = _;
        });
    }
```
使用：
demo2.html

```javascript
    <script src="lib/sea-debug.js"></script>
    <script>
        seajs.use('./demo2');
    </script>
```
demo2.js

```javascript
define(function(require, exports, module) {
    var _ = require('../DIY/2/_underscore');
    console.info(_);
});
```
# 2.支持nodejs

```javascript
root._ = _;
```
修改为：

```javascript
  if (typeof exports != 'undefined' && !exports.nodeType) {
    if (typeof module != 'undefined' && !module.nodeType && module.exports) {
      exports = module.exports = _;
    }
    exports._ = _;
  } else {
    root._ = _;
  }
```
# 3.`_.extend`

使用：

```javascript
    console.log(_.extend({name: 'moe'}, {age: 50}));
    //结果Object {name: "moe", age: 50}
```
    
```javascript
    //类似与_.keys
    _.allKeys = function(obj) {
        if (!_.isObject(obj)) return [];
        var keys = [];
        for (var key in obj) keys.push(key);
        // Ahem, IE < 9.
        // if (hasEnumBug) collectNonEnumProps(obj, keys);
        return keys;
    };
```
主函数:

```javascript
    var createAssigner = function(keysFunc, defaults) {
        return function(obj) {
            var length = arguments.length;
            if (defaults) obj = Object(obj);
            if (length < 2 || obj == null) return obj;
            for (var index = 1; index < length; index++) {
                var source = arguments[index],
                    keys = keysFunc(source),
                    l = keys.length;
                for (var i = 0; i < l; i++) {
                    var key = keys[i];
                    if (!defaults || obj[key] === void 0) obj[key] = source[key];
            //将参数（对象）放入到obj组合到一起
                }
            }
            return obj;
        };
    };
    _.extend = createAssigner(_.allKeys);
    _.extendOwn = _.assign = createAssigner(_.keys);
```
# 4.重要内部函数`cb`

```javascript
    var builtinIteratee;
    //如果是函数则返回上面说到的回调函数；
    //如果是对象则返回一个能判断对象是否相等的函数；
    //默认返回一个获取对象属性的函数
    var cb = function(value, context, argCount) {
        if (_.iteratee !== builtinIteratee) return _.iteratee(value, context);
        if (value == null) return _.identity; //默认的迭代器
        if (_.isFunction(value)) return optimizeCb(value, context, argCount);
        if (_.isObject(value)) return _.matcher(value);
        return _.property(value);
    };
    _.iteratee = builtinIteratee = function(value, context) {
        return cb(value, context, Infinity);
    };
```
## 4.1 `_.identity`
很简单但是是默认的迭代器
```javascript
    _.identity = function(value) {
        return value;
    };
```
测试很简单
```javascript
    var obj1 = {name:'zry'};
    console.log(obj1 === _.identity(obj1));//true
```
## 4.2 `_.matcher`
```javascript
    _.matcher = _.matches = function(attrs) {
        attrs = _.extendOwn({}, attrs);
        return function(obj) {
            return _.isMatch(obj, attrs);
        };
    };
    //两个对象是不是全等于。给定的对象是否匹配attrs指定键/值属性
    _.isMatch = function(object, attrs) {
        var keys = _.keys(attrs),
            length = keys.length;
        if (object == null) return !length;
        var obj = Object(object);
        for (var i = 0; i < length; i++) {
            var key = keys[i];
            if (attrs[key] !== obj[key] || !(key in obj)) return false;
        }
        return true;

    };
```
测试：

```javascript
    var obj2 = {selected: true, visible: true};
    var ready = _.isMatch(obj2,{selected: true, visible: true});
    //返回一个断言函数，这个函数会给你一个断言 可以用来辨别 
    //给定的对象是否匹配attrs指定键/值属性
    console.log(ready);//true
```
## 4.3 `_.property`
property函数在第一篇博客中已经实现
```javascript
    _.property = property;
```
# 5.`_.map`
```javascript
    _.map = _.collect = function(obj, iteratee, context) {
        iteratee = cb(iteratee, context);
        var keys = !isArrayLike(obj) && _.keys(obj),
            length = (keys || obj).length,
            results = Array(length);
        for (var index = 0; index < length; index++) {
            var currentKey = keys ? keys[index] : index;
            results[index] = iteratee(obj[currentKey], currentKey, obj);
            //返回的是（value，key，obj）
        }
        return results;
    };
```
# 6.`_.filter`

```javascript
    _.filter = _.select = function(obj, predicate, context) {
         var results = [];
        predicate = cb(predicate, context);
        _.each(obj, function(value, index, list) {
            if (predicate(value, index, list)) results.push(value);
        });
        return results;
    };
```
测试：
```javascript
var evens = _.filter([1, 2, 3, 4, 5, 6], function(num){ return num % 2 == 0; });//[2,4,6]
```
# 7.两个常用的工具函数`_.escape`,_.unescape`
## 7.1 `_.escape`
要过滤的字符串
```javascript
    var escapeMap = {
        '&': '&amp;',
        '<': '&lt;',
        '>': '&gt;',
        '"': '&quot;',
        "'": '&#x27;',
        '`': '&#x60;'
    };
```
主函数
```javascript
    var createEscaper = function(map) {
        var escaper = function(match) {//match 匹配的子串
            return map[match];
        };
        var source = '(?:' + _.keys(map).join('|') + ')';
        var testRegexp = RegExp(source);
        var replaceRegexp = RegExp(source, 'g');
        return function(string) {
            string = string == null ? '' : '' + string;
            return testRegexp.test(string) ? string.replace(replaceRegexp, escaper) : string;
        };
    };
```
注意值了的`string.replace`函数第二个参数是个函数，那么返回的数据第一个是match（匹配的子串）

| 变量名 | 代表的值 |
|---------|:------------:|
| match | 匹配的子串。（对应于上述的$&。） |
|p1,p2, ...| 假如replace()方法的第一个参数是一个`RegExp`对象，则代表第n个括号匹配的字符串。（对应于上述的$1，$2等。） |
| offset| 匹配到的子字符串在原字符串中的偏移量。（比如，如果原字符串是“abcd”，匹配到的子字符串时“bc”，那么这个参数将是1） |
| string | 被匹配的原字符串。 |
```javascript
_.escape = createEscaper(escapeMap);
```
测试：
```javascript
console.log(_.escape('Curly, Larry & Moe')//Curly, Larry &amp; Moe
```
## 7.2 `_.unescape`
反转要过滤的字符串
```javascript
    _.invert = function(obj) {
        var result = {};
        var keys = _.keys(obj);
        for (var i = 0, length = keys.length; i < length; i++) {
            result[obj[keys[i]]] = keys[i];
        }
        return result;
    };
    var unescapeMap = _.invert(escapeMap);
    _.unescape = createEscaper(unescapeMap);
```
测试：
```javascript
console.log(_.unescape('Curly, Larry &amp; Moe'));//Curly, Larry & Moe
```


**参考阅读：**
- [http://underscorejs.org/](http://underscorejs.org/)
- [underscorejs中文：http://www.bootcss.com/p/underscore/](http://www.bootcss.com/p/underscore/)
- [UnderscoreJS精巧而强大工具包](http://blog.fens.me/nodejs-underscore/)
- [JS高手进阶之路：underscore源码经典](http://www.imooc.com/article/1566)