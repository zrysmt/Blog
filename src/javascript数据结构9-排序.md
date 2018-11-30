---
title: javascript数据结构9-排序
tags: 
- javascript
- 数据结构
categories: 数据结构
date: 2016-10-09 00:00:00
---
排序算法
1. 基本排序			
 - 冒泡排序			
 - 选择排序				
 - 插入排序				
2. 高级排序			
 - 希尔排序			
 - 归并排序			
 - 快速排序			
 -  基数排序    （见【Javascript】四、JS数据结构-队列2-基数排序）				

**注释：完整例子在最后，可以copy运行。**			
**测试数据平台：**				

```javascript
 //数组平台
    function CArray(numElements) {
        this.dataStore = [];
        this.pos = 0;
        this.numElements = numElements;
        this.insert = insert;
        this.toString = toString;
        this.clear = clear;
        this.setData = setData;
        this.swap = swap;
        for (var i = 0; i < numElements; ++i) {
            this.dataStore[i] = i;
        }

        //排序算法
        this.bubbleSort = bubbleSort; //冒泡排序
        this.selectionSort = selectionSort; //选择排序
        this.insertionSort = insertionSort; //插入排序
        this.shellSort = shellSort; //希尔排序
        this.gaps = [5, 3, 1];
    }

    function setData() {
        for (var i = 0; i < this.numElements; ++i) {
            this.dataStore[i] = Math.floor(Math.random() * (this.numElements + 1));
        }
    }

    function clear() {
        for (var i = 0; i < this.dataStore.length; ++i) {
            this.dataStore[i] = 0;
        }
    }

    function insert(element) {
        this.dataStore[this.pos++] = element;
    }

    function toString() {
        var retstr = "";
        for (var i = 0; i < this.dataStore.length; ++i) {
            retstr += this.dataStore[i] + " ";
            if (i > 0 & i % 10 == 0) {
                retstr += "<br/>";
            }
        }
        return retstr;
    }

    function swap(arr, index1, index2) {
        var temp = arr[index1];
        arr[index1] = arr[index2];
        arr[index2] = temp;
    }
```
##  1-冒泡排序

```javascript
//基本排序算法
    //1-冒泡排序  时间复杂度 n2
    function bubbleSort() {
        var numElements = this.dataStore.length;
        var temp;
        for (var i = numElements; i >= 2; --i) {
            for (var j = 0; j <= i - 1; ++j) {
                if (this.dataStore[j] > this.dataStore[j + 1]) {
                    swap(this.dataStore, j, j + 1);
                }
            }
            document.write(this.toString());
            document.write('<br/>');
        }
    }

    function bubbleSortTest(numElements) {
        // var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>冒泡排序过程：【最大的先冒出来排在最后一位】<br/>');
        var start = new Date().getTime();
        myNums.bubbleSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
        //document.write(myNums.toString());
    }
```
## 2-选择排序

```javascript
//2-选择排序
    function selectionSort() {
        var min, temp;
        for (var i = 0; i <= this.dataStore.length - 2; ++i) {
            min = i;
            for (var j = i + 1; j <= this.dataStore.length - 1; ++j) {
                if (this.dataStore[j] < this.dataStore[min]) {
                    min = j;
                }
            }
            swap(this.dataStore, i, min);
            document.write(this.toString());
            document.write('<br/>');
        }
    }

    function selectionSortTest(numElements) {
        //var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>选择排序过程：【最小的选择出来放在第一位】<br/>');
        var start = new Date().getTime();
        myNums.selectionSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
        //document.write(myNums.toString());
    }
```
## 3-插入排序

```javascript
 //3-插入排序
    function insertionSort() {
        var temp, j;
        for (var i = 1; i <= this.dataStore.length - 1; ++i) {
            temp = this.dataStore[i];
            j = i; //j=1
            while (j > 0 && (this.dataStore[j - 1] > temp)) {
                this.dataStore[j] = this.dataStore[j - 1];
                --j;
            }
            this.dataStore[j] = temp;
            document.write(this.toString());
            document.write('<br/>');
        }

    }

    function insertionSortTest(numElements) {
        //var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>插入排序过程：【从首位开始一个一个插入比较】<br/>');
        var start = new Date().getTime();
        myNums.insertionSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
    }
```
## 4-希尔排序

