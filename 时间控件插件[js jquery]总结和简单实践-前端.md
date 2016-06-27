---
title: 时间控件插件[js/jquery]总结和简单实践-前端
tags: 
- FE
- javascript
- 插件
- 实践
categories: 前端技术
---

### 1.控件收集
- 比较常用的是功能丰富的[My97DatePicker](http://www.my97.net/)，缺点是界面效果一般。   

- [jquery时间插件](http://www.jq22.com/jquery-plugins%E6%97%A5%E6%9C%9F%E5%92%8C%E6%97%B6%E9%97%B4-1-jq)     
目前插件有很多种类，有的支持两个日期栏，有的带[农历](http://www.jq22.com/jquery-info7758)，带[节假日](http://www.jq22.com/jquery-info5169),带[时间](http://www.jq22.com/jquery-info4976),带[钟表显示](http://www.jq22.com/jquery-info5698),[html5响应式时间轴页面](http://www.jq22.com/jquery-info7448)。       


- 比较小的一个日期选择器[Pickday](http://www.jq22.com/jquery-info7564),不依赖任何javascript库，并且文件只有5K(目前加了一些扩展，达到11K左右)，有人已经修改成中文版的[Pikaday中文版](https://github.com/owen-hong/Pikaday)(如果地址不能访问，可以用goolge搜索)，我们下面实践的就是这个插件。          

### 2.实践
```html
<!DOCTYPE html ">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8" />
<title>picker时间控件</title>
<link rel="stylesheet" type="text/css" href="css/pikaday.css"/>
<script type="text/javascript" src="js/pikaday.min.js"></script>
</head>
<body>
<input type="text" id="datepicker1" />
<input type="text" id="datepicker2" />

<script type="text/javascript">

    var picker1 = new Pikaday(
    {
        field: document.getElementById('datepicker1'),
        firstDay: 1,
        minDate: new Date('2010-01-01'),
        maxDate: new Date('2020-12-31'),
        yearRange: [2000,2020]
    });
     var picker2 = new Pikaday(
    {
        field: document.getElementById('datepicker2'),
        firstDay: 1,
        minDate: new Date('2010-01-01'),
        maxDate: new Date('2020-12-31'),
        yearRange: [2000,2020]
    });

    var date1 = document.getElementById('datepicker1').val();
    var date2 = document.getElementById('datepicker2').val();
    if(date1>date2){
    	alert('截至时间应该大于开始时间');
    	return;
    }
</script>
</body>
</html>

```