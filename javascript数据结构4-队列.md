---
title: javascript数据结构4-队列
tags: 
- javascript
- 数据结构
categories: 数据结构
---
队列是一种先进先出（FIFO，first-in-first-out）的数据结构

**javascript代码实现队列：**

```html5
<!doctype html>
<html>
<head>
<meta charset=utf-8 /> 
	<title>Queue Sample</title>
</head>
<body>
      
   <script type="text/javascript">
   	    /*定义队列*/
   	    function Queue(){
   	    	this.dataStore=[];
   	    	this.enqueue=enqueue;
   	    	this.dequeue=dequeue;
          this.front=front;
          this.back=back;
          this.toStr=toStr;
          this.isEmpty=isEmpty;
   	    }
     
     function enqueue(element){
         this.dataStore.push(element);
     }
     
     function dequeue(){
        return this.dataStore.shift();
     }
     
     function front(){
     	return this.dataStore[0];
     }
     
     function back(){
     	return this.dataStore[this.dataStore.length-1];
     }
     
     function toStr(){
      	var retStr="";
        for(var i=0;i<this.dataStore.length;i++){
        	retStr+=this.dataStore[i]+"\n";
        }
        //console.log(retStr);
         return retStr;
     }
     //判断是否为空
     function isEmpty(){
      	  if(this.dataStore.length==0){
          		return true;
          	}else{
           		return false;   
           	}
     }
     var que=new Queue();
     que.enqueue("Tom");
     que.enqueue("Sam");
     que.enqueue("Pom");
     console.log(que.dataStore.length);
     document.write(que.toStr());
     que.dequeue();
     document.write(que.toStr());
    console.log(que.toStr);

	</script>
  </body>
</html>
```
**举个案例：**
常用队列模拟排队的人。下面我们使用队列来模拟跳方块舞的人。
当男男女女来到舞池，他们按照自己的性别排成两队。当舞池中有地方空出来时，选两个队列中的第一个人组成舞伴。他们身后的人各自向前移动一位，变成新的队首。当一对舞伴迈入舞池时，主持人会大声喊出他们的名字。当一对舞伴走出舞池，且两排队伍中有任意一队没人时，主持人也会把这个情况告诉大家。

```html5
<!doctype html>
<html>
<head>
<meta charset=utf-8 /> 
	<title>Queue Sample</title>
</head>
<body>
      
   <script type="text/javascript">
   	    /*定义队列*/
   	    function Queue(){
   	    	this.dataStore=[];
   	    	this.enqueue=enqueue;
   	    	this.dequeue=dequeue;
          this.front=front;
          this.back=back;
          this.toStr=toStr;
          this.isEmpty=isEmpty;
   	    }
     
     function enqueue(element){
         this.dataStore.push(element);
     }
     
     function dequeue(){
        return this.dataStore.shift();
     }
     
     function front(){
     	return this.dataStore[0];
     }
     
     function back(){
     	return this.dataStore[this.dataStore.length-1];
     }
     
     function toStr(){
      	var retStr="";
        for(var i=0;i<this.dataStore.length;i++){
        	retStr+=this.dataStore[i]+"\n";
        }
        //console.log(retStr);
         return retStr;
     }
     //判断是否为空
     function isEmpty(){
      	  if(this.dataStore.length==0){
          		return true;
          	}else{
           		return false;   
           	}
     }
    
      //舞蹈员性别  姓名
    var  allStr="F Shun F Tim M Huipin M Lanlan F Ping F Li F Lou M Funr F Sun M Pop";  
     
      function Dancer(name,sex){
      	this.name=name;
        this.sex=sex;
      }
     
     //男女分队
     function getDancers(males,females){
          var numbers=allStr.split(" ");
         //document.write(numbers);
          //console.log(numbers);
          for(var i=0;i<numbers.length-1;++i){
             //var dances=numbers[i].trim();
             var sex=numbers[i];
             i++;
             var name=numbers[i]; 
            //console.log(name);
            //console.log(sex);
            if(sex == "F"){  //??????
            	 famaleDances.enqueue(new Dancer(name,sex));
                 console.log(famaleDances);
            }else{
            	 maleDances.enqueue(new Dancer(name,sex));//整体对象存在队列中
            }
         }
     } 
       //队首男女就是要出队的
       function dance(males,famales){
       		document.write("The dance parter are: ");
            document.write("<br />");
            while(!males.isEmpty() && !famales.isEmpty()){
                fperson=famales.dequeue();
               // console.log(fperson);
                document.write("The Famale dance  is:"+fperson.name);               
                person=males.dequeue();
                document.write(" and The Male dance  is:"+person.name);
              	document.write("<br />");
            }
       } 
      var maleDances=new Queue();
      var famaleDances=new Queue();
         // document.write("1");
      getDancers(maleDances,famaleDances);
      dance(maleDances,famaleDances);
     
       if(!famaleDances.isEmpty()){
          document.write(famaleDances.front().name+"is waiting");
       }
       
       if(!maleDances.isEmpty()){
          document.write(maleDances.front().name+"is waiting");
       }
   </script>
</body>
</html>
```