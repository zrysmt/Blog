---
title: OpenLayers 3实践与原理探究1-ol2 VS ol3
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---
本文的重点在OpenLayers 3，对于OpenLayers 2简单比较说明。
下文中OpenLayers 2简称OL2，OpenLayers 3简称OL3
# 1.OL 2 VS OL 3简单源码和实例
## 1.1 OpenLayers 2
OpenLayers 是一个专为Web GIS 客户端开发提供的JavaScript 类库包，用于实现标准格式发布的地图数据访问。从OpenLayers2.2版本以后，OpenLayers已经将所用到的Prototype.js组件整合到了自身当中，并不断在Prototype.js的基础上完善面向对象的开发，Rico用到地方不多，只是在OpenLayers.Popup.AnchoredBubble类中圆角化DIV
## 1.1.1 OpenLayers 2源码简要分析
源码分析，下载版本为`2.13.1`,源码位置在`lib/OpenLayers`
![ol2源码文件](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol2%E6%BA%90%E7%A0%81%E6%96%87%E4%BB%B6.png)
由于OL2不是本文的重点，所以下面会简诉关于OL2源码。
`lib/OpenLayers/Layer/Image.js`

```javascript
OpenLayers.Layer.Image = OpenLayers.Class(OpenLayers.Layer, {
    isBaseLayer: true,
    url: null,
    extent: null,
    size: null,
    tile: null,
    aspectRatio: null,
    initialize: function(name, url, extent, size, options) {
        //...
    },    
    destroy: function() {
        //...
    },
    clone: function(obj) {
        //...
    },    
    setMap: function(map) {
        //...
    },
    moveTo:function(bounds, zoomChanged, dragging) {
        //...
    }, 
    setTileSize: function() {
        //...
    },
    addTileMonitoringHooks: function(tile) {
        //...
    },
	//...
    CLASS_NAME: "OpenLayers.Layer.Image"
});
```
OpenLayers.Layer.Image 继承了OpenLayers.Layer类
OpenLayers.Layer在上级目录下`lib/OpenLayers/Layer.js`

```javascript
OpenLayers.Layer = OpenLayers.Class({
	id: null,
	name: null,
	//...
	initialize: function(name, options) {
		//...
	},
	destroy: function(setNewBaseLayer) {
	    //...
	},
	clone: function (obj) {
	    //...
	},   
	getOptions: function() {
	    //...
	},
	CLASS_NAME: "OpenLayers.Layer"
});
```
通过分析我们可以知道，`OpenLayers.Class`的第一个参数是等号前面类(A类)继承的类;第二个参数是个对象，前面的部分作为A类的属性，后面的作为A类的方法。通过观察前三个方法我们可以看出，前三个方法依次为`initialize`,`destroy`,`clone`,分别为A类的初始化构造函数，销毁函数，克隆函数。

## 1.1.2 OpenLayers 2简单实例

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>
        创建一个简单的电子地图
    </title>
    <!-- 加载OpenLayers 类库 -->
    <script type="text/javascript" src="http://www.openlayers.cn/olapi/OpenLayers.js">
    </script>
    <style>
    html,
    body {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
    }
    </style>
    <!-- 关键代码在这里了 -->
    <script type="text/javascript">
    function init() {
        // 使用指定的文档元素创建地图
        var map = new OpenLayers.Map("rcp1_map");
        // 创建一个 OpenStreeMap raster layer
        // 把这个图层添加到map中
        var osm = new OpenLayers.Layer.OSM();
        map.addLayer(osm);
        // 设定视图缩放地图程度为最大
        map.zoomToMaxExtent();
    }
    </script>
</head>

<body onload="init()">
    <div id="rcp1_map" style="width: 100%;height: 100%;">
    </div>
</body>

</html>

```
![ol2实例显示效果](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol2%E5%AE%9E%E4%BE%8B%E6%95%88%E6%9E%9C.png)
## 1.2 OpenLayers 3
## 1.2.1 OpenLayers 3源码简要分析
OpenLayers 3对OpenLayers网络地图库进行了根本的重新设计。版本2虽然被广泛使用，但从JavaScript开发的早期发展阶段开始，已日益现实出它的落后。 OL3已运用现代的设计模式从底层重写。

最初的版本旨在支持第2版提供的功能，提供大量商业或免费的瓦片资源以及最流行的开源矢量数据格式。与版本2一样，数据可以被任意投影。最初的版本还增加了一些额外的功能，如能够方便地旋转地图以及显示地图动画。
OpenLayers3同时设计了一些主要的新功能，如显示三维地图，或使用WebGL快速显示大型矢量数据集，这些功能将在以后的版本中加入。

源码分析，下载版本为`3.17.1`,源码位置在`ol/ol`文件夹下
![ol3源码文件](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol3%E6%BA%90%E7%A0%81%E6%96%87%E4%BB%B6.png)
`ol/ol/layer/layer.js`

```javascript
goog.provide('ol.layer.Layer');

