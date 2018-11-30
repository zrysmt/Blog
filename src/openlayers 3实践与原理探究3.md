---
title: OpenLayers 3实践与原理探究3-ol3一个完整的例子
tags:
- OpenLayers 3
- 地图
- js库
- WebGIS
categories: WebGIS
date: 2016-09-28 00:00:00
---
【注】所有代码挂在我的[github](https://github.com/zrysmt/openlayers-3)上，本例对应`demo3`

接着我们看一个比较长的例子，例子实现的是可以绘制图形，可以根据自己的设置打印地图。
我们先看显示效果是：

![ol3完整例子显示效果](https://raw.githubusercontent.com/zrysmt/mdPics/master/ol/ol3%E4%B8%80%E4%B8%AA%E5%AE%8C%E6%95%B4%E7%9A%84%E4%BE%8B%E5%AD%90.jpg)
由于ol3的api现在更新变化挺大的，所以自己运行的例子的时候注意版本是`3.17.1`
例子中的解释比较详细，不具体进行展开介绍。本例子主要分为三部分，在js文件中已经隔开
- 第一部分是地图的初始化，包括添加图层，添加控件
- 第二部分加个标注点，点击显示位置的弹出框
- 第三部分自定义工具，包括点、线、面、圆形、菱形、矩形、多边形的绘制工具和打印地图工具
为了节省篇幅，`index.css`在这里就不在列出，详情可以查看[百度云共享的资源](http://pan.baidu.com/s/1dFt3tIx#path=%252Fblog%252Fjs%25E5%259C%25B0%25E5%259B%25BE%25E5%25BA%2593%252Fopenlayers%25203%25E5%25AE%259E%25E8%25B7%25B5%25E4%25B8%258E%25E5%258E%259F%25E7%2590%2586%25E6%258E%25A2%25E7%25A9%25B6%252Fdemo2)
`indxe.html`

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>完整的例子</title>
    <link rel="stylesheet" href="css/index.css">
    <link rel="stylesheet" href="pubjs/v3.17.1-dist/ol.css">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.2.61/jspdf.min.js"></script>
</head>

<body>
    <div class="toolsets">
        <button id="search-position" class="search-position btn btn-success  btn-sm">查询坐标</button>
        <button id="draw-point" class="draw-point btn btn-success  btn-sm">点</button>
        <button id="draw-line" class="draw-line btn btn-success  btn-sm">线</button>
        <button id="draw-ploygon" class="draw-ploygon btn btn-success  btn-sm">多边形</button>
        <button id="draw-circle" class="draw-circle btn btn-success  btn-sm">圆形</button>
        <button id="draw-square" class="draw-square btn btn-success  btn-sm">菱形</button>
        <button id="draw-box" class="draw-box btn btn-success  btn-sm">矩形</button>
        <button id="reshape" class="reshape btn btn-info  btn-sm">修改形状</button>
        <button id="print" class="print btn btn-info  btn-sm">打印地图</button>
    </div>
    <!-- 打印地图的设置 -->
    <form class="form print-form">
        <label>Page size </label>
        <select id="format">
            <option value="a0">A0 (slow)</option>
            <option value="a1">A1</option>
            <option value="a2">A2</option>
            <option value="a3">A3</option>
            <option value="a4" selected>A4</option>
            <option value="a5">A5 (fast)</option>
        </select>
        <label>Resolution </label>
        <select id="resolution">
            <option value="72">72 dpi (fast)</option>
            <option value="150">150 dpi</option>
            <option value="300">300 dpi (slow)</option>
        </select>
    </form>
    <div id="map" class="map">
        <div style="display: none;">
            <!-- Clickable label for Vienna -->
            <a class="overlay" id="Shanghai" target="_blank" href="http://en.wikipedia.org/wiki/Shanghai">Shanghai</a>
            <div id="marker" title="Marker"></div>
            <!-- Popup -->
            <div id="popup" title="点击查询:"></div>
        </div>
    </div>
    <script type="text/javascript" src="pubjs/jquery.js"></script>
    <script type="text/javascript" src="pubjs/v3.17.1-dist/ol-debug.js"></script>
    <script type="text/javascript" src="js/index.js"></script>
</body>

</html>

```
`index.js`

```javascript
/********************************************************************************************/
var view = new ol.View({
    center: [9101767, 2822912],
    zoom: 6
}); //map.view的变量
/*图层*/
var bglayer = new ol.layer.Tile({
    source: new ol.source.BingMaps({
        // key: 'Your Bing Maps Key from http://www.bingmapsportal.com/here',
        key: 'AgiU9gCjKNfaR2yFSDfLw8e9zUlAYisRvRC2_L-LsGYN2bII5ZUvorfP3QJvxmjn', //自己申请的key
        imagerySet: 'Aerial'
    })
});
var source = new ol.source.Vector({ wrapX: false });
//绘图绘在此矢量图层
var vector = new ol.layer.Vector({
    source: source,
    style: new ol.style.Style({ //修改绘制的样式
        fill: new ol.style.Fill({
            color: 'rgba(255, 255, 255, 0.2)'
        }),
        stroke: new ol.style.Stroke({
            color: '#ffcc33',
            width: 2
        }),
        image: new ol.style.Circle({
            radius: 7,
            fill: new ol.style.Fill({
                color: '#ffcc33'
            })
        })
    })
});

var map = new ol.Map({
    controls: ol.control.defaults().extend([
        new ol.control.FullScreen(), //全屏控件
        new ol.control.ScaleLine(), //比例尺
        new ol.control.OverviewMap(), //鹰眼控件
        // new ol.control.Zoom(),
    ]),
    layers: [bglayer, vector], //添加两个图层
    target: 'map', //div#id='map'
    view: view,
    // interaction:
});


/**上面的部分就可以初始化地图**/
/********************************************************************************************/
/**
 * Marker标注
 */
var pos = ol.proj.fromLonLat([121.3725, 31.008889]); //经纬度坐标转换
// Vienna marker
var marker = new ol.Overlay({
    position: pos,
    positioning: 'center-center',
    element: document.getElementById('marker'),
    stopEvent: false
});
map.addOverlay(marker);
// Shanghai label
var Shanghai = new ol.Overlay({
    position: pos,
    element: document.getElementById('Shanghai')
});
map.addOverlay(Shanghai); //标签 a
/**
 * Popup查询坐标弹出框
 */
// Popup showing the position the user clicked
var container = document.getElementById('popup');
var content = document.getElementById('popup-content');
var closer = document.getElementById('popup-closer');

var popup = new ol.Overlay({
    element: container,
    autoPan: true,
    autoPanAnimation: {
        duration: 250
    }
});
map.addOverlay(popup);
//关闭popup
closer.onclick = function() {
    popup.setPosition(undefined);
    closer.blur();
    return false;
};

$('.search-position').click(function(event) {
    map.removeInteraction(draw); //点击选择时候  取消上次结果

    //在地图上点击
    map.on('click', function(evt) {
        var coordinate = evt.coordinate;
        var hdms = ol.coordinate.toStringHDMS(ol.proj.transform(
            coordinate, 'EPSG:3857', 'EPSG:4326'));

        content.innerHTML = '<p>点击的坐标是:</p><code>' + hdms +
            '</code>';
        popup.setPosition(coordinate);
    });

});
/********************************************************************************************/
/* 自定义工具 */
var draw, select, modify;
$('.toolsets button').click(function(event) {
    console.log($(this).text());
    var geometryFunction, shapeName, maxPoints;
    map.removeInteraction(draw); //点击选择时候  取消绘图交互
    map.removeInteraction(select); //点击选择时候  取消选择
    map.removeInteraction(modify); //点击选择时候  取消修改
    switch ($(this).text()) {
        case "点":
            shapeName = 'Point';
            break;
        case "线":
            shapeName = 'LineString';
            break;
        case "多边形":
            shapeName = 'Polygon';
            break;
        case "圆形":
            shapeName = 'Circle';
            break;
        case "菱形":
            shapeName = 'Circle';
            geometryFunction = ol.interaction.Draw.createRegularPolygon(4);
            break;
        case "矩形":
            shapeName = 'LineString';
            maxPoints = 2;
            geometryFunction = function(coordinates, geometry) {
                if (!geometry) {
                    geometry = new ol.geom.Polygon(null);
                }
                var start = coordinates[0];
                var end = coordinates[1];
                geometry.setCoordinates([
                    [start, [start[0], end[1]], end, [end[0], start[1]], start]
                ]);

                return geometry;
            };
            break;
        case "修改形状":
            reshape.init();
            break;
        case "打印地图":
            printMap.init();
            break;
    }

    draw = new ol.interaction.Draw({
        source: source,
        type: /** @type {ol.geom.GeometryType} */ (shapeName),
        geometryFunction: geometryFunction,
        maxPoints: maxPoints
    });
    map.addInteraction(draw); //增加的交互
});
/*修改地图*/
var reshape = {
    init: function() {
        // select选择形状
        // modify修改形状
        var select = new ol.interaction.Select({
            wrapX: false
        });

        var modify = new ol.interaction.Modify({
            features: select.getFeatures()
        });
        // var selectModify = new ol.interaction.defaults().extend([select, modify]);
        map.addInteraction(select);
        map.addInteraction(modify);
        //interactions: ol.interaction.defaults().extend([select, modify]),
    }
};
/*打印地图*/
var printMap = {
    init: function() {
        map.removeInteraction(draw); //点击选择时候  取消绘制
        var dims = {
            a0: [1189, 841],
            a1: [841, 594],
            a2: [594, 420],
            a3: [420, 297],
            a4: [297, 210],
            a5: [210, 148]
        };
        var loading = 0;
        var loaded = 0;
        // var exportButton = document.getElementById('export-pdf');
        // exportButton.disabled = true;
        document.body.style.cursor = 'progress';

        var format = document.getElementById('format').value;
        var resolution = document.getElementById('resolution').value;
        var dim = dims[format];
        var width = Math.round(dim[0] * resolution / 25.4);
        var height = Math.round(dim[1] * resolution / 25.4);
        var size = /** @type {ol.Size} */ (map.getSize());
        var extent = map.getView().calculateExtent(size);

        var source = bglayer.getSource();
        var tileLoadStart = function() {
            ++loading;
        };

        var tileLoadEnd = function() {
            ++loaded;
            if (loading === loaded) {
                var canvas = this;
                window.setTimeout(function() {
                    loading = 0;
                    loaded = 0;
                    var data = canvas.toDataURL('image/png'); //canvas
                    var pdf = new jsPDF('landscape', undefined, format);
                    pdf.addImage(data, 'JPEG', 0, 0, dim[0], dim[1]);
                    pdf.save('map.pdf');
                    source.un('tileloadstart', tileLoadStart);
                    source.un('tileloadend', tileLoadEnd, canvas);
                    source.un('tileloaderror', tileLoadEnd, canvas);
                    map.setSize(size);
                    map.getView().fit(extent, size);
                    map.renderSync();
                    // exportButton.disabled = false;
                    document.body.style.cursor = 'auto';
                }, 100);
            }
        };

        map.once('postcompose', function(event) {
            source.on('tileloadstart', tileLoadStart);
            source.on('tileloadend', tileLoadEnd, event.context.canvas);
            source.on('tileloaderror', tileLoadEnd, event.context.canvas);
        });

        map.setSize([width, height]);
        map.getView().fit(extent, /** @type {ol.Size} */ (map.getSize()));
        map.renderSync();
    }

};
```

