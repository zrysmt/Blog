---
title: 前端视频播放初探总结，video标签-视频插件jwplayer
tags: 
- FE
- 插件
categories: 前端技术
---

### 1.HTML5原生支持`<video>`    
简单使用：     
```javascript
<video src="../TestRes/test.mp4" autoplay controls></video>
```
html5的video标签只支持mp4、webm、ogg三种格式,`<video>`原生支持的格式     

>  https://developer.mozilla.org/zh-TW/docs/Web/HTML/Supported_media_formats         

H.264已经占领视频市场的80%。如果移动应用视频，建议编译成264格式，有好的高压缩比、高画质。简单说H.264与mp4的关系。[H.264，同时也是MPEG-4第十部分，是由ITU-T视频编码专家组（VCEG）和ISO/IEC动态图像专家组（MPEG）联合组成的联合视频组（JVT，Joint Video Team）提出的高度压缩数字视频编解码器标准](http://baike.baidu.com/view/56322.htm)


### 2.js/jquery插件
- 比如[视频播放插件Video.js](http://www.jq22.com/jquery-info404),Video.js 是一个通用的在网页上嵌入视频播放器的 JS 库，Video.js 自动检测浏览器对 HTML5 的支持情况，如果不支持 HTML5 则自动使用 Flash 播放器.其实HTML5使用的仍然是`<video> `  
- [jW player](https://www.jwplayer.com/)也是比较常用的一款插件，作用同上.并且有[android/ios](https://developer.jwplayer.com/)的SDK. 并且支持交互，广告等，我将在最后一部分写一个简单的Demo。                   
- [12个用于播放音乐和视频文件的jQuery插件](http://www.iteye.com/news/21953/)         

> 包含音频的插件库 
  http://www.jq22.com/jquery-plugins%E9%9F%B3%E9%A2%91%E5%92%8C%E8%A7%86%E9%A2%91-1-jq

### 3.CDN云
如果是建设中小型的视频播放网站或者直播网站，推荐使用视频云服务商，这方面做的好的有[腾讯视频云](https://www.qcloud.com/product/cvm.html)，[七牛直播云](http://www.qiniu.com/),[网易云信](http://netease.im/?&from=googlejj&hmsr=google&hmpl=pinpai&hmcu=danyuan&hmkw=kw0492&gclid=CImwhvz-t80CFdgXaAod6o0AGw),[UCloud直播云](https://www.ucloud.cn/site/active/ulive.html)这些服务商有适于开发者的文档和API，并且按需收费。    

### jwplayer的使用     
插件分为免费版和收费版，免费版足够个人使用      
#### 1)服务
- 1.[填写邮箱](https://www.jwplayer.com/sign-up/),然后在邮箱中设置密码，完成注册。
- 2.确定后进入[Dashboard]
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/jwplayer%E6%8F%92%E4%BB%B6.png)
右上角【Drag&Drop a file】上传视频文件，它会给我们生成不同分辨率的视频，并且只用
``<script src="//content.jwplatform.com/players/NcKoTIsi-Yo4JE2Tw.js"></script>``就可以嵌入我们的网站（注：生成视频过程需要时间）。        

#### 2)[开发平台](https://developer.jwplayer.com/jw-player/docs/developer-guide/api/javascript_api_introduction/)
**2.1)** 下载源码，记得一定要在[官网](https://www.jwplayer.com/)上登陆，登陆进入自己的Dashboard，进入Dashboard的左边Tools栏目，各版本的下载就在下方。
**2.2)** 引入jwplayer.js和key，key所在位置同2.1)     
```javascript
	<script src="//mywebsite.com/jwplayer/jwplayer.js"></script>
	<script>jwplayer.key="ABCdeFG123456SeVenABCdeFG123456SeVen==";</script>
```
初始化使用：     
```javascript
	<div id="myDiv">This text will be replaced with a player.</div>
	<script>
    jwplayer("myDiv").setup({
        "file": "../assert/第1讲：Axure原型作品演示.mp4",
        "image": "http://example.com/myImage.png",
        "height": 360,
        "width": 640
    });
    </script>
```
> 注意：如果div标签在模板引擎中会报错``jwplayerModule.js:10 Uncaught TypeError: jwplayer(...).setup is not a function``

参考阅读  
- [`<video>`的使用方法-MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)         
- [HTML原生视频格式之争](http://www.ruanyifeng.com/blog/2010/05/html5_codec_fight.html)       
- [H.264赢下视频格式大战已十拿八稳，五分之四的视频采用该格式](http://36kr.com/p/70116.html)   
- [``http://caniuse.com#search=video``](http://caniuse.com/)            
- [PHP+FFMPEG自动转码H264标准Mp4文件](https://segmentfault.com/a/1190000000689321)     
- [html5视频播放解决方案](http://www.cnblogs.com/wellsoho/p/3498852.html)       
- [知乎-视频播放插件](https://www.zhihu.com/search?type=content&q=%E8%A7%86%E9%A2%91%E6%92%AD%E6%94%BE%E6%8F%92%E4%BB%B6)
