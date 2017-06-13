---
title: webpack基础实践1
tags: 
- FE
- javascript
- webpack
categories: 前端技术
---

这是个webpack的入门教程，看到网上blog大多是配置好了再解释，这样来的不太直观。本文从第一步开始慢慢做起，一步一步走下来，最后再总结，这样直观看到每个配置行代表什么含义。
webpack的作用是什么？现在说太多可能对于入门的同学来说也不好理解，索性这里就记住一句话，一张图得了
**一张图**
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webpack.png)
**一句话**
webpack是能把各种资源，例如JS(JSX),coffee,样式(CSS/SASS/LESS),图片作为模块来进行打包和处理。
# 1.安装
## 1.1 安装全局webpack
前提是在本地先安装了`node.js`.

```bash
$ npm install webpack -g
```
## 1.2 将依赖写入package.json
新建的话：

```bash
npm init
```
一路回车就可以，其实这些都是项目的描述信息和git地址等信息，这些信息我们可以后面再文件中直接修改
如果是clone的项目，已经有package.json文件了，就运行命令(忽略步骤`1.3`,`2.3`引入css加载器部分)
```bash
npm install
```
## 1.3 安装局部webpack
```bash
npm install webpack --save-dev
```
# 2.开始使用
## 2.1 起步
`entry.js`作为我们的入口文件，其中会包含其他模块(js)或者是CSS	

```javascript
document.write("入口entry.js");
```
index.html

```html
<html>
    <head>
        <meta charset="utf-8">
    </head>
    <body>
        <script type="text/javascript" src="bundle.js" charset="utf-8"></script>
    </body>
</html>
```
运行webpack命令
```bash
$ webpack ./entry.js bundle.js
```
然后index.html就可以work了
```
入口entry.js
```
## 2.2 引入其他模块
`content.js`
```javascript
module.exports = "模块content.js";
```
`entry.js`
```javascript
document.write(require("./content.js"));
```
编译命令同上
运行index.html结果
```
模块content.js
```
## 2.3 引入CSS
引入css加载器

```bash
npm install css-loader --save-dev
npm install style-loader --save-dev
```
style.css

```css
body {
    background: yellow;
}
```
entry.js

```javascript
require("!style!css!./style.css");
document.write(require("./content.js"));
```
最后编译，就是这么简单随意完成了，如果我们想这样`require("./style.css");`引入css，岂不是更加完美
使用编译命令

```bash
webpack ./entry.js bundle.js --module-bind 'css=style!css'
```
## 2.4 引入SASS文件
引入sass加载器

```bash
npm install node-sass --save-dev
npm install sass-loader --save-dev
```
index.scss
```css
body{
    color:white;
}
```
同css一样
entry.js

```javascript
require("!style!css!sass!./index.scss");
```
当然我们还不是很满意。简单点，编译命令的方式简单点。所以我们来到了配置文件
# 3.配置文件
新建文件`webpack.config.js`

```javascript
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" },//css加载器
            { test: /\.scss$/, loader: "style!css!sass" }//sass加载器
        ]
    }
};
```
编译命令就剩下这样的
```bash
webpack
```
# 5.图片的打包
图片是用url-loader加载的。css中的url属性，其实就是一种封装过的require操作。
```bash
npm install url-loader --save-dev
npm install file-loader --save-dev
```
webpack.config.js

```javascript
{test: /\.(jpg|png)$/, loader: "url?limit=8192"}
```
在js中 entry.js

```javascript
var img = document.createElement("img"); 
img.src = require("./img/webpack.png"); 
document.body.appendChild(img);
```
或者直接在css中写
```css
div.img{
    width: 300px;
    height: 300px;
    background: url("./img/font-icon.png");//小于8kb的图片会打包处理成Base64的图片
}
```
# 6.常用webpack编译命令
```bash
webpack //基本命令
webpack --progress --colors //显示打包过程
webpack -w //实时进行打包更新，文件改变时候，自动打包
webpack -p // 对打包后的文件进行压缩，提供production
webpack -d // 提供source map，方便调试。
```

关于对图片打包 AMD/CommonJS/ES6的使用在下一篇博客中`webpack基础实践2`
参考文章
- [webpack官网-getting-start](http://webpack.github.io/docs/tutorials/getting-started/)    
- [webpack前端模块加载工具](http://www.cnblogs.com/YikaJ/p/4586703.html)