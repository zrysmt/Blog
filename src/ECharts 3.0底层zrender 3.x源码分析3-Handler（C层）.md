---
title: ECharts 3.0底层zrender 3.x源码分析3-Handler（C层）
tags:    
- FE
- ECharts
- zrender
- 源码   
categories: 前端技术
date: 2017-01-11 00:00:00
---
这一篇，介绍下Handler处理机制。

Handler负责事件处理,包括'click', 'dblclick', 'mousewheel', 'mouseout',        'mouseup', 'mousedown', 'mousemove', 'contextmenu'等。我们知道canvas API没有提供监听每个元素的机制，这就需要一些处理。处理的思路是：监听事件的作用坐标（如点击时候的坐标），判断在哪个绘制元素的范围中，如果在某个元素中，这个元素就监听该事件。

一些demo和没有在博客中介绍的源码请进我的[github仓库](https://github.com/zrysmt/echarts3/tree/master/zrender)。
>https://github.com/zrysmt/echarts3/tree/master/zrender

# 1.Handle.js整体
同样Handle.js文件的结构是一个构造函数，一个prototype扩展原型，一些混入模式。

我们首先看在入口（zrender.js）中的调用
```javascript
var handerProxy = !env.node ? new HandlerProxy(painter.getViewportRoot()) : null;//env.node默认为false
//HandlerProxy 是移动端的一些处理事件
this.handler = new Handler(storage, painter, handerProxy, painter.root);
```
**构造函数：**
```javascript
var Handler = function(storage, painter, proxy, painterRoot) {
        Eventful.call(this);
        this.storage = storage;
        this.painter = painter;
        this.painterRoot = painterRoot;
        proxy = proxy || new EmptyProxy();
        /**
         * Proxy of event. can be Dom, WebGLSurface, etc.
         */
        this.proxy = proxy;
        // Attach handler
        proxy.handler = this;
        this._hovered;
        /**
         * @private
         * @type {Date}
         */
        this._lastTouchMoment;
        this._lastX;//坐标位置x
        this._lastY;//坐标位置y

        Draggable.call(this);
        util.each(handlerNames, function (name) {
            proxy.on && proxy.on(name, this[name], this);
        }, this);
    };
```
构造函数中保留的有坐标信息。

prototype中的一个重要的方法`dispatchToElement`,针对目标图形元素触发事件。
```javascript
/**
 * 事件分发代理
 *
 * @private
 * @param {Object} targetEl 目标图形元素
 * @param {string} eventName 事件名称
 * @param {Object} event 事件对象
 */
dispatchToElement: function(targetEl, eventName, event) {
    var eventHandler = 'on' + eventName;
    var eventPacket = makeEventPacket(eventName, targetEl, event);
    var el = targetEl;
    while (el) {
        el[eventHandler] && (eventPacket.cancelBubble = el[eventHandler].call(el, eventPacket));
        el.trigger(eventName, eventPacket);//触发
        el = el.parent;
        if (eventPacket.cancelBubble) {
            break;
        }
    }
    if (!eventPacket.cancelBubble) {
        // 冒泡到顶级 zrender 对象
        this.trigger(eventName, eventPacket);
        // 分发事件到用户自定义层
        // 用户有可能在全局 click 事件中 dispose，所以需要判断下 painter 是否存在
        this.painter && this.painter.eachOtherLayer(function(layer) {
            if (typeof(layer[eventHandler]) == 'function') {
                layer[eventHandler].call(layer, eventPacket);
            }
            if (layer.trigger) {
                layer.trigger(eventName, eventPacket);//触发
            }
        });
    }
}
```

混入Eventful(发布订阅模式事件)、Draggable（拖动事件）
```javascript
util.mixin(Handler, Eventful);
util.mixin(Handler, Draggable);
```
# 2.canvas上元素的监听事件

对于一些事件的处理（Handler.js）
```javascript
 util.each(['click', 'mousedown', 'mouseup', 'mousewheel', 'dblclick', 'contextmenu'], function (name) {
        Handler.prototype[name] = function (event) {
            // Find hover again to avoid click event is dispatched manually. Or click is triggered without mouseover
            var hovered = this.findHover(event.zrX, event.zrY, null);
            if (name === 'mousedown') {
                this._downel = hovered;
                // In case click triggered before mouseup
                this._upel = hovered;
            }
            else if (name === 'mosueup') {
                this._upel = hovered;
            }
            else if (name === 'click') {
                if (this._downel !== this._upel) {
                    return;
                }
            }
            
            console.info("hovered:",hovered);
            console.info(this);
            this.dispatchToElement(hovered, name, event);
        };
    });
```
我们在其中打印了this，通过demo/demo1/demo3-chartHasHover.html的例子我们可以发现，点击的时候都会打印this，而且打印3次。

通过打印的hovered，我们可以看出来hovered就是我们点击的对象。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/zrender3-1.jpg)

`findHover`调用的是`isHover`函数，在`isHover`函数中通过`displayable`（Displayable.js）的`contain`或者`rectContain`判断点在哪个元素中。

```javascript
function isHover(displayable, x, y) {
    if (displayable[displayable.rectHover ? 'rectContain' : 'contain'](x, y)) {
        var el = displayable;
        while (el) {
            // If ancestor is silent or clipped by ancestor
            if (el.silent || (el.clipPath && !el.clipPath.contain(x, y))) {
                return false;
            }
            el = el.parent;
        }
        return true;
    }
    return false;
}
```
Displayable.js的`contain`或者`rectContain`方法都是调用`rectContain`方法，判断x,y是否在图形的包围盒上。
```javascript
rectContain: function(x, y) {
    var coord = this.transformCoordToLocal(x, y);
    var rect = this.getBoundingRect();//@module zrender/core/BoundingRect
    return rect.contain(coord[0], coord[1]);
}
```
zrender/core/BoundingRect的`contain`方法
```javascript
 contain: function(x, y) {
     var rect = this;
     return x >= rect.x && x <= (rect.x + rect.width) && 
     y >= rect.y && y <= (rect.y + rect.height);
 }
```
我们再来看看，在painter.js中，其实已经为每个元素生成了它的包围盒上。
```javascript
 var tmpRect = new BoundingRect(0, 0, 0, 0);
 var viewRect = new BoundingRect(0, 0, 0, 0);

 function isDisplayableCulled(el, width, height) {
     tmpRect.copy(el.getBoundingRect());
     if (el.transform) {
         tmpRect.applyTransform(el.transform);
     }
     viewRect.width = width;
     viewRect.height = height;
     return !tmpRect.intersect(viewRect);
 }
```
在绘制每个元素的时候,在`_doPaintEl`方法中调用了`isDisplayableCulled`。



**参考阅读：**
- [canvas-mdn教程](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial)
- [canvas基本的动画-mdn](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_animations)

