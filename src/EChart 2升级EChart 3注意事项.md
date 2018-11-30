---
title: EChart 2升级EChart 3注意事项
tags: 
- FE
- 可视化
- EChart
categories: 前端技术
date: 2016-09-27 19:28:35
---
本文是根据自己的实践进行总结过来的，是不完全的所有升级注意事项。
如果想直接看结果，请移步到第4部分内容
# 1.背景
[EChart 3](http://echarts.baidu.com/index.html)是在2015年12月发布的新版本,相比较[EChart 2](http://echarts.baidu.com/echarts2/),主要的变化总结如下：
- 1) 支持了直角坐标系（catesian，同 grid）、极坐标系（polar）、地理坐标系（geo）
- 2) 移动端的优化,说明白就是将源码体积减小
- 3) 新增更多图表类型，增加了一些动态效果
- 4) 更丰富的交互模式
- 5) EChart 2推荐使用模块化单文件引入，EChart 3可以选择独立文件或者在webpack中使用模块化(在第2部分说明)
- 6) 异步数据加载与更新(在第3部分说明)。

# 2 模块/非模块
## 2.1 EChart 2模块化引入
EChart 2自带有模块化机制，不用使用其它AMD/CMD库就可以require进来echarts提供的模块
EChart 2 引入进来的目录结构：
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echart%202%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)
示例代码：

```html
<!DOCTYPE html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="height:400px"></div>
    <!-- ECharts单文件引入 -->
    <script src="http://echarts.baidu.com/build/dist/echarts.js"></script>
    <script type="text/javascript">
        // 路径配置
        require.config({
            paths: {
                echarts: 'http://echarts.baidu.com/build/dist'
            }
        });
        
        // 使用
        require(
            [
                'echarts',
                'echarts/chart/bar' // 使用柱状图就加载bar模块，按需加载
            ],
            function (ec) {
                // 基于准备好的dom，初始化echarts图表
                var myChart = ec.init(document.getElementById('main')); 
                
                var option = {
                    tooltip: {
                        show: true
                    },
                    legend: {
                        data:['销量']
                    },
                    xAxis : [
                        {
                            type : 'category',
                            data : ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
                        }
                    ],
                    yAxis : [
                        {
                            type : 'value'
                        }
                    ],
                    series : [
                        {
                            "name":"销量",
                            "type":"bar",
                            "data":[5, 20, 40, 10, 10, 20]
                        }
                    ]
                };
        
                // 为echarts对象加载数据 
                myChart.setOption(option); 
            }
        );
    </script>
</body>
```
## 2.2 EChart 2非模块化引入
```html
<!DOCTYPE html>
<head>
    <meta charset="utf-8">
    <title>ECharts</title>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="height:400px"></div>
    <!-- ECharts单文件引入 -->
    <script src="http://echarts.baidu.com/build/dist/echarts-all.js"></script>
    <script type="text/javascript">
        // 基于准备好的dom，初始化echarts图表
        var myChart = echarts.init(document.getElementById('main')); 
        
        var option = {
            tooltip: {
                show: true
            },
            legend: {
                data:['销量']
            },
            xAxis : [
                {
                    type : 'category',
                    data : ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
                }
            ],
            yAxis : [
                {
                    type : 'value'
                }
            ],
            series : [
                {
                    "name":"销量",
                    "type":"bar",
                    "data":[5, 20, 40, 10, 10, 20]
                }
            ]
        };

        // 为echarts对象加载数据 
        myChart.setOption(option); 
    </script>
</body>
```

## 2.3 EChart 3模块化引入
**通过npm命令安装**

```bash
npm install echarts --save
```
**按需引入 ECharts 图表和组件**

