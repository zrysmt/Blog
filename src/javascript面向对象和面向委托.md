---
title: javascript面向对象和面向委托
tags: 
- FE
- javascript
- 设计模式
categories: 前端技术
date: 2016-09-27 00:00:00
---

昨天看了一本书《你不知道的javascript(上)》关于这方面的内容，体会颇深，其中书中讲到的把javascript当作是面向委托的语言比面向对象的解释更加贴切，下面我就简单结合自己的理解，书写阐述一下，也可以作为一种笔记记录。     
### 1. 提取精华——几个重要的方法
#### 1.1 原型链关联
- Bar.prototype = Foo.prototype;
- Bar.prototype = new Foo();
- Bar.prototype = Object.create(Foo.prototype);  
第一种方式，没有创建Bar.prototype的新对象Bar.prototype直接引用了Foo.prototype，修改Bar.prototype会影响Foo.prototype     
第二种方式，创建了一个关联Bar.prototype的新对象，new其实是调用Foo的“构造函数”，有些东西会影响到Bar()的后代。    
第三种方式，Object.create() 方法创建一个拥有指定原型和若干个指定属性的对象。
语法：``Object.create(proto, [ propertiesObject ])``
参数:proto 一个对象，作为新创建对象的原型。    
     propertiesObject 可选。该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符（这些属性描述符的结构与Object.defineProperties()的第二个参数一样）
> [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)  

ES5之前Object.create Poyfill代码：   
```javascript
	if(!Object.create){
		Object.create = function(o){
			function F(){};
			F.prototype = o;
			return new F();  //new的作用参见上述 第二种方式
		}
	}
```
ES5:``Object.setPrototypeOf(Bar.prototype,Foo.prototype)``更加标准可靠
#### 1.2 ES6 class     
内部也是通过原型链实现的，只是一种语法糖。

### 2.针尖麦芒——面向对象(OO) VS 面向委托(对象关联 OLOO)    
- OO：类的继承是复制行为，简单说关系是父子关系    
  OLOO： 只是对象的关联(基于原型/原型链)，简单说关系是兄弟关系，互相关联。

- 代码  
OO风格：  
```javascript
	function Foo(who){
		this.name = who;
	}
	Foo.prototype.identity = function(){
		return "I am "+this.name;
	};

	function Bar(who){
		Foo.call(this,who);
	}
	Bar.prototype = Object.create(Foo.prototype);

	Bar.prototype.speak = function(){
		alert("hello,"+this.identity()+" .");
	};

	var b1 = new Bar('b1');
	var b2 = new Bar('b2');
	b1.speak();
	b2.speak();
```

OLOO风格：
```javascript
	Foo = {
        init: function(who) {
            this.name = who;
        },
        identity: function() {
            return "I am " + this.name;
        }
    };

    Bar = Object.create(Foo);
    Bar.speak = function() {
        alert("hello," + this.identity() + " .");
    };

    var b1 = Object.create(Bar);
    b1.init('b1');
    var b2 = Object.create(Bar);
    b2.init('b2');
    b1.speak();
    b2.speak();
```

### 3.问题探究   
**内省：**我们想看Foo和Bar之间的关系     
OO:对比的是Bar.prototype与Foo的关系，并不是Bar和Foo的关系
```javascript
	console.log(Bar.prototype instanceof Foo);  //true
	console.log(Object.getPrototypeOf(Bar.prototype) === Foo.prototype);//true
	console.log(Foo.prototype.isPrototypeOf(Bar.prototype));//true
```

OLOO:是Bar和Foo的关系
```javascript
	console.log(Object.getPrototypeOf(Bar) === Foo);
	console.log(Foo.isPrototypeOf(Bar));
```