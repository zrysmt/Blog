---
title: javascript数据结构1-数组
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---

书籍：

> 数据结构与算法javascript描述

数组比较简单，这里只是简单介绍：
## 1.使用数组
### 1.1 创建数组
```javascript
//第一种形式
var numbers = new Array(3);
//第二种形式
var numbers = [7,4,1776];

```

大多数JavaScript 专家推荐使用[]操作符，和使用Array 的构造函数相比，这种方式被认为效率更高（new创建的对象，会一直存在于内存中）

### 1.2 读写数组

```javascript
var numbers = [1,2,3,5,8,13,21];
var sum = 0;
for (var i = 0; i < numbers.length; ++i) {
sum += numbers[i];
}
```
### 1.3 字符串生成数组

```javascript
//下面的这一小段程序演示了split() 方法的工作原理：
var sentence = "the quick brown fox jumped over the lazy dog";
var words = sentence.split(" ");
for (var i = 0; i < words.length; ++i) {
	console.log("word " + i + ": " + words[i]);
}
```
### 1.4 对数组的整体性操作 

```javascript
var nums = [];
for (var i = 0; i < 100; ++i) {
nums[i] = i+1;
}
var samenums = nums;
nums[0] = 400;
console.log(samenums[0]); // 显示400
```
这种行为被称为**浅复制**，新数组依然指向原来的数组。一个更好的方案是使用**深复制**，将
原数组中的每一个元素都复制一份到新数组中。可以写一个深复制函数来做这件事：

```javascript
function copy(arr1, arr2) {
for (var i = 0; i < arr1.length; ++i) {
arr2[i] = arr1[i];
}
}
//这样，下述代码片段的输出就和我们希望的一样了：
var nums = [];
for (var i = 0; i < 100; ++i) {
nums[i] = i+1;
}
var samenums = [];
copy(nums, samenums);
nums[0] = 400;
console.log(samenums[0]); // 显示 1
```
## 2. 存取函数 
### 2.1 查找元素 

```javascript
var names = ["David", "Cynthia", "Raymond", "Clayton", "Jennifer"];
putstr("Enter a name to search for: ");
var name = readline();
var position = names.indexOf(name);
if (position >= 0) {
	console.log("Found " + name + " at position " + position);
}
else {
	console.log(name + " not found in array.");
}
```
### 2.2 两个函数使用 
concat  连接
splice 截取
join() 和toString()  将数组转化为字符串

```javascript
var cisDept = ["Mike", "Clayton", "Terrill", "Danny", "Jennifer"];
var dmpDept = ["Raymond", "Cynthia", "Bryan"];
var itDiv = cis.concat(dmp);
console.log(itDiv);
itDiv = dmp.concat(cisDept);
console.log(itDiv);
//输出为：
Mike,Clayton,Terrill,Danny,Jennifer,Raymond,Cynthia,Bryan
Raymond,Cynthia,Bryan,Mike,Clayton,Terrill,Danny,Jennifer
```
## 3. 可变函数 
**简单函数：**
```
 push()          末尾增加元素
 unshift()       在开头添加元素
 pop()           在末尾删除元素
 shift()         在开头删除元素
```
**从数组中间删除元素：**

```javascript
var nums = [1,2,3,7,8,9];
var newElements = [4,5,6];
nums.splice(3,0,newElements);
console.log(nums); // 1,2,3,4,5,6,7,8,9
```
**排序函数：**

```javascript
var nums = [1,2,3,4,5];
nums.reverse();
console.log(nums); // 5,4,3,2,1
```

```javascript
var names = ["David","Mike","Cynthia","Clayton","Bryan","Raymond"];
names.sort();
console.log(names); // Bryan,Clayton,Cynthia,David,Mike,Raymond
```
自定义：

```javascript
function compare(num1, num2) {
return num1 - num2;
}
var nums = [3,1,2,100,4,200];
nums.sort(compare);
console.log(nums); // 1,2,3,4,100,200
//sort() 函数使用了compare() 函数对数组按照数字大小进行排序，而不是按照字典顺序。
```
## 4.迭代器 
```
     函数      说明                           是否生成新数组
  foreach()    全部遍历                           否    
  every()      全部返回true，才返回true            否    
  some()       只要一个返回true，就返回true         否    
  reduce()     不断调用累加值                      否    
  map()        符合条件的，类比foreach()           是    
  filter()     返回结果为true的函数                是    
```
## 5.二维数组和多维数组 

```javascript   
Array.matrix = function(numrows, numcols, initial) {
var arr = [];
for (var i = 0; i < numrows; ++i) {
	var columns = [];
	for (var j = 0; j < numcols; ++j) {
	columns[j] = initial;
	}
arr[i] = columns;
}
return
```

## 6.两种特殊的数组 
数组的函数同样适用

**对象数组**
```javascript
function Point(x,y) {
	this.x = x;
	this.y = y;
}
```
**对象中的数组**

```javascript
function weekTemps() {
this.dataStore = [];
this.add = add;
this.average = average;
}
function add(temp) {
this.dataStore.push(temp);
}
function average() {
var total = 0;
for (var i = 0; i < this.dataStore.length; ++i) {
total += this.dataStore[i];
}
return total / this.dataStore.length;
}
	var thisWeek = new weekTemps();
	thisWeek.add(52);
	thisWeek.add(55);
	thisWeek.add(61);
	thisWeek.add(65);
	thisWeek.add(55);
	thisWeek.add(50);
	thisWeek.add(52);
	thisWeek.add(49);
console.log(thisWeek.average()); // 显示54.875

```