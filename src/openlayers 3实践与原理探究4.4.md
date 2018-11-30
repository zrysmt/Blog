---
title: OpenLayers 3实践与原理探究4.4-ol3源码分析-render
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---
前面几节的内容介绍了`Map`,`View`,`Source`,`Layer`,这些其实我们都是要么在对象属性中设置 ，要么是通过方法设置，实质上是通过共享的全局变量设置地图包含的图层，地图的显示效果，但是如果真正上绘制在浏览器上，需要渲染在canvas(ol3常用的渲染方式).

由于源码代码量比较大，这里只是从大部分介绍流程。
网上有人(OpenLayers 3源码那些事)总结一张图的不错，这里拿来用一下
![ol3渲染流程](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol3%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B.png)
# 0.从`map.js`开始
- **1) render/renderSync**
具体流程用注释的形式标出

```javascript
//renderSync是异步的，同样道理
ol.Map.prototype.render = function() {
  if (this.animationDelayKey_ === undefined) {
    this.animationDelayKey_ = ol.global.requestAnimationFrame(
        this.animationDelay_);
  }
};
```
- **2) animationDelay_**

```javascript
this.animationDelay_ = function() {
    this.animationDelayKey_ = undefined;
    this.renderFrame_.call(this, Date.now());
  }.bind(this);
```
- **3) renderFrame_**

```javascript
ol.Map.prototype.renderFrame_ = function(time) {

  var i, ii, viewState;

  var size = this.getSize();
  var view = this.getView();
  var extent = ol.extent.createEmpty();
  /** @type {?olx.FrameState} */
  var frameState = null;
  if (size !== undefined && ol.size.hasArea(size) && view && view.isDef()) {
    var viewHints = view.getHints(this.frameState_ ? this.frameState_.viewHints : undefined);
    var layerStatesArray = this.getLayerGroup().getLayerStatesArray();
    var layerStates = {};
    for (i = 0, ii = layerStatesArray.length; i < ii; ++i) {
      layerStates[goog.getUid(layerStatesArray[i].layer)] = layerStatesArray[i];
    }
    viewState = view.getState();
    frameState = /** @type {olx.FrameState} */ ({      //1.准备frameState
      animate: false,
      attributions: {},
      coordinateToPixelMatrix: this.coordinateToPixelMatrix_,
      extent: extent,
      focus: !this.focus_ ? viewState.center : this.focus_,
      index: this.frameIndex_++,
      layerStates: layerStates,//layer的常量属性，通过共享作为全局变量
      layerStatesArray: layerStatesArray,
      logos: ol.object.assign({}, this.logos_),
      pixelRatio: this.pixelRatio_,
      pixelToCoordinateMatrix: this.pixelToCoordinateMatrix_,
      postRenderFunctions: [],
      size: size,
      skippedFeatureUids: this.skippedFeatureUids_,
      tileQueue: this.tileQueue_,
      time: time,
      usedTiles: {},
      viewState: viewState,//view的常量属性，通过共享作为全局变量
      viewHints: viewHints,
      wantedTiles: {}
    });
  }

  if (frameState) {
    var preRenderFunctions = this.preRenderFunctions_;//2.渲染前
    var n = 0, preRenderFunction;
    for (i = 0, ii = preRenderFunctions.length; i < ii; ++i) {
      preRenderFunction = preRenderFunctions[i];
      if (preRenderFunction(this, frameState)) {
        preRenderFunctions[n++] = preRenderFunction;
      }
    }
    preRenderFunctions.length = n;

    frameState.extent = ol.extent.getForViewAndSize(viewState.center,
        viewState.resolution, viewState.rotation, frameState.size, extent);
  }

  this.frameState_ = frameState;
  this.renderer_.renderFrame(frameState);                //3.渲染

  if (frameState) {
    if (frameState.animate) {
      this.render();
    }
    Array.prototype.push.apply(
        this.postRenderFunctions_, frameState.postRenderFunctions);//4.渲染后

    var idle = this.preRenderFunctions_.length === 0 &&
        !frameState.viewHints[ol.ViewHint.ANIMATING] &&
        !frameState.viewHints[ol.ViewHint.INTERACTING] &&
        !ol.extent.equals(frameState.extent, this.previousExtent_);

    if (idle) {
      this.dispatchEvent(
          new ol.MapEvent(ol.MapEventType.MOVEEND, this, frameState));
      ol.extent.clone(frameState.extent, this.previousExtent_);
    }
  }

  this.dispatchEvent(
      new ol.MapEvent(ol.MapEventType.POSTRENDER, this, frameState));

  goog.async.nextTick(this.handlePostRender, this);

};
```

