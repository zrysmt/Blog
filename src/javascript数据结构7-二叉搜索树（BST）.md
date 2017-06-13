---
title: javascript数据结构7-二叉搜索树（BST）
tags: 
- javascript
- 数据结构
categories: 数据结构
---
## 二叉树 ：
![这里写图片描述](http://img.blog.csdn.net/20151110101942285)

闲话少说，直接上代码：

```html5
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>BST</title>
</head>
<body>
<script>
  //结点
    function Node(data,left,right){
        this.data=data;
        this.left=left;
        this.right=right;
        this.floor=floor;  //层数
        this.show=show;
    }
    function floor(){
      return this.floor;
    }
    function show(){
        return this.data;
    }
  
    function BST(){
        this.root=null;
        this.insert=insert; //插入数据
        this.inOrder=inOrder; //中序排列，详细见后面解释
        this.preOrder=preOrder; //先序排序
        this.postOrder=postOrder; //后续排序
        this.getMax=getMax; //得到最大值
      	this.getMin=getMin; //得到最小值
      	this.find=find; //查找
      	this.remove=remove; //删除节点
    }
    
    function insert(data){
       var n=new Node(data,null,null);
        if(this.root==null){
            this.root=n;
            this.root.floor=1;
        }else{
            var current=this.root;
            var parent;
                var c=1;    
            while(true){
                parent=current;  
                if(data<current.data){
                        current=current.left;
                        c++;  //计算层数的计数器加1
                    if(current==null){
                        parent.left=n;
                        parent.left.floor=c;
                        break;
                    }
                }else{
                    current=current.right;
                        c++;   //加1
                    if(current==null){
                        parent.right=n;
                        //rHeight++;
                        // console.log("**"+rHeight+"**");
                        parent.right.floor=c;
                        break;
                    }
                }
            }
        }
    }
  //中序遍历
  function inOrder(node){
      if(!(node==null)){
          inOrder(node.left);
          document.write(node.show()+" ");
          document.write("层数"+node.floor+"<br/>");
          inOrder(node.right);
      }
       //console.count("inOrder被执行的次数");
  }
  //先序遍历
   function preOrder(node){
      if(!(node==null)){
          document.write(node.show()+" ");
          preOrder(node.left);
          preOrder(node.right);
      }
  }
  
  //后序遍历
   function postOrder(node){
      if(!(node==null)){
          postOrder(node.left);        
          postOrder(node.right);
            document.write(node.show()+" ");
      }
  }
  
  //查找最大值
  function getMax(){
     	var current=this.root;
    	while(!(current.right==null)){
          	current=current.right;
        }
    	return current.data;
  }
  //查找最小值
   function getMin(){
     	var current=this.root;
    	while(!(current.left==null)){
          	current=current.left;
        }
    	return current.data;
  }
  //带参数---查找最小值
  function getSmallest(node){
  		while(!(node.left==null)){
  			node=node.left;
  		}
  	 return node;
  }
  //查找
  function find(data){
    	var current=this.root;
    	while(current!=null){
          	if(current.data==data){
              document.write("<br/>找到【"+data+"】节点<br/>");
              return current;
            }else if(data<current.data){
               current=current.left;
            }else{
               current=current.right;
            }
         
        }
     document.write("<br/>没有找到【"+data+"】 节点<br/>");
//     	return current;
  }
  
  //删除节点-详细解释见后后面
     function remove(data) {
			root = removeNode(this.root, data);
       		//其实root=没有用处，只是保留了函数执行的地址
	}
	function removeNode(node, data) {
          if (node == null) {
				return null;
			}
		if (data == node.data) {
			// 没有子节点的节点
			if (node.left == null && node.right == null) {
					return null;
			}
			// 没有左子节点的节点
			if (node.left == null) {
					return node.right;
			}
			// 没有右子节点的节点
			if (node.right == null) {
					return node.left;
			}
			// 有两个子节点的节点
			var tempNode = getSmallest(node.right);
			node.data = tempNode.data;
			node.right = removeNode(node.right, tempNode.data);
					return node;
			}
			else if (data < node.data) {
				node.left = removeNode(node.left, data);
               
				return node;
			}
			else {
				node.right = removeNode(node.right, data);
              // console.log(node);
				return node;
			}
}
  
  //测试
      var nums=new BST();
      nums.insert(56);
      nums.insert(22);  
      nums.insert(81);
      nums.insert(10);
      nums.insert(30);
      nums.insert(77);
      nums.insert(92);
      nums.insert(100);
      document.write("*****************中序遍历***************</br>");
      inOrder(nums.root);
      document.write("</br>***************先序遍历***************</br>");
      preOrder(nums.root);
      document.write("</br>***************后序遍历***************</br>");
      postOrder(nums.root);
      //nums.show();
      //console.log(nums);  
      document.write("</br>***************最大值/最小值************</br>");
      document.write(nums.getMax()+"/"+nums.getMin());
      document.write("</br>***************查找************</br>");
  	  nums.find(100);
      document.write("</br>***************删除节点81后************</br>");
      nums.remove(81);
      console.log(nums);
      preOrder(nums.root);
</script>
</body>
</html>
```
结果：
![这里写图片描述](http://img.blog.csdn.net/20151110102901708)
## 遍历 
中序遍历：
![这里写图片描述](http://img.blog.csdn.net/20151110102019745)
理解双层递归

```javascript
function inOrder(node) {
    if (!(node == null)) {
        inOrder(node.left);                             //@1
        document.document(node.show() + " ");           //@2
        inOrder(node.right);                            //@3
    }
}


inOrder(nums.root);    //开始执行
```
从根节点开始：
![这里写图片描述](http://img.blog.csdn.net/20151110102238331)

![这里写图片描述](http://img.blog.csdn.net/20151110102715132)
![这里写图片描述](http://img.blog.csdn.net/20151110102731867)

## 删除节点： 

```javascript
function remove(data) {
		root = removeNode(this.root, data);  
	}
```
简单接受待删除的数据，具体执行是removeNode函数；

```javascript
function removeNode(node, data) {
	}
```
**待删除的节点是：**
**1.叶子结点**，只需要将从父节点只想它的链接指向null；

```javascript
if (node.left == null && node.right == null) {
		return null;   
	}
```

```javascript
if (node.left == null && node.right == null) {
				return null;   //递归，找到节点置为空即可
			}
			   //其他情况
			}
			else if (data < node.data) {
				node.left = removeNode(node.left, data);  //#1            
				return node; //#2
			}
			else {
				node.right = removeNode(node.right, data);//#3
				return node; //#4
			}
```

通过if else逻辑找到node节点


----------
	//#1  node.left=null(后面的函数递归后返回的值)
	//#3  node.right=null(后面的函数递归后返回的值)

----------
**2.只包含一个子节点**，原本指向它的节点指向它的子节点。
**3.左右子树都有的时候**。两种做法：找到左子树的最大值或者右子树的最小值。这里我们用第二种。

 - 查找右子树的最小值，创建一个临时的节点tempNode。
 - 将临时节点的值赋值给待删除节点
 - 删除临时节点

注意：

----------
//#2 //#4必须有，如果没有，则删除节点下面的所有子树都将被删除。
真个过程举个形象的说明，遍历的时候把节点之间的链条解开进行查询，return node；递归查询到最后一级后，由下向上对链条进行缝合。

----------
