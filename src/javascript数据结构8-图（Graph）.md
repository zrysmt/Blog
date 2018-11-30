---
title: javascript数据结构8-图（Graph）
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---

 
 **图（graph）**
图由边的集合及顶点的集合组成           
**有向图：**          
![有向图](http://img.blog.csdn.net/20151208125116430)               
**无向图：**          
![这里写图片描述](http://img.blog.csdn.net/20151208125202948)             
**代码：**        

```html5
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Graph</title>
</head>
<body>
<script>
	function Graph(v){
      	this.vertices=v;
      	this.edges=0;
      	this.adj=[];
     	for(var i=0;i<this.vertices;++i){
        	this.adj[i]=[];
          // this.adj[i].push("");
      }
      	this.addEdge=addEdge;
      	this.showGraph=showGraph;
      
      //深度优先搜索
      this.dfs=dfs;
      this.marked=[];
      for(var i=0;i<this.vertices;++i){
        	this.marked[i]=false;
      }

      // 广度搜索
      this.bfs=bfs;
      	
    }
  
	//   增加顶点
  	function addEdge(v,w){
      	this.adj[v].push(w);
      	this.adj[w].push(v);
		    this.edges++;      
    }
  	
  	//遍历
  	function showGraph(){
      	for(var i=0;i<this.vertices;++i){
          	document.write('<br/>');
          	document.write(i+"-->");
          	for(var j=0;j<this.vertices;++j){
              	if(this.adj[i][j]!=undefined){
                  	document.write(this.adj[i][j]+' ');
                }
            }
        }
    }
  
  	//深度优先搜索
  	function dfs(v){
  		//var w;
  		this.marked[v]=true;
    	if(this.adj[v]!=undefined){
        	document.write("<br/>访问的节点:"+v);
      }
      // for(var w in this.adj[v]){
      //console.log(this.adj[0].length);    
          var w=this.adj[v].shift();
          while(w!=undefined){
              if(!this.marked[w]){
              	this.dfs(w);
              }
              w=this.adj[v].shift();
          }
     
          //console.log(w);
          //console.log(this.adj[v]);
    }

    // 广度搜索
    function bfs(s){
        var queue=[];
        this.marked[s]=true;
        queue.push(s);//添加到队尾
        var w;  //存放邻接表
        while(queue.length>0){

           var v=queue.shift();//从队首删除
           if(v!=undefined){
              document.write("<br/>访问的节点:"+v);
           }

             w=this.adj[v].shift();
             while(w!=undefined){
                if(!this.marked[w]){
                    this.marked[w]=true;
                    queue.push(w);
                }
                 w=this.adj[v].shift();
             }
        }
    }
  	//测试
  	var  graph=new Graph(5);
  	graph.addEdge(0,1);  
  	graph.addEdge(0,2);  
  	graph.addEdge(1,3);  
  	graph.addEdge(2,4);  
    //console.log(graph);
  	//console.log(graph.adj);
  	graph.showGraph();
    document.write("<br/>");
    document.write("======深度度优先搜索=====");
    graph.dfs(0);
    document.write("<br/>");
    document.write("======广度优先搜索=====");
    var  graph1=new Graph(5);
    graph1.addEdge(0,1);  
    graph1.addEdge(0,2);  
    graph1.addEdge(1,3);  
    graph1.addEdge(2,4);  
    graph1.bfs(0);
</script>
</body>
</html>
```
**运行结果：**

> 
0-->1 2          
1-->0 3         
2-->0 4             
3-->1              
4-->2              
======深度度优先搜索=====                 
访问的节点:0              
访问的节点:1  			
访问的节点:3			
访问的节点:2			
访问的节点:4			
======广度优先搜索=====			
访问的节点:0			
访问的节点:1			
访问的节点:2			
访问的节点:3			
访问的节点:4			

深度搜索的含义：      
![深度搜索](http://img.blog.csdn.net/20151208125306927)     
广度搜索的含义：       
![广度搜索](http://img.blog.csdn.net/20151208125325697)