goog.require('ol.events');
goog.require('ol.events.EventType');
goog.require('ol');
goog.require('ol.Object');
goog.require('ol.layer.Base');
goog.require('ol.layer.LayerProperty');
goog.require('ol.object');
goog.require('ol.render.EventType');
goog.require('ol.source.State');

ol.layer.Layer = function(options) {
  var baseOptions = ol.object.assign({}, options);
  delete baseOptions.source;

  ol.layer.Base.call(this, /** @type {olx.layer.BaseOptions} */ (baseOptions));//继承extends {ol.layer.Base}
  this.mapPrecomposeKey_ = null;
  this.mapRenderKey_ = null;
  this.sourceChangeKey_ = null;

  if (options.map) {
    this.setMap(options.map);
  }

  ol.events.listen(this,
      ol.Object.getChangeEventType(ol.layer.LayerProperty.SOURCE),
      this.handleSourcePropertyChange_, this);

  var source = options.source ? options.source : null;
  this.setSource(source);
};
ol.inherits(ol.layer.Layer, ol.layer.Base);

ol.layer.Layer.visibleAtResolution = function(layerState, resolution) {
   //...
};

ol.layer.Layer.prototype.getLayersArray = function(opt_array) {
   //...
};
ol.layer.Layer.prototype.getLayerStatesArray = function(opt_states) {
   //...
};
   //...
```
OL3利用[google的closure库](https://developers.google.com/closure/)组织代码的，库文件在源码的`closure-library`文件夹下。
在源码中用到的常用的功能是：
- 类声明：使用goog.provide方法声明和注册一个类(表示自己能提供什么功能)。
- 依赖声明：使用goog.require方法声明具体依赖的其它类(表示提供这些功能需要额外的哪功能的支持)
这一点也是OL2和OL3源代码重写的最重要的变化。

## 1.2.1 OpenLayers 3实例

```html
<!doctype html>
<html lang="en">

<head>
    <link rel="stylesheet" href="http://openlayers.org/en/v3.17.1/css/ol.css" type="text/css">
    <style>
    .map {
        height: 400px;
        width: 100%;
    }
    </style>
    <script src="http://openlayers.org/en/v3.17.1/build/ol.js" type="text/javascript"></script>
    <title>OpenLayers 3 example</title>
</head>

<body>
    <h2>My Map</h2>
    <div id="map" class="map"></div>
    <script type="text/javascript">
    var map = new ol.Map({
        target: 'map',
        layers: [
            new ol.layer.Tile({
                source: new ol.source.OSM()
            })
        ],
        view: new ol.View({
            center: ol.proj.fromLonLat([37.41, 8.82]),
            zoom: 4
        })
    });
    </script>
</body>

</html>
```
![ol3实例显示效果](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol3%E5%AE%9E%E4%BE%8B%E6%95%88%E6%9E%9C.png)
# 2.OL 2 VS OL 3变化简要总结
|比较部分|OL2|OL3|
|------|:------------:|------:|
|源码|使用Prototype和Rico|使用Clourse|
|实现| var map = new OpenLayers.Map("rcp1_map");var osm = new OpenLayers.Layer.OSM();map.addLayer(osm);|var map = new ol.Map({});|
|渲染方式|dom|canvas/webgl/dom|

补充说明：
三种渲染方式可以在`ol/ol/map.js`看到

```javascript
ol.DEFAULT_RENDERER_TYPES = [
  ol.RendererType.CANVAS,
  ol.RendererType.WEBGL,
  ol.RendererType.DOM
];
```
- dom是OL2常用的方式，显示地图时候(如openstreetmap)会采用img集合的方式。
- OL3常用的渲染方式是基于canvas绘图技术。
- WebGL快速显示大型矢量数据集
OL2和OL3的差别比较大，目前看从OL2升级OL3的比较麻烦。
最后分享一个ol2和ol3的[资料](http://pan.baidu.com/s/1dFt3tIx)，包含源码，教程和书籍


参考阅读：
- 1.2-2015_OpenLayers_3_入门教程详细版.pdf
- [OpenLayers](http://www.openlayers.cn/portal.php)
- [OpenLayers 3官网](http://openlayers.org/)
- [OpenLayers 2官网](http://openlayers.org/two/)
- [google closure库](https://developers.google.com/closure/)
- [JS 库浅析之 Google Closure](http://www.kuqin.com/webpagedesign/20100309/81059.html)