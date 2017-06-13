---
title: CSS3--font-face使用
tags: 
- FE
- CSS
- CSS3
categories: 前端技术
---
# 1.介绍
- @font-face是CSS3中的一个模块，他主要是把自己定义的Web字体嵌入到你的网页中，不用担心兼容性，@font-face在IE4中都支持。
- 如果是用字体做logo，英文的话字体和图片占用大小差不多，但是中文的字体包一般比较大，最好还是使用图片的形式。

# 2.快速实践
- [下载字体](http://www.dafont.com/)需要格式为.tff格式的字体文件
- 搜索Webfont Generator，或者直接使用[该网站](https://www.web-font-generator.com/)提供的服务。这很简单，进入网站后选择.tff字体文件上传，勾选同意的复选框，点击`Generate web font`，点击`Download Package`下载，解压缩文件。
- 使用
新建index.css

```css
 @font-face {
     font-family: 'Happy-Camper-Regular';
     src: url('../fonts2/Happy-Camper-Regular.eot');
     src: url('../fonts2/Happy-Camper-Regular.eot?#iefix') format('embedded-opentype'), url('../fonts2/Happy-Camper-Regular.woff') format('woff'), url('../fonts2/Happy-Camper-Regular.ttf') format('truetype'), url('../fonts2/Happy-Camper-Regular.svg#SingleMaltaRegular') format('svg');
     font-weight: normal;
     font-style: normal;
 }
 
 h2.demo {
 	font-size: 100px;
    font-family: 'Happy-Camper-Regular'
 }

```
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>字体</title>
	<link rel="stylesheet" href="index.css">
</head>
<body>
	<h2 class="demo">hello world!You are my Destiny</h2>
</body>
</html>
```

# 3.字体icon
使用某些字体，如：`WebSymbols-Regular`[百度云下载地址](http://pan.baidu.com/s/1jIO0Y2q)，`Guifx`字体，包括现在开源的比较流行的`Font Awesome`,使用方法同上。在html文件中如下示例：

```html
	<span>A</span>
	<span>B</span>
	<span>C</span>
	<span>D</span>
	<span>F</span>
```
每一行显示的是其对应的图标
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/font-icon.png)
参考文献：
- [下载字体的地方](http://www.dafont.com/)
- [CSS3 @font-face](http://www.w3cplus.com/content/css3-font-face)
- [@font-face制作Web Icon](http://www.w3cplus.com/css3/web-icon-with-font-face)
