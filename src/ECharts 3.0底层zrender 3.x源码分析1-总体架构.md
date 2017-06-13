---
title: ECharts 3.0底层zrender 3.x源码分析1-总体架构
tags:    
- FE
- ECharts
- zrender
- 源码   
categories: 前端技术
---

zrender是一个轻量级的Canvas类库，作为百度[Echarts 3.0](http://echarts.baidu.com/index.html)的底层基础。截至目前查看的zrender源码和文档，包括官网文档都还停留在2.x时代，我打算用一个系列介绍下zrender 3.x的使用和源码，一些demo和没有在博客中介绍的源码请进我的[github仓库](https://github.com/zrysmt/echarts3/tree/master/zrender)。
>https://github.com/zrysmt/echarts3/tree/master/zrender

基于版本 3.2.2。

# 1.总体架构
官网上的一张图和解释。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/zrender1.jpg)
**MVC结构**分别在Stroage.js,Painter.js,Handler.js文件下，我们稍后会详细解释，现在我们大概来看下它们分别的作用。
*   Stroage(M) : shape数据CURD管理
*   Painter(V) : canvase元素生命周期管理，视图渲染，绘画，更新控制
*   Handler(C) : 事件交互处理，实现完整dom事件模拟封装
*   shape : 图形实体，分而治之的图形策略，可定义扩展
*   tool : 绘画扩展相关实用方法，工具及脚手架
*   animation : 动画扩展，提供promise式的动画接口和常用缓动函数

