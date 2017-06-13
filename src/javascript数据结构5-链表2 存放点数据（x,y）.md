---
title: javascript数据结构5-链表2 存放点数据（x,y）
tags: 
- javascript
- 数据结构
categories: 数据结构
---

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