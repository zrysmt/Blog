---
title: this指针结论部分
tags: 
- FE
- javascript
categories: 前端技术
---
文章只是简单列举了方式和一些会改变this指针的情况     

### 探寻之，必昭然若揭   
1. new            this-->新创建的对象   
   ``var bar = new foo()``   
2. call/bind      this-->指定的对象   
	``var bar = foo.call(obj2)``   
3. 隐式绑定       this-->上下文对象    
	``var bar = obj1.foo()``   
4. 默认绑定       this-->全局对象window    

四种情况也是按照优先级排列的    

### 实践之，定了然于胸    
- 回掉函数会改变this指针   
  绑定     
 ```javascript
  dbTools.queryUsrDB2Datas(function(){
      usrResDiv.fyDiv.apply(usrResDiv,arguments);
 	});
 ```
- setTimeout/setinterval函数   