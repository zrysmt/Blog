---
title: OpenLayers 3实践与原理探究4.2-ol3源码分析-Map,View
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---
# 1.Map
`ol/ol/map.js` 

## 1.1构造函数
```javascript
ol.Map = function(options) {
  ol.Object.call(this);//@extends {ol.Object}
   var optionsInternal = ol.Map.createOptionsInternal(options);	
}
```
**常量对象共享地图设置，并且将常量对象共享出去，作为公用变量**

```javascript
goog.provide('ol.MapProperty');
ol.MapProperty = {
  LAYERGROUP: 'layergroup',
  SIZE: 'size',
  TARGET: 'target',
  VIEW: 'view'
};
```
`createOptionsInternal`函数对map中的配置对象进行处理。

```javascript
//分别处理layer、layerGroup、target、view、renderer、controls、interactions、overlays
 var layerGroup = (options.layers instanceof ol.layer.Group) ?
      options.layers : new ol.layer.Group({layers: options.layers});
  values[ol.MapProperty.LAYERGROUP] = layerGroup;

  values[ol.MapProperty.TARGET] = options.target;

  values[ol.MapProperty.VIEW] = options.view !== undefined ?
      options.view : new ol.View();

  //......
  return {
    controls: controls,
    interactions: interactions,
    keyboardEventTarget: keyboardEventTarget,
    logos: logos,
    overlays: overlays,
    rendererConstructor: rendererConstructor,
    values: values
  };
```
运用实例-初始化：

```javascript
var map = new ol.Map({
    controls: ol.control.defaults().extend([
        new ol.control.ScaleLine(), //比例尺
    ]),
    layers: [bglayer, vector], //添加两个图层
    target: 'map', //div#id='map'
    view: view,
    interaction:interaction
});
```



## 1.2 方法 事件
方法 事件列表：
```
addControl
addInteraction
addLayer
addOverlay
beforeRender
changed
dispatchEvent
forEachFeatureAtPixel
forEachLayerAtPixel
get
getControls
getCoordinateFromPixel
getEventCoordinate
getEventPixel
getInteractions
getKeys
getLayerGroup
getLayers
getOverlayById
getOverlays
getPixelFromCoordinate
getProperties
getRevision
getSize
getTarget
getTargetElement
getView
getViewport
hasFeatureAtPixel
on
once
removeControl
removeInteraction
removeLayer
removeOverlay
render
renderSync
set
setLayerGroup
setProperties
setSize
setTarget
setView
un
unByKey
unset
updateSize
```
实现方式举几个例子：

```javascript
ol.Map.prototype.addControl = function(control) {
  var controls = this.getControls();
  goog.asserts.assert(controls !== undefined, 'controls should be defined');
  controls.push(control);
};

ol.Map.prototype.addInteraction = function(interaction) {
  var interactions = this.getInteractions();
  goog.asserts.assert(interactions !== undefined,
      'interactions should be defined');
  interactions.push(interaction);
};
```
`controls`控件,控件添加到`map`上，对控件设置两个监听事件，增加、删除。

```javascript
this.controls_ = optionsInternal.controls;
this.controls_.forEach( function(control) {
        control.setMap(this);
      }, this);

  ol.events.listen(this.controls_, ol.CollectionEventType.ADD, function(event) {
        event.element.setMap(this);
      }, this);

  ol.events.listen(this.controls_, ol.CollectionEventType.REMOVE, function(event) {
        event.element.setMap(null);
      }, this);
```

# 2.View
`ol/ol/view.js` 

## 2.1构造函数

```javascript
ol.View = function(opt_options) {
  ol.Object.call(this);
  var options = opt_options || {};
  this.projection_ = ol.proj.createProjection(options.projection, 'EPSG:3857');
  //... ...
 }
```
运用实例：

```javascript
var map = new ol.Map({
	target: 'map',
	view: new ol.View({
            projection: 'EPSG:4326', //WGS 84
            center: [0, 0],
            zoom: 2,
            maxResolution: 0.703125
        }),

});
```
```javascript
goog.provide('ol.View');
goog.provide('ol.ViewHint');
goog.provide('ol.ViewProperty');
```
提供不只是一个构造函数，还有两个常量对象，其中一个常量对象包含三个常量如下所示：

```javascript
ol.ViewProperty = {
  CENTER: 'center',
  RESOLUTION: 'resolution',
  ROTATION: 'rotation'
};
```
其实，我们在对象属性中写出来的，如`center`,作用等同于用下面介绍的方法`setCenter`,其本质是写入常量中，共享出去作为公用。
## 2.2方法 事件
举例；

```javascript
ol.View.prototype.setCenter = function(center) {
  this.set(ol.ViewProperty.CENTER, center);
};
```