```javascript
// 引入 ECharts 主模块
var echarts = require('echarts/lib/echarts');
// 引入柱状图
require('echarts/lib/chart/bar');
// 引入提示框和标题组件
require('echarts/lib/component/tooltip');
require('echarts/lib/component/title');

// 基于准备好的dom，初始化echarts实例
var myChart = echarts.init(document.getElementById('main'));
// 绘制图表
myChart.setOption({
    title: { text: 'ECharts 入门示例' },
    tooltip: {},
    xAxis: {
        data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
    },
    yAxis: {},
    series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
    }]
});
```
## 2.4 EChart 3非模块化引入
同2.2类似

```html
 <script src="echarts.min.js"></script>
 ```

# 3.异步数据加载与更新
由于地图模块分辨率变高，为了不增大源码的体积，地图模块采用按照需要下载引入
在[地图下载页面](http://echarts.baidu.com/download-map.html),下载需要的世界地图/中国地图/中国分省地图，下载后的格式有js和json两种。
对应js格式的，引入的方式是通过script标签

```html
<script src="echarts.js"></script>
<script src="map/js/china.js"></script>
<script>
var chart = echarts.init(document.getElementById('main'));
chart.setOption({
    series: [{
        type: 'map',
        map: 'china'
    }]
});
</script>
```
通过JSON格式引入就可以实现异步数据加载与更新

```javascript
$.get('map/json/china.json', function (chinaJson) {
    echarts.registerMap('china', chinaJson);
    var chart = echarts.init(document.getElementById('main'));
    chart.setOption({
        series: [{
            type: 'map',
            map: 'china'
        }]
    });
});
```

# 4.升级总结
## 4.1 配置变化举例

| EChart 2      | EChart 3      | 
| ------------- |:-------------:|
| option.series.mapLocation      | 删去，使用left，top，bottom，right定义位置|
| option.series.textFixed      | 删去地区的名称文本位置修正      |   
|dataRange颜色标识属性(示例4.1-1)|visualMap|
|单个echarts 实例中最多只能存在一个 grid 组件|ECharts 3 中可以存在任意个 grid 组件
一个网格中(示例4.1-2)|
| addData , setSeries 方法设置配置项|统一使用setOption(示例4.1-3)|
|级联this.myChart = ec.init(dom).showLoading({effect:'bubble'}).hideLoading();|不支持|
| myChart.component.tooltip.showTip 这种形式调用相应的接口触发图表行为|dispatchAction(示例4.1-4)|
示例4.1-1

```javascript
     dataRange: {
         realtime: false,
         itemHeight: 80,
         splitNumber:6,
         borderWidth:1, 
         textStyle: { color: '#333333' },
         text: ['高', '低'],
         calculable: true
     },
//=======================================================
    visualMap: {
        min: 0,
        max: 1000000,
        text: ['High', 'Low'],
        realtime: false,
        calculable: true,
        inRange: {
            color: ['lightskyblue', 'yellow', 'orangered']
        }
    },
```
示例4.1-2
图略，官网例子：http://echarts.baidu.com/gallery/editor.html?c=scatter-anscombe-quartet
部分配置属性变化，需要修改

示例4.1-3

```javascript
myChart.setOption({
    visualMap: {
        inRange: {
            color: ...
        }
    }
})//注意最后一定要再次setOption(option);option是配置对象
```
示例4.1-4

```javascript
myChart.on('brushselected', renderBrushed);
setTimeout(function() {
    self.myChart.dispatchAction({
        type: 'brush',
        areas: [{
            geoIndex: 0,
            brushType: 'polygon',
            coordRange: [
                [119.72, 34.85],
                [117.05, 34.06],
                [117.49, 33.75],
                [123.16, 29.92],
                [121.64, 34.08]
            ]
        }]
    });
}, 0);


function renderBrushed(params) { //... ...
}
```

## 4.2 第2部分的模块与非模块更改时候需要注意
## 4.3 地图模块
EChart 2:

```javascript
chart.setOption({
    series: [{
        type: 'map',
        map: 'china'//'world'
    }]
});
```
EChart 3，需要下载地图，使用方式见第3部分。
