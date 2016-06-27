---
title: 在线浏览online office/pdf探究
tags: 
- FE
- javascript
categories: 前端技术
---
### 需求
实现类似百度文库的功能。百度文库是有专门进行转化成html文件的服务器

我们想要用最小的代价实现，有两条路可以走：   
1. 前端js实现[目前来看比较困难]。    
2. 使用js/jquery插件，pdf有插件[pdfjs](https://mozilla.github.io/pdf.js/)    
	office系列暂时没发现    
3. 使用服务，google服务，国内没办法使用   
	**google docs viewer** ，支持office/pdf    
    使用方式较简单，api在后台将文件转成一张张的图片（不是一次性全部转换，是滑动时才转换下一页），然后显示在网页上
   
    方式1：(url参考需转码)     
    ```javascript
    <iframe src="http://docs.google.com/gview?url=http://xyz/test.xls
    &embedded=true" style="width:600px; height:500px;" frameborder="0">    
    </iframe>    
    ```
    方式2：(url参考需转码)    
    直接在浏览器输入：http://docs.google.com/viewer?url=http://xyz/test.xls    

4. 服务器端实现   
   实现成本较高    
**下面以php服务器端语言为例**   
> http://jingyan.baidu.com/article/76a7e409a93301fc3b6e159e.html     

大概流程是：office文档上传-->转化成PDF-->转化成SWF数据集XML-->前台展示       


### 参考资料：
> http://www.cnblogs.com/java-koma/archive/2011/10/14/2212052.html
> 转化格式的在线工具  https://smallpdf.com/cn