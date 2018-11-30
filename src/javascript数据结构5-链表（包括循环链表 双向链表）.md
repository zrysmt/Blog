---
title: javascript数据结构5-链表（包括循环链表 双向链表）
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---

## 1.一般链表 
图解链表：
![这里写图片描述](http://img.blog.csdn.net/20151020163459524)
链表
![这里写图片描述](http://img.blog.csdn.net/20151020163616943)
![这里写图片描述](http://img.blog.csdn.net/20151020163716990)

实现：

```html5
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" >
  </head>
  <body>
    <script>
    function Node(ele) {
        this.ele=ele;
        this.next=null;
      }
      
    function LList(){
    	   this.head=new Node("head");
     	   this.find=find;
      	   this.insert=insert;
           this.findPrevious=findPrevious;
           this.remove=remove;
           this.display=display;
       // this.Node=Node;
      }
      
      function find(item){
        	var currNode=this.head;
     		 // document.write(currNode);
      		 // console.log(currNode);
            while(currNode.ele!=item)
      		  {currNode=currNode.next;}
      		  return currNode;
      }
      function insert(newElement,item)
      {
      var newNode=new Node(newElement);
      var current=this.find(item);
       newNode.next=current.next;
       current.next=newNode;
      }
      function display(){
      var currNode=this.head;
        while(!(currNode.next==null))
        {document.write(currNode.next.ele+" ");
         currNode=currNode.next;
        }
      }
      
      function findPrevious(item){
        var currNode=this.head;
        while(!(currNode.next==null)&&(currNode.next.ele != item)){
            currNode=currNode.next;
        }
           return currNode;
      }
      function remove(item){
         var prevNode=this.findPrevious(item);
       // document.write(prevNode.ele);
        if(!(prevNode.next==null)){
           prevNode.next=prevNode.next.next;
        }
      }
     var cities=new LList();
      document.write("=========插入数据==========<br/>");
      cities.insert("Con","head");
      cities.insert("Rus","Con");
      cities.insert("Alm","Rus");
      cities.insert("Tom","Alm");
      cities.display();
      document.write("<br/>=========删除数据==========<br/>");
      cities.remove("Rus");
      cities.display();
      
    </script>
  </body>
</html>
```
对比：find()  findPrevious()
while语句多了条件**!(currNode.next==null)**
这个能保证remove()调用时候，删除链表中没有的节点，会返回最后一个节点，这样remove()执行没有任何结果，而链表能够正常的显示。find()中不能使用，原理一样，想一想为什么？
##  2.双向链表 
![这里写图片描述](http://img.blog.csdn.net/20151020164155783)

```html5
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>无标题</title>
</head>
<body>
	<script type="text/javascript">
	function Node(element) {
			this.element = element;
			this.next = null;
			this.previous = null;
	}
	function LList() {
		this.head = new Node("head");
		this.find = find;
		this.insert = insert;
		this.display = display;
		this.remove = remove;
		this.findLast = findLast;
		this.dispReverse = dispReverse;
	}
	function dispReverse() {
		var currNode = this.head;
		currNode = this.findLast();
		while (!(currNode.previous == null)) {
			document.write(currNode.element+" ");
			currNode = currNode.previous;
		}
	}
	function findLast() {
		var currNode = this.head;
		while (!(currNode.next == null)) {
			currNode = currNode.next;
		}
		return currNode;
	}
	function remove(item) {
		var currNode = this.find(item);
		if (!(currNode.next == null)) {
			currNode.previous.next = currNode.next;
			currNode.next.previous = currNode.previous;
			currNode.next = null;
			currNode.previous = null;
		}
	}
	//findPrevious 没用了，注释掉
	/*function findPrevious(item) {
	var currNode = this.head;
	while (!(currNode.next == null) &&
	(currNode.next.element != item)) {
	currNode = currNode.next;
	}
	return currNode;
	}*/
	function display() {
		var currNode = this.head;
		while (!(currNode.next == null)) {
		document.write(currNode.next.element+" ");
		currNode = currNode.next;
		}
	}
	function find(item) {
		var currNode = this.head;
		while (currNode.element != item) {
		currNode = currNode.next;
		}
		return currNode;
	}
	function insert(newElement, item) {
		var newNode = new Node(newElement);
		var current = this.find(item);
		newNode.next = current.next;  //1
		newNode.previous = current;   //2
		current.next = newNode;       //3
	}
	var cities = new LList();
	cities.insert("Conway", "head");
	cities.insert("Russellville", "Conway");
	cities.insert("Carlisle", "Russellville");
	cities.insert("Alma", "Carlisle");
	//cities.insert("C", "Russellville"); //按照原程序写的话，出现很大问题
	cities.display();
	document.write("<br/>");
	cities.remove("Carlisle");
	cities.display();
	document.write("<br/>");
	cities.dispReverse();
</script>
</body>
</html>
```
这是可以完全运行，但是加上黄色的一句话（从中间随便插入一句）就会出现问题

```
Conway Russellville C Carlisle Alma 
Conway Russellville Alma   //C不显示了
Alma Russellville Conway
```
![这里写图片描述](http://img.blog.csdn.net/20151020164454697)
看链表结构：
1,2,3条线都有，但是从中间插入的时候，会发现缺少4是不行的，于是insert()函数加上这句：

```
if(newNode.next!=null){
	newNode.next.previous=newNode;
}
```
就行了。同样书中的删除节点图解，也是知识考虑了在尾部删除

![这里写图片描述](http://img.blog.csdn.net/20151020163955246)

存储一个对象的时候：点对象

```html5
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Document</title>
</head>
<body>
<script>
  
function Node(element){
  this.element=element;
  this.next=null;
}
  
  function Point(x,y){
   this.x=x;
    this.y=y;
  }
  function LList(){
    this.head=new Node('head');
    //this.head.next=this.head;
    this.find=find;
    this.insert=insert;
    this.display=display;
    this.remove=remove;
    this.findPrevious=findPrevious;
  }
  
  function display(){
    var curr=this.head;
    while(!(curr.next==null)){
       document.write(curr.next.element.x+'/'+curr.next.element.y);
      curr=curr.next;
    }
    return curr;
  }
  
    function find(item){
      var currNode=this.head;
      while(!(currNode.element==item)){currNode=currNode.next;
  }
  return currNode;
}
  
  function insert(newElement, item) {
   var newNode = new Node(newElement);
   var current = this.find(item);
   newNode.next = current.next;
   current.next = newNode;
}
  
  function  findPrevious(item){
    var currNode=this.head;
    while(!(currNode.next==null)&&(currNode.next.element!=item)){
      currNode=currNode.next;
    }
    return currNode;
  }
  
   function remove(item){
     var prevNode=this.findPrevious(item);
     if((prevNode.next!=null)){
       prevNode.next=prevNode.next.next;
     }
   }
   var p1=new Point(1,2);
   var p2=new Point(3,4);
   
   //document.write(p2.x);
  // console.log(p1);
   var points=new LList();
    points.insert(p1,'head');
    points.insert(p2,p1);
    points.display();
</script>
</body>
</html>
```
##  3.循环链表 ##
![这里写图片描述](http://img.blog.csdn.net/20151020164724099)

```html5
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>循环链表</title>
</head>
<body>
	<script type="text/javascript">
	function Node(element) {
			this.element = element;
			this.next = null;
	}
	function LList() {
		this.head = new Node("head");
		this.head.next=this.head;
		this.find = find;
		this.insert = insert;
		this.display = display;
		this.remove = remove;
	}
	function remove(item) {
		var currNode = this.find(item);
		if (!(currNode.next == null)) {
			currNode.previous.next = currNode.next;
			currNode.next.previous = currNode.previous;
			currNode.next = null;
			currNode.previous = null;
		}
	}
	
	function display() {
		var currNode = this.head;
		while (!(currNode.next.element=="head")&&!(currNode.next == null)) {
			document.write(currNode.next.element+" ");
			currNode = currNode.next;
		}
	}
	function find(item) {
		var currNode = this.head;
		while (!(currNode.next.element=="head")&&(currNode.element != item)) {
			currNode = currNode.next;
		}
		return currNode;
	}
	function insert(newElement, item) {
		var newNode = new Node(newElement);
		var currNode = this.find(item);
		if(!(currNode.next.element=="head")){
			newNode.next=currNode.next;  //从中间插入
			currNode.next=newNode;
		}else{ //从尾部插入
			newNode.next=this.head;  //从中间插入
			currNode.next=newNode;
		}
	}
	var cities = new LList();
	cities.insert("Conway", "head");
	cities.insert("Russellville", "Conway");
	cities.insert("Carlisle", "Russellville");
	cities.insert("Alma", "Carlisle");
	cities.insert("C", "Russellville"); 
	//cities.insert("D", "Rus"); 
	cities.display();
	
</script>
</body>
</html>
```