---
title: javascript的setter getter方法总结
tags: 
- javascript
- es
categories: 前端技术
---

javascript的setter getter方法一共有五种实现方式
- 1.通过对象初始化器定义
- 2.使用 Object.create 方法
- 3.使用 Object.defineProperty 方法
- 4.使用 Object.defineProperties 方法
- 5.使用 Object.prototype.__defineGetter__ 以及 Object.prototype.__defineSetter__ 方法

# 1.通过对象初始化器定义

```javascript
var o = {
    a : 8,
    get b(){return this.a +1;},//通过 get,set的 b,c方法间接性修改 a 属性
    set c(x){this.a = x/2}
};
console.log(o.a);//8
console.log(o.b);//9
o.c = 50;
console.log(o.a);//25
```
我们试着将get set的方法改写成同名，注意属性a与方法a会混淆，将属性a改为_a,结果如下：

```javascript
var o = {
    _a : 8,
    get a(){return this._a +1;},//死循环
    set a(x){this._a = x/2}
};
console.log(o._a);//8
console.log(o.a);//9
o.a = 50;
console.log(o._a);//25
console.log(o.a);//26
```
es6中的新语法：

```javascript
var b = "bb";
var c = "cc";
var o = {
    _a : 8,
    get [b](){return this._a +1;},
    set [c](x){this._a = x/2},
};
console.log(o._a);//8
console.log(o[b]);//9
o["cc"] = 50;//等同于o.c = 50;
console.log(o._a);//25
```
# 2.使用 `Object.create` 方法

```javascript
var o = null;
o = Object.create(Object.prototype,
    {
        bar:{
            get :function(){
                return 10;
            },
            set : function (val) {
                console.log("Setting o.bar",val);
            }
        }
    });
console.log(o.bar);//10
o.bar = 12;//Setting o.bar 12
console.log(o.bar);//10
```
如果这样写：

```javascript
var o = { a: 10 };
o = Object.create(Object.prototype, {
    bar: {
        get: function() {
            return o.a;//或者this.a结果一样
        },
        set: function(val) {
            this.a = val;
        }
    }
});
console.log(o.bar); //undefined
o.bar = 12; 
console.log(o.bar); //12
```
# 3.使用 `Object.defineProperty` 方法

```javascript
var o = { a: 10 } //声明一个对象,包含一个 a 属性,值为1
Object.defineProperty(o, "b", {
    get: function() {
        return this.a;
    },
    set: function(val) {
        this.a = val;
    },
    configurable: true
});

console.log(o.b);//10
o.b = 2;
console.log(o.b);//2
```
# 4.使用 `Object.defineProperties` 方法

```javascript
var obj = { a: 1, b: "sss" };
Object.defineProperties(obj, {
    "A": {
        get: function() {
            return this.a + 1;
        },
        set: function(val) { this.a = val; }
    },
    "B": {
        get: function() {
            return this.b + 2;
        },
        set: function(val) { this.b = val }
    }
});

console.log(obj.A);//2
console.log(obj.B);//sss2
obj.A = 3;
obj.B = "hello";
console.log(obj.A);//4
console.log(obj.B);//hello2
```
# 5.使用`Object.prototype.__defineGetter__` 以及 `Object.prototype.__defineSetter__` 方法
这两种方法是非标准，最好不要在开发中使用

```javascript
var o = { _a: 1 };
o.__defineGetter__("a", function() {
    return this._a;
});
o.__defineSetter__("a", function(val) {
    this._a = val;
})
console.log(o.a);//1
o.a = 2;
console.log(o.a);//2
```