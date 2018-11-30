---
title: OpenLayers 3实践与原理探究4.3-ol3源码分析-Source,Layer
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---

# 3.Source
`ol/ol/Source`文件夹下 

## 3.1构造函数
### 3.1.1 `ol.source.Source` ol.source的基础类
`ol/ol/Source/source.js`

```javascript
ol.source.Source = function(options) {}
```
### 3.1.2 `ol.source.OSM` 
`ol/ol/Source/osmsource.js`
`openStreetMap`：

```javascript
ol.source.OSM = function(opt_options) {}
```
具体不进行展开描述。
运用实例：

```javascript
var osmSource = new ol.source.OSM();
```
### 3.1.3 `ol.source.TileWMS` 
先看个例子：

```javascript
var map = new ol.Map({
        target: 'map', //document.getElementById("map")
        layers: [
            new ol.layer.Tile({
                title: "Global Imagery",
                source: new ol.source.TileWMS({
                    url: 'http://demo.boundlessgeo.com/geoserver/wms',
                    params: {
                        'LAYERS': 'ne:NE1_HR_LC_SR_W_DR'
                    }
                })
            })
        ],
    
    });
```
我们可以一路找下源头：
`tilewmssource.js`-->`tileimagessource.js`-->`uritilesource.js`-->`tilespurce.js`-->`source.js`
我们可以发现实例中的`source.url`是在`uritilesource.js`
处理的
我们先考虑单url的情况(当然存在url数组的情况)

```javascript
if (options.url) {
    this.setUrl(options.url);
}
//... ...
ol.source.UrlTile.prototype.setUrl = function(url) {
  var urls = this.urls = ol.TileUrlFunction.expandUrl(url);
  this.setTileUrlFunction(this.fixedTileUrlFunction ?
      this.fixedTileUrlFunction.bind(this) :
      ol.TileUrlFunction.createFromTemplates(urls, this.tileGrid), url);
};
```
处理函数为`fixedTileUrlFunction`,在`tilewmssource.js`
`fixedTileUrlFunction`-->`getRequestUrl_`-->`ol.uri.appendParams(url, params)`请求地图
params包含请求地图的宽、高、分辨率、地图范围
# 4.Layer
`ol/ol/Layer`文件夹下 

## 4.1构造函数
### 4.1.1 `ol/ol/Layer/layer.js`

```javascript
ol.layer.Layer = function(options) {}
```
```javascript
ol.inherits(ol.layer.Layer, ol.layer.Base);
```
`ol.layer.Base`定义layer的基本属性和基本属性的setter，getter方法

实际在api接口上使用的是具体的图层
### 4.1.2 矢量地图`ol/ol/Layer/vectorlayer.js`
```javascript
ol.layer.Vector = function(opt_options) {
	var baseOptions = ol.object.assign({}, options);
	ol.layer.Layer.call(this, /** @type {olx.layer.LayerOptions} */ (baseOptions));
}
```
实际调用的方法，仍然在`ol/ol/Layer/layer.js`中
### 4.1.3 瓦块地图`ol/ol/Layer/titlelayer.js`
```javascript
ol.layer.Tile = function(opt_options) {
	var baseOptions = ol.object.assign({}, options);
	ol.layer.Layer.call(this, /** @type {olx.layer.LayerOptions} */ (baseOptions));
}
```
还有`heatmaplayer.js`,`imagelayer.js`,`vectortilelayer.js`对应热力图，图片地图，矢量瓦块地图
总结：
`ol/ol/Layer/layer.js`是通用的方法部分
各个具体的地图*.js是各个地图的专有方法。
运用实例：

```javascript
var map = new ol.Map({
        target: 'map', //document.getElementById("map")
        layers: [
            new ol.layer.Tile({
                title: "Global Imagery",
                source: new ol.source.TileWMS({
                    url: 'http://demo.boundlessgeo.com/geoserver/wms',
                    params: {
                        'LAYERS': 'ne:NE1_HR_LC_SR_W_DR'
                    }
                })
            })
        ],
    
    });
```
`source`在layer.js中处理

```javascript
var source = options.source ? options.source : null;
  this.setSource(source);

ol.layer.Layer.prototype.setSource = function(source) {
  this.set(ol.layer.LayerProperty.SOURCE, source);//添加到常量上，其实也是将source对象共享出去了
};
```
## 4.2方法 事件
### 4.2.1 `ol/ol/Layer/layer.js`
主要是一下方法
```javascript
ol.layer.Layer.visibleAtResolution
getLayersArray
getLayerStatesArray
getSource
getSourceState
handleSourceChange_
handleSourcePropertyChange_
setMap
setSource
```