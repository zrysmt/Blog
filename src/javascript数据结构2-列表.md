---
title: javascript数据结构2-列表
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---

## 1. 类型定义 
```
 listSize（属性）         列表的元素个数
 pos（ 属性）             列表的当前位置
 length（ 属性） 	 	  返回列表中元素的个数
 clear（ 方法）  		  清空列表中的所有元素
 toString（ 方法）  	  返回列表的字符串形式
 getElement（ 方法） 	  返回当前位置的元素
 insert（ 方法）  		  在现有元素后插入新元素
 append（ 方法） 		  在列表的末尾添加新元素
 remove（ 方法） 		  从列表中删除元素
 front（ 方法）  		  将列表的当前位置设移动到第一个元素
 end（ 方法）  			  将列表的当前位置移动到最后一个元素
 prev（方法）  			  将当前位置后移一位
 next（ 方法）  	      将当前位置前移一位
 currPos（ 方法）  		  返回列表的当前位置
 moveTo（方法） 	      将当前位置移动到指定位置
```
## 2.实现列表类 

```html5
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
	<title>实现列表类</title>
</head>
<body>

<script type="text/javascript">
	function List() {
   this.listSize = 0;
   this.pos = 0;
   this.dataStore = [];
   //this.clear = clear;
   this.find = find;
   this.toString = toString;
   //this.insert = insert;
   this.append = append;
   this.remove = remove;
   this.front = front;
   //this.end = end;
  // this.prev = prev;
   //this.next = next;
   this.length = length;
   //this.currPos = currPos;
   //this.moveTo = moveTo;
   this.getElement = getElement;
  // this.length = length;
}

function append(element) {
   this.dataStore[this.listSize++] = element;
}

function find(element) {
   for (var i = 0; i < this.dataStore.length; ++i) {
      if (this.dataStore[i] == element) {
         return i;
      }
   }
   return -1;
}

function remove(element) {
   var foundAt = this.find(element);
   if (foundAt > -1) {
      this.dataStore.splice(foundAt,1);
      --this.listSize;
      return true;
   }
   return false;
}

function toString() {
    return this.dataStore;
}

 function front(){
  	// return this.dataStore[0];
  	//或者
  	 this.pos=0;
  }
function getElement(){
	return this.dataStore[this.pos];
}
var names = new List();
names.append("Cynthia");
names.append("Raymond");
names.append("Barbara");
console.log(names.toString());
names.remove("Raymond");
console.log(names.toString());
// console.log(names.front());
names.front();
console.log(names.getElement());
</script>
</body>
</html>
```
## 3.实际例子
从txt文件中读取数据（注意：这种方法只是适合在IE浏览器）
文档内容：

> 1.sam
2.tim
3.jom
4.dim
5.pop
6.hello
7.ming

```html5
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" /> 
	<title>无标题</title>
</head>
<body>
  <script type="text/javascript">
  function List(){
   this.listSize = 0;
   this.pos = 0;
   this.dataStore = [];
   //this.clear = clear;
   this.find = find;
   this.toString = toString;
   //this.insert = insert;
   this.append = append;
   this.remove = remove;
   this.front = front;
   this.end = end;
   //this.prev = prev;
   this.next = next;
   this.length = length;
   this.currPos = currPos;
   //this.moveTo = moveTo;
   this.getElement = getElement;
   this.length = length;
  }
  
  function append(element){
  	this.dataStore[this.listSize++]=element;
  }
  function find(element){
  	for (var i = 0; i < this.dataStore.length; i++) {
  		if(this.dataStore[i]==element){
  			return i;
  		}
  	};
  	return -1;
  }

  function remove(element){
  	var foundAt=this.find(element);
  	if(foundAt > -1){
  		this.dataStore.splice(foundAt,1);
  		--this.listSize;
  		return true;
  	}
  	return false;
  }

  function toString(){
  	return this.dataStore;
  }

  function front(){
  	// return this.dataStore[0];
  	 this.pos=0;
  }
    function end(){
  	// return this.dataStore[0];
  	 this.pos=this.listSize-1;
  }
  function currPos(){
  	return this.pos;
  }
  function next(){
  	if(this.pos<this.listSize-1){
  		++this.pos();
  	}
  }
	function getElement(){
		return this.dataStore[this.pos];
	}
  function createArr(){
  	// var arr=read(file).split("/n");
  	//读取文件
  	var s=[],arr=[];
  	  var fso, f1, ts;
      var ForReading = 1;
      var src="E:\\jsDS\\test.txt";
      fso = new ActiveXObject("Scripting.FileSystemObject");
      ts = fso.OpenTextFile(src,1,true); 
      // document.all.mailbdy.value=ts.ReadAll();
      while (!ts.AtEndOfStream) 
		{ 
			str=ts.Readline(); 
      	    // s=str.split("\n");
      	    s.push(str);
		}  
		/*console.log("==================");
		console.log(s);
		console.log("==================");*/
  	for (var i = 0; i < s.length; i++) {
  		 arr[i]=s[i].trim();
  	};
  	return arr;
  }

  function displayList(list){
  	// for (list.front();list.currPos()<list.length();list.next()) {
  		var lists=[];
  		list.front();
  		while(list.currPos() < list.length){
  			lists.push(list.getElement());
  			list.next();
  		}
         return lists;
  	
  }

	  var movies=createArr();
	  var mlist=new List();
	  for (var i = 0; i < movies.length; i++) {
	   console.log(movies[i]);
	  	mlist.append(movies[i]);
	  };
	  console.log(mlist);
	   //console.log(displayList(mlist));
  </script>
</body>
</html>
```