**源码结构**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/zrender2.jpg)
目录的介绍
- animation 动画有关；
- contain 包含判断；
- container Group.js 元素组的概念；
- core 核心代码，包含一些工具（util.js）、事件（event.js）、唯一ID(guid.js)、矩阵运算有关（matrix.js）等；
- dom HandleProxy.js dom事件有关；
- graphic 图形有关，shape文件夹下就是各个图形的js文件；
- mixin 混入模式要混入的函数;
- tool 工具函数，包括颜色工具(color.js)，path工具(path.js)和转换工具（transformPath.js）;
- vml IE中的画笔，[vml解释进入](http://www.g168.net/txt/vml/]
- 全局的文件 
  + config.js    配置文件
  + Element.js   元素文件作为zrender最基本的元素
  + Handle.js    C层，控制层
  + Layer.js     图层管理
  + Painter.js   V层，视图层
  + Storage.js   M层，数据管理层
  + zrender.js   **入口**

# 2.入口（zrender.js）
## 2.1 初始化
类似于jquery的无new化处理，init调用即可
调用：
```javascript
var zr = zrender.init(document.getElementById('main'));
```
源码：
```javascript
    var instances = {};    // ZRender实例map索引
    var zrender = {};
    zrender.init = function(dom, opts) {
        var zr = new ZRender(guid(), dom, opts);
        instances[zr.id] = zr;
        return zr;
    };
    
```
## 2.2 构造函数
我们可以在构造函数中，看到MVC的管理机制。
```javascript
var ZRender = function(id, dom, opts) {
    opts = opts || {};
    this.dom = dom;
    this.id = id;
    var self = this;
    var storage = new Storage();
    var rendererType = opts.renderer;
    if (useVML) {//IE中使用VML渲染
        if (!painterCtors.vml) {
            throw new Error('You need to require \'zrender/vml/vml\' to support IE8');
        }
        rendererType = 'vml';
    } else if (!rendererType || !painterCtors[rendererType]) {
        rendererType = 'canvas';
    }
    var painter = new painterCtors[rendererType](dom, storage, opts);

    this.storage = storage;//M
    this.painter = painter;//V

    var handerProxy = !env.node ? new HandlerProxy(painter.getViewportRoot()) : null;
    this.handler = new Handler(storage, painter, handerProxy, painter.root);//C

    console.log(this);//这里是我增加的为了调试使用的
    /**
     * @type {module:zrender/animation/Animation}
     * 动画控制
     */
    this.animation = new Animation({
        stage: {
            update: zrUtil.bind(this.flush, this)
        }
    });
    this.animation.start();
    this._needsRefresh;

    // 修改 storage.delFromMap, 每次删除元素之前删除动画
    var oldDelFromMap = storage.delFromMap;
    var oldAddToMap = storage.addToMap;
    storage.delFromMap = function(elId) {
        var el = storage.get(elId);
        oldDelFromMap.call(storage, elId);
        el && el.removeSelfFromZr(self);
    };
    storage.addToMap = function(el) {
        oldAddToMap.call(storage, el);
        el.addSelfToZr(self);
    };
};
```

## 2.3 ZRender.prototype

具体的方法及其注释可以在我的github中查看，这里只将方法名放在这里。
```javascript
ZRender.prototype = {
        constructor: ZRender,
        /**
         * 获取实例唯一标识
         * @return {string}
         */
        getId: function () {},
        /**
         * 添加元素后就会渲染
         * @param  {module:zrender/Element} el
         */
        add: function (el) {
            this.storage.addRoot(el);
            this._needsRefresh = true;
        },
        /**
         * 删除元素
         * @param  {module:zrender/Element} el
         */
        remove: function (el) { },
        configLayer: function (zLevel, config) {},
        /** Repaint the canvas immediately*/
        refreshImmediately: function () {},
        /** Mark and repaint the canvas in the next frame of browser*/
        refresh: function() {},
        flush: function () {},

        /**Add element to hover layer */
        addHover: function (el, style) {},
        /** Add element from hover layer
         * @param  {module:zrender/Element} el
         */
        removeHover: function (el) {},

        /** Clear all hover elements in hover layer*/
        clearHover: function () {},
        /** Refresh hover in next frame*/
        refreshHover: function () {},
        /**Refresh hover immediately*/
        refreshHoverImmediately: function () {     ;
        },
        resize: function(opts) {},
        clearAnimation: function () {},
        /** Get container width */
        getWidth: function() {},
        getHeight: function() {},
        /** Converting a path to image */
        pathToImage: function(e, width, height) {},
        /**
         * Set default cursor
         * @param {string} [cursorStyle='default'] 例如 crosshair
         */
        setCursorStyle: function (cursorStyle) {},
        /**发布订阅模式 */
        on: function(eventName, eventHandler, context) {},
        off: function(eventName, eventHandler) {},
        trigger: function (eventName, event) {},
        /** Clear all objects and the canvas */
        clear: function () {},
        /** Dispose self */
        dispose: function () {}
    };
```
源码的方法，我们以`add`举例子，它其实调用的是`this.storage.addRoot`方法,使用MVC机制处理。
使用示例：
```javascript
var circle1 = new Circle({
    shape: {
        cx: 100,
        cy: 100,
        r: 30
    },
    style: {
        fill: 'blue'
    },
    draggable: true
});
zr.add(circle1);
circle1.on('mouseover', function() {
    zr.addHover(this, {
        stroke: 'yellow',
        lineWidth: 10,
        opacity: 1
    });
    zr.refresh();
});
circle1.on('mouseout', function() {
    zr.removeHover(this);
});
```
注意：这里有`addHover`方法，所以会渲染两个canvas。如果没有addHover，就只会渲染一个canvas。
# 3.MVC简单概述
MVC对应三个文件的结构很简单，其实就是一个构造函数，一个prototype原型扩展。
## 3.1 M--数据管理层(Storage.js）
我们看构造函数，将元素存储在this._elements(对象)、this._roots（数组）和this._displayList（数组）中，然后负责在其中进行增（addRoot，addToMap）删(delRoot,delFromMap)改（updateDisplayList）查（get，getDisplayList）。
```javascript
   var Storage = function () {
        // 所有常规形状，id索引的map
        this._elements = {};
        //和this._elements存放的元素一样，只不过是数组
        this._roots = [];
        //和this.roots一样
        this._displayList = [];
        //this._displayList的长度
        this._displayListLen = 0;
    };
```
## 3.2 C--控制层（Handle.js）
Handler负责事件处理,包括'click', 'dblclick', 'mousewheel', 'mouseout',        'mouseup', 'mousedown', 'mousemove', 'contextmenu'等。我们知道canvas API没有提供监听每个元素的机制，这就需要一些处理。处理的思路是：监听事件的作用坐标（如点击时候的坐标），判断在哪个绘制元素的范围中，如果在某个元素中，这个元素就监听该事件。具体的思路可以查看参考阅读给的链接文章。

```javascript
    Handler.prototype = {
        mousemove：function (event){}//... ...
    }
    util.mixin(Handler, Eventful);//混入，下面我们会解释到
    util.mixin(Handler, Draggable);
```
## 3.3 V--视图层（Painter.js）

Painter负责真正的绘图操作，这里是比较繁重的部分
*   1.负责canvas及其周边DOM元素的创建与处理
*   2.负责调用各个Shape（预定义好的）进行绘制
*   3.提供基本的操作方法，渲染(render)、刷新(refresh)、尺寸变化(resize)、擦除(clear)等

Painter是调用canvas API实现的绘制,包括颜色，渐变色，变换，矩阵变化，绘制图片、文本等。IE8使用[excanvas](https://code.google.com/p/explorercanvas/)兼容。
# 4.设计模式总结
设计模式的总结，我在一篇[博客](http://blog.csdn.net/future_todo/article/details/53992141)中有写,要想看这方面的知识，可以在这里看。
## 4.1 AMD模式
AMD即是“异步模块定义”的意思，所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。源码的结构是这样的
- 定义

```javascript
define(function(require) {
    return ZRender;
}
```
- 使用

```javascript
require([module], callback);
```
我们的Demo使用的是百度封装好的AMD模式esl.js(或者使用requirejs也可以),引入方式和使用示例如下：

```html
<script src="../libs/esl.js"></script>
```
```javascript
require(['zrender', 'zrender/graphic/shape/Circle', 'zrender/graphic/shape/Polygon'],
function(zrender, Circle, Polygon) { //... ...
});
```
## 4.2 继承
在core->util.js，主要的思想就是将子类的prototype指向父类的prototype；子类的构造函数指向自己。
```javascript
   function inherits(clazz, baseClazz) {
       var clazzPrototype = clazz.prototype;
       function F() {}
       F.prototype = baseClazz.prototype;
       clazz.prototype = new F();
       for (var prop in clazzPrototype) {//属性也继承了
           clazz.prototype[prop] = clazzPrototype[prop];
       }
       clazz.prototype.constructor = clazz;
       clazz.superClass = baseClazz;//superClass是个自己定义的属性
   }
```
另外不要忘了，在构造函数中应该重写父类的属性。例如：Displayable的父类是Element:

```javascript
function Displayable(opts) {
     Element.call(this, opts);
}
```
实现继承：
```javascript
zrUtil.inherits(Displayable, Element);
```
## 4.3 混入模式
简而言之，混入就是将一个对象的方法复制给另外一个对象。实现在util.js中
```javascript
function mixin(target, source, overlay) {
   target = 'prototype' in target ? target.prototype : target;
   source = 'prototype' in source ? source.prototype : source;
   defaults(target, source, overlay);
}
function defaults(target, source, overlay) {
    for (var key in source) {
        if (source.hasOwnProperty(key) && (overlay ? source[key] != null : target[key] == null)) {
            target[key] = source[key];
        }
    }
    return target;
}
```
调用
```javascript
zrUtil.mixin(Displayable, RectText);
```
## 4.4 jquery的extend模式
实现很简单，类似混入模式,将source对象的方法复制给target对象。
```javascript
function extend(target, source) {
     for (var key in source) {
         if (source.hasOwnProperty(key)) {
             target[key] = source[key];
         }
     }
     return target;
 }
```
## 4.5 发布订阅模式
逻辑在mixin文件夹中的Eventful.js，为Handle(handle.js)混入方法
```javascript
util.mixin(Handler, Eventful);
```
包括一下几种方法
- `one`一次绑定事件
- `on` 绑定事件
- `isSilent`是否绑定了事件
- `off`解绑事件
- `trigger`事件分发，触发事件
- `triggerWithContext`带有context的事件分发

# 5.逻辑关系

- 步进关系

说明：-->为扩展或混入，==>为继承自父类，（）内部为所在位置, [ ]为扩展或者混入的方式。

`Element`[Animatable Transformable Eventful] (Element.js) ==> 
`Displayable`[ReactText] (Displayable.js) ==> 
`Path`[Sub] (Path.js) ==>
`Sub`(Path.js) -->
`各类型的shape`

底层对象是封装过的Element。

- 绘制的逻辑

`add`（zrender.js）-->`addRoot`(Storage.js) --> `addToMap`(Storage.js) --> 
`dirty`[标记为脏的，下一帧渲染] (path.js) --> `refresh`(Painter.js)-->`_paintList`[遍历_displayList] (Painter.js)-->
`_doPaintEl`[渲染单个元素] Painter.js) -->`brush`(Path.js)-->`buildPath` (各个类型的shape)


**参考阅读：**
- [zrender官方网站](http://ecomfe.github.io/zrender/)
- [vml解释](http://www.g168.net/txt/vml/],相当于IE里面的画笔，能实现你所想要的图形，而且结合脚本，可以让图形产生动态的效)
- [ZRender源码分析系列](http://www.cnblogs.com/hhstuhacker/category/603743.html)
- [ZRender源码分析系列源码注释](https://github.com/lonelyclick/nts/tree/master/zrender/src)
- [HTML5 Canvas绘制的图形的事件处理](http://blog.csdn.net/vuturn/article/details/45822905)