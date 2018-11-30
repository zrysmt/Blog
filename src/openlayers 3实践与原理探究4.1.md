---
title: OpenLayers 3实践与原理探究4.1-ol3源码分析-底层基础
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---
因为下面的内容会分模块介绍源码，所以这里为了方便，首先介绍源码的目录结构
在OpenLayers 3官网的[下载页面](http://openlayers.org/download/)下载我们在开发工程中需要的文件(如：v3.17.1.zip)，注意如果需要编译源代码，需要下载包含编译功能的文件包：https://github.com/openlayers/ol3/releases 下载指定release版本的源码，注意是Source code (zip)或者Source code (tar.gz)。
![ol3源码目录结构.png](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol3%E6%BA%90%E7%A0%81%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)
- `apidoc`是ol3的api文档，打开`ol.html`就可以在浏览器中离线使用，当然也可以在官网中查看api；
- `build`是ol3编译过的文件，工程开发中可以直接使用，下部分的案例是基于离线的源码的；
- `closure-library`是google的closure库文件夹；
- `css`里面只有`ol.css`一个文件，是定义ol3的全局样式，项目开发中需要引入；
- `doc`提供给我们一些的案例，打开`quickstart.html`即可看到快速开始的案例；
- `examples`是比较丰富的例子，和官网中的examples一样；
- `ol`就是我们要分析的源码文件夹；
- `ol.ext`是ol3所要使用的js库。

`ol/ol`文件夹下是我们分析的源码，分析基本思路：文件夹下的文件是公用的部分(A部分)，文件夹是分部分写的(B部分)。
# 0.底层基础
## 0.1 `ol.js`
第一行就可以看出，`ol.js`提供全局的第一命名空间`ol`

```javascript
goog.provide('ol');
```
唯一的一个方法是：继承

```javascript
ol.inherits = function(childCtor, parentCtor) {
  childCtor.prototype = Object.create(parentCtor.prototype);
  childCtor.prototype.constructor = childCtor;
};
```
## 0.2 `object.js`

```javascript
goog.provide('ol.Object');
goog.provide('ol.ObjectEvent');
goog.provide('ol.ObjectEventType');

goog.require('ol.Observable');
goog.require('ol.events');
goog.require('ol.events.Event');
goog.require('ol.object');
```
ol命名空间下所有的基本对象，比如`map`对象，`feature`矢量地图对象，都应该建立在`ol.Object`基础上。如：
***
`map.js`

```javascript
ol.Object.call(this);
ol.inherits(ol.Map, ol.Object);
```
`feature.js`

```javascript
ol.Object.call(this);
ol.inherits(ol.Feature, ol.Object);
```
***
通过这行代码：
```javascript
ol.inherits(ol.Object, ol.Observable);
```
我们发现`ol.Object`继承`ol.Observable`
`Observable.js`

```javascript
ol.inherits(ol.Observable, ol.events.EventTarget);
```
我们发现`ol.Observable`继承`ol.EventTarget`；
这样，我们可以知道继承`ol.Object`后也就继承了基础事件`ol.events`。
## 0.3 `events.js`
ol3的基础事件

```javascript
goog.provide('ol.events');
goog.provide('ol.events.EventType');
goog.provide('ol.events.KeyCode');

goog.require('ol.object');
```
提供的所有基础事件

```javascript
ol.events.EventType = {
  CHANGE: 'change',
  CLICK: 'click',
  DBLCLICK: 'dblclick',
  DRAGENTER: 'dragenter',
  DRAGOVER: 'dragover',
  DROP: 'drop',
  ERROR: 'error',
  KEYDOWN: 'keydown',
  KEYPRESS: 'keypress',
  LOAD: 'load',
  MOUSEDOWN: 'mousedown',
  MOUSEMOVE: 'mousemove',
  MOUSEOUT: 'mouseout',
  MOUSEUP: 'mouseup',
  MOUSEWHEEL: 'mousewheel',
  MSPOINTERDOWN: 'mspointerdown',
  RESIZE: 'resize',
  TOUCHSTART: 'touchstart',
  TOUCHMOVE: 'touchmove',
  TOUCHEND: 'touchend',
  WHEEL: 'wheel'
};
```
我们分析一下提供的几个方法

```javascript
ol.events.bindListener_ = function(listenerObj) {};
ol.events.listen = function(target, type, listener, opt_this, opt_once) {};
ol.events.unlisten = function(target, type, listener, opt_this) {};
ol.events.unlistenAll = function(target) {};
```
其中方法名末尾带有"_"为私有方法，不带的为提供出去的共有方法。
## 0.4 `math.js`
提供基础的数学运算方法，角度转化弧度函数如：

```javascript
ol.math.toRadians = function(angleInDegrees) {
  return angleInDegrees * Math.PI / 180;
};
```
## 0.5 `animation.js`
提供bounce、pan、rotate、zoom四种方法
## 0.6 `collection.js`
对ol命名空间下的对象集合的操作。

```javascript
goog.provide('ol.Collection');
goog.provide('ol.CollectionEvent');
goog.provide('ol.CollectionEventType');

goog.require('ol.events.Event');
goog.require('ol.Object');
```
```javascript
ol.CollectionEventType = {
  ADD: 'add'
  REMOVE: 'remove'
};
```
继承

```javascript
ol.inherits(ol.CollectionEvent, ol.events.Event);
ol.inherits(ol.Collection, ol.Object);
```
方法举例：

```javascript
ol.Collection.prototype.remove = function(elem) {
  var arr = this.array_;
  var i, ii;
  for (i = 0, ii = arr.length; i < ii; ++i) {
    if (arr[i] === elem) {
      return this.removeAt(i);
    }
  }
  return undefined;
};
```
## 0.7 `uri.js`
通过url加载地图，其中`params`包含请求地图的宽、高、分辨率、地图范围

```javascript
ol.uri.appendParams = function(uri, params) {
  var qs = Object.keys(params).map(function(k) {
    return k + '=' + encodeURIComponent(params[k]);
  }).join('&');
  // remove any trailing ? or &
  uri = uri.replace(/[?&]$/, '');
  // append ? or & depending on whether uri has existing parameters
  uri = uri.indexOf('?') === -1 ? uri + '?' : uri + '&';
  return uri + qs;
};
```