---
title: this指针结
tags: 
- FE
- javascript
categories: 前端技术
---
文章只是简单列举了方式和一些会改变this指针的情况     

# 1.探寻之，必昭然若揭   
1. new绑定            this-->新创建的对象   
   ``var bar = new foo()``   
2. call/bind 硬绑定      this-->指定的对象   
	``var bar = foo.call(obj2)``   
3. 隐式绑定       this-->上下文对象    
	``var bar = obj1.foo()``   
4. 默认绑定       this-->全局对象window    

四种情况也是按照优先级排列的    

# 2.实践之，定了然于胸    
## 2.1 回掉函数会改变this指针   
  绑定     
 ```javascript
  dbTools.queryUsrDB2Datas(function(){
      usrResDiv.fyDiv.apply(usrResDiv,arguments);
 	});
 ```
## 2.2 setTimeout/setinterval函数会改变this指针(例子见第三部分)
## 2.3 绑定的例外
- `foo.call(null)` 使用`null`或者`undefined`,忽略传入对象的`this`,实际运用的是默认绑定，这也是这样方法的弊端，this指向`window`。
修改`var DMZ = Object.create(null); foo.apply(DMZ,[2,3]);`

- 间接引用

```javascript
function foo(){
		console.log(this.a);
	}
	var a = 2;
	var o = {a:3,foo:foo};
	var p = {a:4};

	o.foo();//3
	(p.foo = o.foo)(); //2 this-->window
	p.foo();  //4
```
`p.foo = o.foo`返回值是目标函数的引用，因此调用位置是foo(),而不是`p.foo()`,`o.foo()`;
# 3.避免之，需谨小事微
除了第一部分的方法外，还有一些常用的方法。
## 3.1 ES5中我们经常会使用`self = this`，如：

```javascript
function foo(){
	var self = this;
	setTimeout(function(){
		console.log(self.a);
	},100);
}

var obj = {
	a:2;
}
foo.call(obj);//2
```
## 3.2 ES6中的箭头函数(this词法)

```javascript
function foo(){
	setTimeout => {
		console.log(this.a);//this继承来自foo()
	},100);
}

var obj = {
	a:2;
}
foo.call(obj);//2
```