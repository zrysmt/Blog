---
title: javascript数据结构4-队列2-基数排序
tags: 
- javascript
- 数据结构
categories: 数据结构
---

第一次按个位上的数字进行排序，第二次按十位上的数字进行排序  

排序：91, 46, 85, 15, 92, 35, 31, 22
**经过基数排序第一次扫描**之后，数字被分配到如下盒子中：

```
Bin 0:
Bin 1: 91, 31
Bin 2: 92, 22
Bin 3:
Bin 4:
Bin 5: 85, 15, 35
Bin 6: 46
Bin 7:
Bin 8:
Bin 9:
```

根据盒子的顺序，对数字进行第一次排序的结果如下：
91, 31, 92, 22, 85, 15, 35, 46
然后根据**十位上的数值再将上次排序**的结果分配到不同的盒子中：

```
Bin 0:
Bin 1: 15
Bin 2: 22
Bin 3: 31, 35
Bin 4: 46
Bin 5:
Bin 6:
Bin 7:
Bin 8: 85
Bin 9: 91, 92
```
Javascript实现代码：

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
  //========================================================================
     function distribute(nums, queues, n, digit) {
        for (var i = 0; i < n; ++i) {
          if (digit == 1) {  //个位
              queues[nums[i]%10].enqueue(nums[i]);
            } else {  //十位
              queues[Math.floor(nums[i] / 10)].enqueue(nums[i]);
            }
          }
      }
    function collect(queues, nums) {
        var i = 0;
        for (var digit = 0; digit < 10; ++digit) {
            while (!queues[digit].isEmpty()) {  
            nums[i++] = queues[digit].dequeue();
            }
          }
     }
     function dispArray(arr){
       for (var i = 0; i < arr.length; i++) {
         document.write(arr[i]+" ");
         // document.write('<br/>');
       };
     }
    var  N=200;//N是要排序的个数 基数排序测试时间：: 4.000ms
    var queues=[];
    for (var i = 0; i < 10; i++) {
      queues[i]=new Queue(); //分别存储Bin0--bin9
     };
    var nums=[];
    for (var i = 0; i < N; i++) {
       nums[i]=Math.floor(Math.random()*101);
       // document.write(nums[i]);
       // document.write('<br/>');
    }
    
    /*********************/
    /*测试一下时间*/
    console.time("基数排序测试时间：");//改变N的值进行测试
    document.write("原数据：");
    dispArray(nums);
    distribute(nums,queues,N,1);
    collect(queues,nums);   //nums分散了，收集起来
    document.write("<br/>个位排序后：");
    dispArray(nums);
    distribute(nums,queues,N,10);
    collect(queues,nums); 
    document.write("<br/>基数排序后：");  
    dispArray(nums);
    console.timeEnd("基数排序测试时间：");
    
   </script>
    
</body>
</html>
```


