---
title: javascript数据结构3-栈
tags: 
- javascript
- 数据结构
categories: 数据结构
---

后进先出（LIFO,last-in-first-out）的数据结构
类比：堆叠盘子，只能从上面拿走盘子

```html5
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
	<title>栈</title>
</head>
<body>
   <script type="text/javascript">
		function Stack() {
		   this.dataStore = [];
		   this.pos = 0;
		   this.push=push;
		   this.pop=pop;
		   this.peek=peek;
		   this.clear = clear;
		   this.length=length;
		}
        
        function push(element){
        	this.dataStore[this.pos++]=element;
        }
        function peek(){
        	return this.dataStore[this.top-1];
        }
        function pop(){
        	return this.dataStore[--this.top];
        }
        function clear(){
        	this.top=0;
        }
        function length(){
        	return this.top;
        }
       /************************************************************************/
       	var s=new Stack();
       	s.push("Tom");
       	s.push("Som");
       	s.push("Dom");
       	s.push("Fom");
       	// document.write(s.dataStore);
       	console.log(s);
   </script>
</body>
</html>
```
例子：
十进制转化为二进制，使用栈实现：

```javascript
/*数制间的相互转换*/
        function mulBase(num,base){
        	var s=new Stack();
        	do{
        		s.push(num% base);
        		num=Math.floor(num /=base);
        	}while(num > 0);
        	var cov="";
          console.log(s.length());
        	while(s.length() >0){
        		cov += s.pop();
        		
        	}
        	return cov;
        }
       var num=32;
       var newNum=mulBase(32,2);  //十进制转换为二进制
       console.log(newNum);
       document.write(newNum);
```