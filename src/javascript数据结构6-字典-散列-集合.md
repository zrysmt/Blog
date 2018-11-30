---
title: javascript数据结构6-字典 散列 集合
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---

## 6.1 字典  
字典是一种以键- 值对形式存储数据的数据结构，就像电话号码簿里的名字和电话号码一

```html5 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>字典sample</title>
</head>
<body>
<script>
	function Dictionary(){
	   this.add = add;
	   this.datastore = new Array();
	   this.find = find;
	   this.remove = remove;
	   this.showAll = showAll;
	   this.count = count;
	   this.clear = clear;
	}

	function add(key, value) {
	   this.datastore[key] = value;
	}

	function find(key) {
	   return this.datastore[key];
	}

	function remove(key) {
	   delete this.datastore[key];
	}

	function showAll() {
	if(this.datastore!=null){
	   var datakeys=Array.prototype.slice.call(Object.keys(this.datastore));
	   for (var key in datakeys) {
	      document.write(datakeys[key] + " -> " + this.datastore[datakeys[key]]+" ");
	      // console.log(Object.keys(this.datastore));
	      console.log(key);
	   }
	 }else{
	 	document.write("字典为空");
	 }
	}

	function count() {
	   var n = 0;
	   for  (var key in Object.keys(this.datastore)) {
	      ++n;
	   }
	   return n;
	}

	function clear() {
	   // for  (var key in Object.keys(this.datastore)) {
	   //    delete this.datastore[key];
	   // } 
	   delete this.datastore;
	}

	//测试
	var dic=new Dictionary();
	dic.add("123","R");
	dic.add("456","Python");
	dic.add("789","JavaScipt");
	document.write("</br>**************字典数目**************</br>");
	var n=dic.count();
	document.write(n);
	document.write("</br>**************全部显示**************</br>");
	dic.showAll();
	document.write("</br>**************删除123--->R*************</br>");
	dic.remove("123");
	dic.showAll();
	document.write("</br>**************清除**************</br>");
	dic.clear();
	dic.showAll();
</script>
</body>
</html>
```

## 6.2 散列（HashTable） 

它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度
使用：MD5 和 SHA-1 可以说是目前应用最广泛的Hash算法
    java中已经实现
![这里写图片描述](http://img.blog.csdn.net/20151110094955099)

```html5
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>HashTable散列表</title>
</head>
<body>
<script>
	function HashTable() {
	   this.table = new Array(137); //为了避免碰撞，首先要确保散列表中用来存储数据的数组其大小是个质数。这一点很关
键，这和计算散列值时使用的取余运算有关。
	   this.simpleHash = simpleHash;   //简单的散列表
	   this.betterHash = betterHash;   //更好的HashTable，避免碰撞
	   this.showDistro = showDistro;
	   this.put = put;
	   //this.get = get;
	}
	function put(data) {
	   var pos = this.simpleHash(data);
	   this.table[pos] = data;
	}
	   
	function simpleHash(data) {
	   var total = 0;
	   for (var i = 0; i < data.length; ++i) {
	      total += data.charCodeAt(i);
	   }
	   document.write("Hash value: " + data + " -> " + total+"<br/>");
	   return total % this.table.length;
	}
	function showDistro() {
	   var n = 0;
	   for (var i = 0; i < this.table.length; ++i) {
	      if (this.table[i] != undefined) {
	         document.write(i + ": " + this.table[i]+"<br/>");
	      }
	   }
	}
	function betterHash(string) {
	   const H = 31;  //较小的质数  书上37不行 
	   var total = 0;
	   for (var i = 0; i < string.length; ++i) {
	      total += H * total + string.charCodeAt(i);
	   }
	   total = total % this.table.length;
	   if (total < 0) {
	      total += this.table.length-1;
	   }
	   return parseInt(total);
	}
	var someNames = ["David", "Jennifer", "Donnie", "Raymond",
                 "Cynthia", "Mike", "Clayton", "Danny", "Jonathan"];
	var hTable = new HashTable();
	
	for (var i = 0; i < someNames.length; ++i) {
  		 hTable.put(someNames[i]);
	}
	hTable.showDistro();
</script>
</body>
</html>
```
![这里写图片描述](http://img.blog.csdn.net/20151110095013990)
这就是碰撞，为避免碰撞，使用betterHash
修改：
```javascript
function put(data) {
	   // var pos = this.simpleHash(data);
	   var pos = this.betterHash(data);
	   this.table[pos] = data;
	}
```
 
