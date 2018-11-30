---
title: ECharts 3.0底层zrender 3.x源码分析2-Painter（V层）
tags:    
- FE
- ECharts
- zrender
- 源码   
categories: 前端技术
date: 2017-01-11 00:00:00
---
上一篇介绍了zrender的总体结构，这一篇我们就详细介绍View层--Painter(Painter.js)。

一些demo和没有在博客中介绍的源码请进我的[github仓库](https://github.com/zrysmt/echarts3/tree/master/zrender)。
>https://github.com/zrysmt/echarts3/tree/master/zrender

Painter利用canvas负责真正的绘图操作。
*   1.负责canvas及其周边DOM元素的创建与处理
*   2.负责调用各个Shape（预定义好的）进行绘制
*   3.提供基本的操作方法，渲染(render)、刷新(refresh)、尺寸变化(resize)、擦除(clear)等

# 1.渲染结构分析

两个例子都是渲染到div上。

```html
<div id="main" style="width:1000px;height:600px;margin:0;"></div>
```

zrender 3.x版本渲染结果（demo/demo1/demo3-chart.html）
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/zrender2-1.jpg)
我们可以看到渲染结果都会新建一层div（从下面的分析我们可以得到这个div就是`_domRoot`），里面嵌套canvas。如果有使用addHover（有hover层,data-zr-dom-id="zr_100000"）的话，hover层会单独列一个canvas画布。
```javascript
sector.on('mouseover', function() {
    zr.addHover(this, {
        stroke: 'yellow',
        lineWidth: 10,
        opacity: 1
    });
    zr.refresh();
});
sector.on('mouseout', function() {
    zr.removeHover(this);
});
```
```html
<div id="main" style="width: 1000px; height: 600px; margin: 0px; -webkit-tap-highlight-color: transparent; user-select: none;">
    <div style="position: relative; overflow: hidden; width: 1000px; height: 600px; padding: 0px; margin: 0px; border-width: 0px; cursor: default;">
        <canvas width="1000" height="600" data-zr-dom-id="zr_0" style="position: absolute; left: 0px; top: 0px; width: 1000px; height: 600px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); padding: 0px; margin: 0px; border-width: 0px;"></canvas>
        <canvas width="1000" height="600" data-zr-dom-id="zr_100000" style="position: absolute; left: 0px; top: 0px; width: 1000px; height: 600px; user-select: none; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); padding: 0px; margin: 0px; border-width: 0px;"></canvas>
    </div>
</div>
```
# 2.构造函数

```javascript
var Painter = function (root, storage, opts) {
        // In node environment using node-canvas
        var singleCanvas = !root.nodeName // In node ?
            || root.nodeName.toUpperCase() === 'CANVAS';
        this._opts = opts = util.extend({}, opts || {});
        this.dpr = opts.devicePixelRatio || config.devicePixelRatio;
        this._singleCanvas = singleCanvas;
        /**
         * 绘图容器
         * @type {HTMLElement}
         */
        this.root = root;
        var rootStyle = root.style;
        if (rootStyle) {
            rootStyle['-webkit-tap-highlight-color'] = 'transparent';
            rootStyle['-webkit-user-select'] =
            rootStyle['user-select'] =
            rootStyle['-webkit-touch-callout'] = 'none';
            root.innerHTML = '';
        }
        /**
         * @type {module:zrender/Storage}
         */
        this.storage = storage;
        /**
         * 存储图层画布，这个变量很重要
         * @type {Array.<number>}
         * @private
         */
        var zlevelList = this._zlevelList = [];

        /** 图层
         * @type {Object.<string, module:zrender/Layer>}
         * @private
         */
        var layers = this._layers = {};
        this._layerConfig = {};

        if (!singleCanvas) {//没有画布，就使用div
            this._width = this._getSize(0);
            this._height = this._getSize(1);

            var domRoot = this._domRoot = createRoot(
                this._width, this._height
            );
            root.appendChild(domRoot);
        }
        else {//已经有块画布
            // Use canvas width and height directly
            var width = root.width;
            var height = root.height;
            this._width = width;
            this._height = height;

            // Create layer if only one given canvas
            // dpr设置为1，是因为canvas已经定了宽和高
            var mainLayer = new Layer(root, this, 1);
            mainLayer.initContext();
            // FIXME Use canvas width and height
            // mainLayer.resize(width, height);
            layers[0] = mainLayer;
            zlevelList.push(0);
        }

        this.pathToImage = this._createPathToImage();
        // Layers for progressive rendering
        this._progressiveLayers = [];
        this._hoverlayer;
        this._hoverElements = [];
    };
```
# 3.Painter.prototype
```javascript
Painter.prototype = {
    constructor: Painter,
    isSingleCanvas: function() {},
    getViewportRoot: function() {},
    refresh: function(paintAll) {},
    addHover: function(el, hoverStyle) {},
    removeHover: function(el) {},
    clearHover: function(el) {},
    refreshHover: function() {},
    _startProgessive: function() {},
    _clearProgressive: function() {},
    _paintList: function(list, paintAll) {},
    _doPaintList: function(list, paintAll) {},
    _doPaintEl: function(el, currentLayer, forcePaint, scope) {},
    getLayer: function(zlevel) {},
    insertLayer: function(zlevel, layer) {},
    eachLayer: function(cb, context) {},
    eachBuildinLayer: function(cb, context) {},
    eachOtherLayer: function(cb, context) {},
    getLayers: function() {},
    _updateLayerStatus: function(list) {},
    clear: function() {},
    _clearLayer: function(layer) {},
    configLayer: function(zlevel, config) {},
    delLayer: function(zlevel) {},
    resize: function(width, height) {},
    clearLayer: function(zlevel) {},
    dispose: function() {},
    getRenderedCanvas: function(opts) {},
    getWidth: function() {},
    getHeight: function() {},
    _getSize: function(whIdx) {},
    _pathToImage: function(id, path, width, height, dpr) {},
    _createPathToImage: function() {}
}
```
**我们再来回顾下整个渲染的过程：**
`add`（zrender.js）-->`addRoot`(Storage.js) --> `addToMap`(Storage.js) --> 
`dirty`[标记为脏的，下一帧渲染] (path.js) --> `refresh`(Painter.js)-->`_paintList`[遍历_displayList] (Painter.js)-->
`_doPaintEl`[渲染单个元素] Painter.js) -->`brush`(Path.js)-->`buildPath` (各个类型的shape)