```javascript
//高级排序算法
    //4-shell排序
    function shellSort() {
        //gaps的长度，分为三大步
        for (var g = 0; g < this.gaps.length; ++g) { //3层 
            for (var i = this.gaps[g]; i < this.dataStore.length; ++i) {
                var temp = this.dataStore[i];
                for (var j = i; j >= this.gaps[g] && this.dataStore[j - this.gaps[g]] > temp; j -= this.gaps[g]) { //第一次循环:j=5 如果第1个数【序号0】 > 第6个数【temp序号5】
                    this.dataStore[j] = this.dataStore[j - this.gaps[g]]; //第一次循环j=5  那么第6个数【序号5】 = 第1个数【序号0】
                    //小的排在前面
                }
                this.dataStore[j] = temp;

            }
            document.write(this.toString());
            document.write('<br/>');
        }

    }

    function shellSortTest(numElements) {
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>希尔排序过程：<br/>');
        var start = new Date().getTime();
        myNums.shellSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write('排序结果是:<br/>');
        document.write(myNums.toString());
        document.write('<br/>');
        document.write("所需要的时间是：" + time);
    }
```
## 5-快速排序

```javascript
 //5-快速排序
    function qSort(arr) {
        if (arr.length == 0) {
            return [];
        }
        var left = [];
        var right = [];
        var pivot = arr[0];
        for (var i = 1; i < arr.length; i++) {
            if (arr[i] < pivot) {
                left.push(arr[i]);
            } else {
                right.push(arr[i]);
            }
        }
        return qSort(left).concat(pivot, qSort(right));
    }

    function qSortTest(numElements) {
        var a = [];
        for (var i = 0; i < numElements; i++) {
            a[i] = Math.floor((Math.random() * numElements) + 1);
        }
        document.write(a);
        document.write('<br/>');
        var start= new Date().getTime();
        document.write(qSort(a));
        document.write("<br/>需要时间是：");
        var stop= new Date().getTime();
        var time=stop-start;
        document.write(time);
    }
```

## **完整例子：**