**下面讲诉的渲染的第3步 `renderFrame(frameState)`**
有关渲染的源代码在`ol/ol/render`,`ol/ol/renderer`下。
`ol/ol/render`文件夹下是渲染的基本属性和方法，利用设计模式中的工厂模式，用来构造`ol/ol/renderer`。
# 1.渲染Map
`ol/ol/renderer/maprenderer.js`

```javascript
ol.renderer.Map = function(container, map) {}
```
这只是个父类，具体实现类位置在：
`ol/ol/renderer/canvas/canvasmaprenderer.js`--ol.render.canvas.Map(默认)
`ol/ol/renderer/canvas/webglmaprenderer.js`--ol.render.webgl.Map
`ol/ol/renderer/canvas/dommaprenderer.js`--ol.render.dom.Map

主要介绍第一种`ol/ol/renderer/canvas/canvasmaprenderer.js`

```javascript
ol.renderer.canvas.Map = function(container, map) {
	ol.renderer.Map.call(this, container, map);
	this.context_ = ol.dom.createCanvasContext2D();

	  this.canvas_ = this.context_.canvas;

	  this.canvas_.style.width = '100%';
	  this.canvas_.style.height = '100%';
	  this.canvas_.className = ol.css.CLASS_UNSELECTABLE;
	  container.insertBefore(this.canvas_, container.childNodes[0] || null);
}
```
渲染逻辑
```javascript
ol.renderer.canvas.Map.prototype.renderFrame = function(frameState) {}
```
# 2.渲染Layer
`ol/ol/renderer/layerrenderer.js`

```javascript
ol.renderer.Layer = function(layer) {

  ol.Observable.call(this);

  this.layer_ = layer;
};
```
这只是个父类，具体实现类位置在：
`ol/ol/renderer/canvas/canvaslayerrenderer.js`--ol.render.canvas.Layer(默认)
`ol/ol/renderer/canvas/webgllayerrenderer.js`--ol.render.webgl.Layer
`ol/ol/renderer/canvas/domlayerrenderer.js`--ol.render.dom.Layer

主要介绍第一种ol.render.canvas.Layer(默认)：
这个类又分为三种类型：
- ol.render.canvas.TileLayer
- ol.render.canvas.VectorLayer
- ol.render.canvas.VectorTileLayer
 

`ol.render.canvas.TileLayer`渲染逻辑

```javascript
ol.renderer.canvas.TileLayer.prototype.prepareFrame = function(
    frameState, layerState) {}    //第一步
ol.renderer.canvas.TileLayer.prototype.composeFrame = function(//第二步
    frameState, layerState, context) {
  var transform = this.getTransform(frameState, 0);
  this.dispatchPreComposeEvent(context, frameState, transform);
  this.renderTileImages(context, frameState, layerState);
  this.dispatchPostComposeEvent(context, frameState, transform);
};

ol.renderer.canvas.TileLayer.prototype.renderTileImages = function(context, frameState, layerState) {}
```


参考文献：
- [OpenLayers 3源码解析视频](http://www.jianshu.com/p/573cde18575a)
- [OpenLayers 3源码那些事(上)](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=4&ved=0ahUKEwjV852YrvDNAhVEQo8KHbbAAzwQFgg0MAM&url=%68%74%74%70%3a%2f%2f%77%65%69%6c%69%6e%2e%6d%65%2f%64%6f%63%2f%4f%70%65%6e%4c%61%79%65%72%73%25%32%30%33%25%45%36%25%42%41%25%39%30%25%45%37%25%41%30%25%38%31%25%45%39%25%38%32%25%41%33%25%45%34%25%42%41%25%39%42%25%45%34%25%42%41%25%38%42%28%25%45%34%25%42%38%25%38%41%29%2e%64%6f%63%78&usg=AFQjCNHRsX5ZW6MBpwJUCSEZgOuR7NL4Rw)
- [OpenLayers 3源码那些事(下)](http://weilin.me/ol3-shares/source-explain-2/)