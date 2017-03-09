---
title: ECharts 3.0源码简要分析1-总体架构
tags:    
- FE
- ECharts
- zrender
- 源码   
categories: 前端技术
---


百度的[Echarts 3.0](http://echarts.baidu.com/index.html)作为前端领域可视化重要的开源库，是我们在日常工作生活中经常使用的，所以有必要一起来了解下Echarts的源码。我打算用一个系列介绍下Echarts 3.x的使用和源码，一些demo和没有在博客中介绍的源码请进我的[github仓库](https://github.com/zrysmt/echarts3/tree/master/echarts)。
>https://github.com/zrysmt/echarts3/tree/master/echarts

本博文Echarts版本基于3.3.2。

Echarts的源码是在zrender的基础上封装的，所以要看明白echarts源码须要先了解zrender的源码，不过为了本博文的独立可读性，这里也会将用到的zrender源码简单说明。如果要了解zrender具体的源码，这里给出了zrender源码解读博客和源码注释仓库。
> [github仓库:`https://github.com/zrysmt/echarts3/tree/master/zrender`](https://github.com/zrysmt/echarts3/tree/master/zrender)
>
> [ECharts 3.0底层zrender 3.x源码分析1-总体架构](http://blog.csdn.net/future_todo/article/details/54341386)
> [ECharts 3.0底层zrender 3.x源码分析2-Painter（V层）](http://blog.csdn.net/future_todo/article/details/54341426)
> [ECharts 3.0底层zrender 3.x源码分析3-Handler（C层）](http://blog.csdn.net/future_todo/article/details/54341458)

# 1.源码结构和打包
## 1.1 源码打包
源码使用webpack打包，查看文件`webpack.config.js`可知，将echarts源码编译成三个版本，分别为常用版本,精简版本，完整版本，分别对应webpack入口文件为`index.common.js`、`index.simple.js`、`index.js`。
> 注：三个文件引用的都是`lib`文件下的文件，执行下面一步提示的命令`npm insall`后就可以得到`lib`文件夹，它里面的文件和`src`文件夹中的文件主要内容是相同的，不同之处在于：前者文件是通过类似CMD的模式打包的，后者文件是通过webpack进行打包的。我们在下面就分析`src`文件夹下的源码。注释也在其中。

执行命令顺序为
```javascript
npm install  //安装所有依赖包
webpack      //打包
webpack -p   //打成压缩包（.min.js）
```
最后生成的文件在`dist`文件夹下。
## 1.2 源码结构
首先我们要明白两个重要的概念components和charts：charts是指各种类型的图表，例如line，bar，pie等，在配置项中指的是series对应的配置；components组件是在配置项中除了serie的其余项，例如title，legend，toobox等。

源码的重要目录及说明如下(注：dist为编译后生成的文件夹)

+ extension  （扩展中使用）
+ lib  （源码中没有，执行webpack编译后才存在）
+ map  （世界地图，中国地图及中国各个省份地图的js和json两种格式的文件）
+ src  （核心源码）
+ test （示例demo）
+ theme （主题）

# 2 渲染情况
完整的例子代码[戳我](https://github.com/zrysmt/echarts3/blob/master/echarts/demo/bar.html)。

最外层是id为main的div，是我们自己写的用来渲染echarts图表的。
echarts渲染了两个div，一个div用来渲染主要的图表的，div里面嵌套一个canvas标签，
第二个div是为了显示hover层信息的。
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/echarts/echarts1.jpg)

# 3.入口`echarts.js`
位置：`src/echarts.js`。

大体的结构是一个构造函数（ECharts），原型上（ECharts.prototype）多个方法，一个echarts对象（包括对象上的属性和方法）。

和zrender一样，使用`init`方法进行初始化。
## 3.1 `init`方法
源码：

```javascript
echarts.init = function(dom, theme, opts) {
    if (__DEV__) {//是否是debug模式
        //...     //错误判断这部分内容省略
    }

    var chart = new ECharts(dom, theme, opts);//实例化ECharts
    chart.id = 'ec_' + idBase++;//chart实例的id号，唯一，逐一递增
    instances[chart.id] = chart;//唯一instance(实例)对象

    dom.setAttribute &&
        dom.setAttribute(DOM_ATTRIBUTE_KEY, chart.id);//为外层dom设置了一个属性，属性值等于chart.id

    enableConnect(chart);//按照顺序更新状态，一共三个状态
        /*var STATUS_PENDING = 0;
        var STATUS_UPDATING = 1;
        var STATUS_UPDATED = 2;*/

    return chart;
};
```
- `if (__DEV__)`验证是否是debug模式，如果是就会有错误提示（错误判断这部分内容省略），否者就是生产模式，没有错误提示。
- 参数说明

```javascript
/**
* @param {HTMLDomElement} dom 实例容器，一般是一个具有高宽的div元素
* @param {Object} [theme] 主题（说明见下面）
* @param {Object} opts 配置属性，下面几个属性
* @param {number} [opts.devicePixelRatio] Use window.devicePixelRatio by default
* @param {string} [opts.renderer] Currently only 'canvas' is supported.
* @param {number} [opts.width] Use clientWidth of the input `dom` by default.
*                              Can be 'auto' (the same as null/undefined)
* @param {number} [opts.height] Use clientHeight of the input `dom` by default.
*                               Can be 'auto' (the same as null/undefined)
*/
```
- 主题theme

```javascript
/*theme主题，可以在官网下载(http://echarts.baidu.com/download-theme.html),或者自己构建
 * 使用：
 * <script src="theme/vintage.js"></script>
 * <script>
 * // 第二个参数可以指定前面引入的主题
 * var chart = echarts.init(document.getElementById('main'), 'vintage');
 * </script>
 */
```
使用：
```javascript
var chart = echarts.init(document.getElementById('main'), null, {
   renderer: 'canvas'});
```
## 3.2 构造函数
构造函数里面是属性的初始化和zrender的初始化（`this._zr`）。
```javascript
function ECharts(dom, theme, opts) {
    opts = opts || {};
    if (typeof theme === 'string') {
        theme = themeStorage[theme];
    }
    this.id;
    this.group;
    this._dom = dom;
    var zr = this._zr = zrender.init(dom, {
        renderer: opts.renderer || 'canvas',
        devicePixelRatio: opts.devicePixelRatio,
        width: opts.width,
        height: opts.height
    });//构造函数第三个参数使用的zrender处理的

    this._throttledZrFlush = throttle.throttle(zrUtil.bind(zr.flush, zr), 17);
    this._theme = zrUtil.clone(theme);
    this._chartsViews = [];//存储所有的charts，为后面便利该变量渲染之
    this._chartsMap = {};
    this._componentsViews = [];//存储配置项组件的属性，为后面便利该变量渲染之
    this._componentsMap = {};
    this._api = new ExtensionAPI(this);
    //this._api是有'getDom', 'getZr', 'getWidth', 'getHeight', 'dispatchAction', 'isDisposed',
    //'on', 'off', 'getDataURL', 'getConnectedDataURL', 'getModel', 'getOption'方法的对象
    this._coordSysMgr = new CoordinateSystemManager();
    Eventful.call(this);
    this._messageCenter = new MessageCenter();
    this._initEvents();//初始化鼠标事件
    this.resize = zrUtil.bind(this.resize, this);

    this._pendingActions = [];
    function prioritySortFunc(a, b) {
        return a.prio - b.prio;
    }
    timsort(visualFuncs, prioritySortFunc);
    timsort(dataProcessorFuncs, prioritySortFunc);
    zr.animation.on('frame', this._onframe, this);
}
```
## 3.3 `setOption`
首先我们来看下使用api的情况，我们在前面已经说过使用`init`方法初始化echarts了，接下来只需要配置option就可以得到渲染的图表。例子中有很多省略，完整的例子代码[戳我](https://github.com/zrysmt/echarts3/blob/master/echarts/demo/bar.html)。
```javascript
chart.setOption({
    backgroundColor: '#eee',
    title: {
        text: '我是柱状图',
        padding: 20
    },
    legend: {
        inactiveColor: '#abc',
        borderWidth: 1,
        data: [{name: 'bar'}, 'bar2', '\n', 'bar3', 'bar4'],
        align: 'left',
        tooltip: {show: true }
    },
    toolbox: {
        top: 25,
        feature: {
            magicType: { type: ['line', 'bar', 'stack', 'tiled']},
            dataView: {},
            saveAsImage: {pixelRatio: 2}
        },
        iconStyle: {
            emphasis: {textPosition: 'top'}
        }
    },
    tooltip: {},
    xAxis: { //...
    },
    yAxis: { //...
    },
    series: [{
        name: 'bar',   type: 'bar',   stack: 'one',
        itemStyle: itemStyle,  data: data1
    }, {
        name: 'bar2',  type: 'bar',  stack: 'one',
        itemStyle: itemStyle,  data: data2
    }, { //... ...
    }, { //... ...
    }]
});
```
源码的主要部分列下来：
```javascript
/**
 * @param {Object} option 配置项
 * @param {boolean} notMerge 可选，是否不跟之前设置的option进行合并，默认为false，即合并。
 * @param {boolean} [lazyUpdate=false] Useful when setOption frequently.
 * //可选，在设置完option后是否不立即更新图表，默认为false，即立即更新
 */
echartsProto.setOption = function(option, notMerge, lazyUpdate) {

    this[IN_MAIN_PROCESS] = true;

    if (!this._model || notMerge) { //不和之前的option合并
        var optionManager = new OptionManager(this._api); //option配置管理
        var theme = this._theme;
        var ecModel = this._model = new GlobalModel(null, null, theme, optionManager);
        ecModel.init(null, null, theme, optionManager);
        //不合并的时候会重绘，option为最后一次使用setOption方法的参数
    }


    this.__lastOnlyGraphic = !!(option && option.graphic); //是否设置了graphic属性
    //graphic 是原生图形元素组件。可以支持的图形元素包括：image, text, circle, sector, 
    //ring, polygon, polyline, rect, line, bezierCurve, arc, group,
    //http://echarts.baidu.com/option.html#graphic
    zrUtil.each(option, function(o, mainType) {
        mainType !== 'graphic' && (this.__lastOnlyGraphic = false);
    }, this);

    //setOption之前先执行的函数列表optionPreprocessorFuncs
    this._model.setOption(option, optionPreprocessorFuncs);

    if (lazyUpdate) { //为true，不立刻更新
        this[OPTION_UPDATED] = true;
    } else {
        updateMethods.prepareAndUpdate.call(this); //准备更新
        // Ensure zr refresh sychronously, and then pixel in canvas can be
        // fetched after `setOption`.
        this._zr.flush(); //调用zrender中的方法，立即刷新
        this[OPTION_UPDATED] = false;
    }

    this[IN_MAIN_PROCESS] = false;

    flushPendingActions.call(this, false);
};
```
说明： 这里`setOption`调用的顺序是这样的`echarts.setOption`==>`GlobalModel.setOption(GlobalModel.js)`==>`OptionManager.setOption(OptionMManager.js)`
    
其中有两个关键的方法：`prepareAndUpdate`和`flush`,分别用来准备刷新和刷新，渲染图表，下面我们来一步一步看`prepareAndUpdate`方法。
## 3.4 `doRender`方法

接着上面的`prepareAndUpdate`方法看准备渲染图表视图的执行顺序：`updateMethods.prepareAndUpdate`==>`updateMethods.update`==>`doRender`==>`render`。
在`doRneder`函数中渲染所有components和charts,`render`方法分别对应在各个components和charts中有具体的实现。
```javascript
function doRender(ecModel, payload) {
    var api = this._api;
    // Render all components 渲染所有的配置组件,例如title,grid,toolbox,tooltip等
    each(this._componentsViews, function(componentView) {
        var componentModel = componentView.__model;
        console.info("componentModel:", componentModel);
        componentView.render(componentModel, ecModel, api, payload);
        //在componentModal文件夹下调用相应的render方法
        updateZ(componentModel, componentView);
    }, this);

    each(this._chartsViews, function(chart) {
        chart.__alive = false;
    }, this);

    // Render all charts 渲染所有的charts
    ecModel.eachSeries(function(seriesModel, idx) {
        var chartView = this._chartsMap[seriesModel.__viewId]; //this._chartsMap
        chartView.__alive = true;

        chartView.render(seriesModel, ecModel, api, payload);
        chartView.group.silent = !!seriesModel.get('silent');
        
        updateZ(seriesModel, chartView);
        updateProgressiveAndBlend(seriesModel, chartView);

    }, this);

    // If use hover layer 如果使用hover，更新hover层
    updateHoverLayerStatus(this._zr, ecModel);

    each(this._chartsViews, function(chart) {
        if (!chart.__alive) {
            chart.remove(ecModel, api);
        }
    }, this);
}
```
# 4.关于option的处理
`3.3`部分已经说过使用`setOption`处理配置项options，这里介绍下源码里面是怎样管理配置项的。主要源码在echarts/model/Model（以下简称Model），echarts/model/Global（以下简称GlobalModel，继承Model），echarts/model/OptionManager（以下简称OptionManager）

```javascript
this._model.setOption(option, optionPreprocessorFuncs);
```
注意这里面有个对象`this._model`用来存储配置项options的。

- Model模块是一些基本的方法，主要的方法就是`get`,`getModel`通过options的对象名获取对象值。还混合了lineStyle，areaStyle，textStyle,itemStyle方法用来管理与线，文本，项目有关的options属性。

- GlobalModel继承Model，暴露Model的方法，再封装一些自己独有的方法。

- OptionManager是用来管理options配置项的，有重要的`setOption`方法，`mergeOption`方法(私有方法，合并options），`parseRawOption`方法（私有方法，解析options）


# 5.component组件和charts图表
component组件和charts图表均有`render`方法，这是我们来重点探究的方法。
## 5.1 component组件
component组件和配置项的属性一一对应，对于复杂点的配置项，组件文件夹下的管理方式是按照MVC方式的，如legend文件夹下有基本的`LegendModel.js`(M),`LegendView`(V),`LegendAction`(C)，与其他的组件一样，还可能有其他的一些js文件。

先看一个比较简单的例子，如`title.js`,我们来分析下它的`render`方法。
- 首先是使用`extendComponentModel`，相当于Model层，用来配置一些默认的配置项（有的复杂点的会把这些单独拆分开一个Model文件）。
- 然后是`extendComponentView`,相当于View层，`render`方法就在其中（有的复杂点的会把这些单独拆分开一个View文件）。

在`render`方法中首先当然是获取到配置options对象的内容。通过titleModel.getModel（path）（每个组件都会有一个对应的Model名称）获取对应的属性path。
然后调用`new graphic.Text()`去调用zrender里面的方法，渲染到canvas上，具体的实现参考上面给出的zrender源码分析内容。
## 5.2 charts图表
在charts文件夹下是各种类型的图表，包含line，bar，pie，map等,每种类型的文件夹下都有下面的几个文件结尾的js文件。
+ `**Series.js` 继承`src/modal/series.js`中的基础方法，用来管理配置中的`series`属性，还提供一些默认的配置`defaultOption`;
+ `**Veiw.js` 继承至`src/view/Chart.js`（这里面相当于接口，没有具体实现方法），主要方法是`render`,渲染视图
`render`方法调用的是`zrender.js`中的内容，如`lineView.js`调用的是`new graphic.Rect`.

# 6.事件
示例中的使用：

```javascript
chart.on('click', function(params) {
    console.log(params);
});
```
关键源码：

```javascript
function createRegisterEventWithLowercaseName(method) {
  return function (eventName, handler, context) {
     // Event name is all lowercase
     eventName = eventName && eventName.toLowerCase();
     Eventful.prototype[method].call(this, eventName, handler, context);
   };
}
echartsProto.on = createRegisterEventWithLowercaseName('on');
echartsProto.off = createRegisterEventWithLowercaseName('off');
echartsProto.one = createRegisterEventWithLowercaseName('one');
```
`Eventful`使用的是zrender中的（zrender/mixin/Eventful），是事件扩展，包括on,off,one,trigger等方法。

我们知道canvas API没有提供监听每个元素的机制，这就需要一些处理。处理的思路是：监听事件的作用坐标（如点击时候的坐标），判断在哪个绘制元素的范围中，如果在某个元素中，这个元素就监听该事件。具体的思路可以查看[HTML5 Canvas绘制的图形的事件处理](http://blog.csdn.net/vuturn/article/details/45822905)。


参考阅读：
- [echarts 3中文官网](http://echarts.baidu.com/index.html)
- [echarts 3 github网址](https://github.com/ecomfe/echarts)
- [ECharts 3.0底层zrender 3.x源码分析1-总体架构](http://blog.csdn.net/future_todo/article/details/54341386)
- [ECharts 3.0底层zrender 3.x源码分析2-Painter（V层）](http://blog.csdn.net/future_todo/article/details/54341426)
- [ECharts 3.0底层zrender 3.x源码分析3-Handler（C层）](http://blog.csdn.net/future_todo/article/details/54341458)