```javascript
<!DOCTYPE html>
<html>
<meta charset="utf-8">

<head>
    <title>排序算法总结</title>
</head>

<body>
    <input type="button" value="冒泡排序过程查看" onclick="bubbleSortTest(10)">
    <input type="button" value="选择排序过程查看" onclick="selectionSortTest(10)">
    <input type="button" value="插入排序过程查看" onclick="insertionSortTest(10)">
    <input type="button" value="希尔排序过程查看" onclick="shellSortTest(10)">
    <input type="button" value="快速排序过程查看" onclick="qSortTest(10)">
    <br/>
    <p>查看执行函数，建议先删除排序函数中的document.write过程打印,即是:</p>
    <input type="button" value="冒泡排序执行时间" onclick="bubbleSortTest(10000)">
    <input type="button" value="选择排序执行时间" onclick="selectionSortTest(10000)">
    <input type="button" value="插入排序执行时间" onclick="insertionSortTest(10000)">
    <input type="button" value="希尔排序执行时间" onclick="shellSortTest(10000)">
    <input type="button" value="快速排序执行时间" onclick="qSortTest(10000)">
    <script type="text/javascript">
    //===========================================
    //数组平台
    function CArray(numElements) {
        this.dataStore = [];
        this.pos = 0;
        this.numElements = numElements;
        this.insert = insert;
        this.toString = toString;
        this.clear = clear;
        this.setData = setData;
        this.swap = swap;
        for (var i = 0; i < numElements; ++i) {
            this.dataStore[i] = i;
        }

        //排序算法
        this.bubbleSort = bubbleSort; //冒泡排序
        this.selectionSort = selectionSort; //选择排序
        this.insertionSort = insertionSort; //插入排序
        this.shellSort = shellSort; //希尔排序
        this.gaps = [5, 3, 1];
    }

    function setData() {
        for (var i = 0; i < this.numElements; ++i) {
            this.dataStore[i] = Math.floor(Math.random() * (this.numElements + 1));
        }
    }

    function clear() {
        for (var i = 0; i < this.dataStore.length; ++i) {
            this.dataStore[i] = 0;
        }
    }

    function insert(element) {
        this.dataStore[this.pos++] = element;
    }

    function toString() {
        var retstr = "";
        for (var i = 0; i < this.dataStore.length; ++i) {
            retstr += this.dataStore[i] + " ";
            if (i > 0 & i % 10 == 0) {
                retstr += "<br/>";
            }
        }
        return retstr;
    }

    function swap(arr, index1, index2) {
        var temp = arr[index1];
        arr[index1] = arr[index2];
        arr[index2] = temp;
    }
    //使用测试平台类

    //=================================================================
    //基本排序算法
    //1-冒泡排序  时间复杂度 n2
    function bubbleSort() {
        var numElements = this.dataStore.length;
        var temp;
        for (var i = numElements; i >= 2; --i) {
            for (var j = 0; j <= i - 1; ++j) {
                if (this.dataStore[j] > this.dataStore[j + 1]) {
                    swap(this.dataStore, j, j + 1);
                }
            }
            document.write(this.toString());
            document.write('<br/>');
        }
    }

    function bubbleSortTest(numElements) {
        // var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>冒泡排序过程：【最大的先冒出来排在最后一位】<br/>');
        var start = new Date().getTime();
        myNums.bubbleSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
        //document.write(myNums.toString());
    }
    //2-选择排序
    function selectionSort() {
        var min, temp;
        for (var i = 0; i <= this.dataStore.length - 2; ++i) {
            min = i;
            for (var j = i + 1; j <= this.dataStore.length - 1; ++j) {
                if (this.dataStore[j] < this.dataStore[min]) {
                    min = j;
                }
            }
            swap(this.dataStore, i, min);
            document.write(this.toString());
            document.write('<br/>');
        }
    }

    function selectionSortTest(numElements) {
        //var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>选择排序过程：【最小的选择出来放在第一位】<br/>');
        var start = new Date().getTime();
        myNums.selectionSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
        //document.write(myNums.toString());
    }

    //3-插入排序
    function insertionSort() {
        var temp, j;
        for (var i = 1; i <= this.dataStore.length - 1; ++i) {
            temp = this.dataStore[i];
            j = i; //j=1
            while (j > 0 && (this.dataStore[j - 1] > temp)) {
                this.dataStore[j] = this.dataStore[j - 1];
                --j;
            }
            this.dataStore[j] = temp;
            document.write(this.toString());
            document.write('<br/>');
        }

    }

    function insertionSortTest(numElements) {
        //var numElements = 10;
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>插入排序过程：【从首位开始一个一个插入比较】<br/>');
        var start = new Date().getTime();
        myNums.insertionSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write("所需要的时间是：" + time);
    }
    //高级排序算法
    //4-shell排序
    function shellSort() {
        //gaps的长度，分为三大步
        for (var g = 0; g < this.gaps.length; ++g) { //3层 
            for (var i = this.gaps[g]; i < this.dataStore.length; ++i) {
                var temp = this.dataStore[i];
                for (var j = i; j >= this.gaps[g] && this.dataStore[j - this.gaps[g]] > temp; j -= this.gaps[g]) { //第一次循环:j=5 如果第1个数【序号0】 > 第6个数【temp序号5】
                    this.dataStore[j] = this.dataStore[j - this.gaps[g]]; //第一次循环j=5  那么第6个数【序号5】 = 第1个数【序号0】
                    //小的排在前面
                }
                this.dataStore[j] = temp;

            }
            document.write(this.toString());
            document.write('<br/>');
        }

    }

    function shellSortTest(numElements) {
        var myNums = new CArray(numElements);
        myNums.setData();
        document.write(myNums.toString());
        document.write('<br/>希尔排序过程：<br/>');
        var start = new Date().getTime();
        myNums.shellSort();
        var stop = new Date().getTime();
        var time = stop - start;
        document.write('排序结果是:<br/>');
        document.write(myNums.toString());
        document.write('<br/>');
        document.write("所需要的时间是：" + time);
    }

    //5-快速排序
    function qSort(arr) {
        if (arr.length == 0) {
            return [];
        }
        var left = [];
        var right = [];
        var pivot = arr[0];
        for (var i = 1; i < arr.length; i++) {
            if (arr[i] < pivot) {
                left.push(arr[i]);
            } else {
                right.push(arr[i]);
            }
        }
        return qSort(left).concat(pivot, qSort(right));
    }

    function qSortTest(numElements) {
        var a = [];
        for (var i = 0; i < numElements; i++) {
            a[i] = Math.floor((Math.random() * numElements) + 1);
        }
        document.write(a);
        document.write('<br/>');
        var start= new Date().getTime();
        document.write(qSort(a));
        document.write("<br/>需要时间是：");
        var stop= new Date().getTime();
        var time=stop-start;
        document.write(time);
    }
    </script>
</body>

</html>

```