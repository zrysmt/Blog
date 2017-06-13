---
title: openlayers 3扩展，调用百度地图、高德地图、天地图服务
tags:
- OpenLayers 3
- 地图
- WebGIS
categories: WebGIS
---
调用这三个商业地图服务，我们使用的都是切片（Tile）地图服务，关于切片地图的含义这里做简单的介绍：
切片地图就是指将显示的地图切成一块一块的(256 * 256)分别显示加载。openlayers 3中有这样图层加载类，`ol.layer.Tile`,对应的source类有`ol.source.TileImage`,`ol.source.XYZ`,这两者的关系通过源码可以看到
`ol.inherits(ol.source.XYZ, ol.source.TileImage);`,`ol.source.TileImage`是父类。


对于天地图，我们访问[天地图](http://map.tianditu.com/map/index.html)地图主页服务，打开控制台->`Network`,我们可以看到请求的一些地址如下：

```
http://t2.tianditu.com/DataServer?T=vec_w&x=53&y=24&l=6'
```
其中重要的信息是x,y,z分别表示x坐标，y坐标和zoomLevel,

其实在openlayers 3源码中有Bing地图和OSM地图的扩展了，我们可以仿照它们进行一些扩展。

## 1.扩展天地图

源码使用模块化打包

```javascript
var ol = require('openlayers');

ol.source.TianMap = function(options){
	var options = options ? options : {};
  	var attributions;
  	if(options.attributions !== undefined){
  		attributions = option.attributions;
  	}else{
  		attributions = [ol.source.BaiduMap.ATTRIBUTION];
  	}

    var url;
    if(options.mapType == "sat"){
    	url = "http://t{0-4}.tianditu.com/DataServer?T=img_w&x={x}&y={y}&l={z}";
    }else if(options.mapType == "satLabel"){
    	url = "http://t{0-4}.tianditu.com/DataServer?T=cia_w&x={x}&y={y}&l={z}";
    }else if(options.mapType == "label"){
    	url = "http://t{0-4}.tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}";
    }else{
    	url = "http://t{0-4}.tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}";
    }
  	
  	ol.source.XYZ.call(this, {
  	  attributions: attributions,
      projection: ol.proj.get('EPSG:3857'),
  	  cacheSize: options.cacheSize,
  	  crossOrigin: 'anonymous',
  	  opaque: options.opaque !== undefined ? options.opaque : true,
  	  maxZoom: options.maxZoom !== undefined ? options.maxZoom : 19,
  	  reprojectionErrorThreshold: options.reprojectionErrorThreshold,
  	  tileLoadFunction: options.tileLoadFunction,
  	  url: url,
  	  wrapX: options.wrapX
  	});
}
ol.inherits(ol.source.TianMap, ol.source.XYZ);

ol.source.TianMap.ATTRIBUTION = new ol.Attribution({
  	html: '&copy; <a class="ol-attribution-tianmap" ' +
      'href="http://www.tianditu.cn/">' +
      '天地图</a>'
});
module.exports = ol.source.TianMap;
```
使用方法

```javascript
var tianMapSat = new ol.layer.Tile({
    title: "天地图卫星",
    source: new ol.source.TianMap({mapType:"sat"})
});
map.addLayer(tianMapSat);
```
## 2. 扩展百度地图
百度地图坐标进行了加偏，所以需要使用`projzh`转化
百度地图使用的是定制化的墨卡托投影和BD-09 datum,所以将WGS-84坐标转化为百度坐标需要两步
first transform from WGS-84 to BD-09 (which itself uses the GCJ-09 transform), and then do the forward transform to Baidu Mercator
第一步是将WGS-84 转化为 BD-09，然后转化为百度墨卡托
```
baiduMercator.forward(bd09.fromWGS84(point))
```
反过来的转化为
```
bd09.toWGS84(baiduMercator.inverse(point))
```

> https://github.com/tschaub/projzh

```javascript
var ol = require('openlayers');
var projzh = require('projzh');
/* projzh处理百度坐标的问题，算法基于proj4m project
 * https://www.versioneye.com/nodejs/projzh/0.5.0
 * https://github.com/tschaub/projzh
 */
ol.source.BaiduMap = function(options){
	var options = options ? options : {};

  	var attributions;
  	if(options.attributions !== undefined){
  		attributions = option.attributions;
  	}else{
  		attributions = [ol.source.BaiduMap.ATTRIBUTION];
  	}

    var extent = [72.004, 0.8293, 137.8347, 55.8271];

    //定义百度坐标
    //地址：https://github.com/openlayers/openlayers/issues/3522
    var baiduMercator = new ol.proj.Projection({
        code: 'baidu',
        extent: ol.extent.applyTransform(extent, projzh.ll2bmerc),
        units: 'm'
    });
    
    ol.proj.addProjection(baiduMercator);
    ol.proj.addCoordinateTransforms('EPSG:4326', baiduMercator, projzh.ll2bmerc, projzh.bmerc2ll);
    ol.proj.addCoordinateTransforms('EPSG:3857', baiduMercator, projzh.smerc2bmerc, projzh.bmerc2smerc);


  	var resolutions = [];
    for(var i=0; i<19; i++){
        resolutions[i] = Math.pow(2, 18-i);
    }
    var tilegrid  = new ol.tilegrid.TileGrid({
        origin: [0,0],
        resolutions: resolutions,
        extent: ol.extent.applyTransform(extent, projzh.ll2bmerc),
        tileSize: [256, 256]
    });
  	var satUrls = [0, 1, 2, 3, 4].map(function(sub) {
        return 'http://shangetu' + sub +
            '.map.bdimg.com/it/u=x={x};y={y};z={z};v=009;type=sate&fm=46&udt=20150601';
    });
    var urls = [0, 1, 2, 3, 4].map(function(sub) {
        return 'http://online' + sub +
            '.map.bdimg.com/onlinelabel/qt=tile&x={x}&y={y}&z={z}&v=009&styles=pl&udt=20170301&scaler=1&p=1';
    });
    ol.source.TileImage.call(this, {
  		crossOrigin: 'anonymous',   //跨域
	    cacheSize: options.cacheSize,
        // projection: ol.proj.get('EPSG:3857'),
  		projection:'baidu',
  		tileGrid: tilegrid,
  		tileUrlFunction: function(tileCoord, pixelRatio, proj){
            if(!tileCoord) return "";

            var z = tileCoord[0];
            var x = tileCoord[1];
            var y = tileCoord[2];
            var hash = (x << z) + y;
            var index = hash % urls.length;
            index = index < 0 ? index + urls.length : index;
            if(options.mapType == "sat"){
                return satUrls[index].replace('{x}', x).replace('{y}', y).replace('{z}', z);
  			}
            return urls[index].replace('{x}', x).replace('{y}', y).replace('{z}', z);

        },
    	wrapX: options.wrapX !== undefined ? options.wrapX : true

  	});
}

ol.inherits(ol.source.BaiduMap,ol.source.TileImage);

ol.source.BaiduMap.ATTRIBUTION = new ol.Attribution({
  	html: '&copy; <a class="ol-attribution-baidumap" ' +
      'href="http://map.baidu.com/">' +
      '百度地图</a>'
});

module.exports = ol.source.BaiduMap;
```
调用
```javascript
var baiduMapSat = new ol.layer.Tile({
    title: "百度地图卫星",
    source: new ol.source.BaiduMap({mapType:"sat"})
});
map.addLayer(baiduMapSat);
```
## 3. 扩展高德地图

```javascript
var ol = require('openlayers');

ol.source.AMap = function(options){
	var options = options ? options : {};

  	var attributions;
  	if(options.attributions !== undefined){
  		attributions = option.attributions;
  	}else{
  		attributions = [ol.source.AMap.ATTRIBUTION];
  	}

  	var url;
  	if(options.mapType == "sat"){
  		url ="http://webst0{1-4}.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}";
  	}else{
  		url = "http://webrd0{1-4}.is.autonavi.com/appmaptile?lang=zh_cn&size=1&scale=1&style=7&x={x}&y={y}&z={z}";
  	}
    
    ol.source.XYZ.call(this, {
  		crossOrigin: 'anonymous',   //跨域
	    cacheSize: options.cacheSize,
        projection: ol.proj.get('EPSG:3857'),
        // urls:urls,
        url:url,
    	wrapX: options.wrapX !== undefined ? options.wrapX : true

  	});

}

ol.inherits(ol.source.AMap,ol.source.XYZ);


ol.source.AMap.ATTRIBUTION = new ol.Attribution({
  	html: '&copy; <a class="ol-attribution-amap" ' +
      'href="http://ditu.amap.com/">' +
      '高德地图</a>'
});

module.exports = ol.source.AMap;
```
调用
```javascript
var aMapSat = new ol.layer.Tile({
    title: "高德地图卫星",
    source: new ol.source.AMap({mapType:"sat"})
});
map.addLayer(aMapSat);
```

最后推荐一个[github仓库](https://github.com/zrysmt/openlayers3-react),Openlayers 3 使用React 组件化+wepack+ES6实践,
包括扩展在其中的具体使用方法
> https://github.com/zrysmt/openlayers3-react

# 参考阅读
- [openlayers github](https://github.com/openlayers/openlayers)
- [openlayers github Issues](https://github.com/openlayers/openlayers/issues)
- [openlayers 官方地址](https://openlayers.org/en/latest/apidoc/)
- [Openlayers 3 使用React 组件化+wepack+ES6实践记录笔记](http://blog.csdn.net/future_todo/article/details/61206783)
- [OpenLayers 3 之 加载百度地图](http://blog.csdn.net/qingyafan/article/details/49403989)
- [OpenLayers 3 之 加载天地图](http://blog.csdn.net/qingyafan/article/details/49565245)