- `refresh`刷新，刷新去绘制

```javascript
/**
* 刷新
* @param {boolean} [paintAll=false] 强制绘制所有displayable
*/
refresh: function(paintAll) {
    var list = this.storage.getDisplayList(true); //要绘制的图形
    var zlevelList = this._zlevelList;
    this._paintList(list, paintAll); //去绘制
    // Paint custum layers 绘制layer层
    for (var i = 0; i < zlevelList.length; i++) {
        var z = zlevelList[i];
        var layer = this._layers[z];
        if (!layer.isBuildin && layer.refresh) {
            layer.refresh();
        }
    }
    this.refreshHover(); //刷新hover层
    if (this._progressiveLayers.length) {
        this._startProgessive();
    }
    return this;
}
```
- `_paintList`

```javascript
_paintList: function(list, paintAll) {
    if (paintAll == null) {
        paintAll = false;
    }
    this._updateLayerStatus(list);
    this._clearProgressive();
    this.eachBuildinLayer(preProcessLayer);
    this._doPaintList(list, paintAll); //全部标注为脏的渲染【dirty(false)】
    this.eachBuildinLayer(postProcessLayer);
}
```
- `_doPaintList`

注意这里已经遍历了（遍历的是_displayList数组），所以后面的只针对单个元素绘制即可。
```javascript
//... ...
for (var i = 0, l = list.length; i < l; i++) {
   //... ...
   this._doPaintEl(el, currentLayer, paintAll, scope);//绘制每个元素
}
```
- `_doPaintEl`

```javascript
//... ...
el.brush(ctx, scope.prevEl || null);//在Path.js中的方法brush
```

# 4.分析Painter对象

![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/zrender2-2.jpg)

这一系列的操作是：

- 创建canvas外层包裹着_domRoot(div)
- canvas要绘制的东西都存储在storage中的_displayList数组中
- 遍历`_displayList`
- 最后调用buildPath的canvas绘制。

# 5.Hover图层

如第1部分所见，如果增加了hover层（addHOver方法），那么会增加一层canvas，现在就来看这一层canvas是如何作用的。
`addHover`(zrender.js)-->

- `addHover`(zrender.js)

```javascript
addHover: function(el, style) {
    if (this.painter.addHover) {
        this.painter.addHover(el, style);
        this.refreshHover();
    }
}
```
- `addHover`(Painter.js)

```javascript
addHover: function(el, hoverStyle) {
    if (el.__hoverMir) {
        return;
    }
    var elMirror = new el.constructor({
        style: el.style,
        shape: el.shape
    });
    elMirror.__from = el;
    el.__hoverMir = elMirror;
    elMirror.setStyle(hoverStyle);
    this._hoverElements.push(elMirror);
    //存放到this._hoverElements(数组)
}
```
我们在第三部分已经看到`refresh`方法中的`refreshHover`，渲染canvas时候，会渲染两个canvas，一个是主canvas，一个是hover层canvas，第二个canvas就是使用`refreshHover`方法。

- `refreshHover`(Painter.js)

```javascript
refreshHover: function() {
    var hoverElements = this._hoverElements;
    var len = hoverElements.length;
    var hoverLayer = this._hoverlayer;
    hoverLayer && hoverLayer.clear();
    if (!len) {
        return;
    }
    timsort(hoverElements, this.storage.displayableSortFunc);
    if (!hoverLayer) {//不存在则会新创建一层canvas
        hoverLayer = this._hoverlayer = this.getLayer(1e5);
    }

    var scope = {};
    hoverLayer.ctx.save();
    for (var i = 0; i < len;) {
        var el = hoverElements[i];
        var originalEl = el.__from;
        if (!(originalEl && originalEl.__zr)) {
            hoverElements.splice(i, 1);
            originalEl.__hoverMir = null;
            len--;
            continue;
        }
        i++;
        if (!originalEl.invisible) {
            el.transform = originalEl.transform;
            el.invTransform = originalEl.invTransform;
            el.__clipPaths = originalEl.__clipPaths;
            // el.
            this._doPaintEl(el, hoverLayer, true, scope);//同第3部分
        }
    }
    hoverLayer.ctx.restore();
}
```


**参考阅读：**
- [canvas-mdn教程](- [canvas-mdn教程]（https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial
)
- [canvas基本的动画-mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_animations)