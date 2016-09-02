---
title: webpack项目实践
tags: 
- FE
- javascript
- webpack
categories: 前端技术
---
# 1.目录结构初步构想
上面的例子只是介绍了webpack的基本用法，并没有按照一个实际的项目进行构建目录结构，对于一个多页面的项目我们定义的目录结构如下
```
- web/                  # web根目录
  - src/                # 开发目录
    - home/             # 主页目录
      + css/            # css/sass资源目录
      + img/            # 图片资源目录
      + js/             # js&jsx资源目录
      entry.js          # webpack入口文件
      home.html         # 页面文件
    - about/            # about页目录
      + css/            # css/sass资源目录
      + img/            # 图片资源目录
      + js/             # js&jsx资源目录
      entry.js          # webpack入口文件
      about.html        # about页面文件
  - dist/               # 编译输出目录，即发布目录
    - home/             # 编译输出的home目录
    - about/            # 编译输出的about目录
    - common/           # 编译输出的公共资源目录
      + js/             # 编译输出的公共js目录
      + css/            # 编译输出的公共css目录
      + img/            # 编译输出的公共图片目录
  index.html            # 系统html入口
  webpack.config.js     # webpack配置文件
  package.json          # 项目配置
  .babelrc              # 配置es-2015
  README.md             # 项目说明
  ```
将上两篇博客`webpack基础实践1-2`中的例子配置文件改成如下
```javascript
var path = require("path");

module.exports = {
    entry: "./src/home/entry.js", //入口文件
    output: {
        path: path.join(__dirname, 'dist','home'),
        filename: "bundle.js",
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }, //css加载器
            { test: /\.scss$/, loader: "style!css!sass" }, //sass加载器
            { test: /\.(jpg|png)$/, loader: "url?limit=8192" }, //图片加载器
            { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader', }//babel加载器
        ]
    }
};
```
入口文件等只需要改换成相对路径即可
# 2.独立css文件
需要配合插件一起使用
```bash
npm install extract-text-webpack-plugin --save-dev
```
比1中配置文件增加/修改的内容

```javascript
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
    //... ...  
    module: {
        loaders: [
            { test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")}, //css加载器
            { test: /\.scss$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")}, //sass加载器
        ]
    },
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
};
```
此时在html文件中引入就可以了

```html
<link rel="stylesheet" href="dist/home/main.css">
```
# 3.多入口
为了模拟数据，我们在`home`文件夹下新建了一个`entry2.js`入口

```javascript
var m2 = require("./js/module2.js");
document.write(m2);
/*es6*/
require("./js/es6test2.js");
```
配置文件如下

```javascript
var path = require("path");
var ExtractTextPlugin = require("extract-text-webpack-plugin");

module.exports = {
    entry: {
        page1:"./src/home/entry.js",
        page2:"./src/home/entry2.js"

    }, //入口文件
    output: {
        path: path.join(__dirname, 'dist','home'),
        filename: "[name].js",
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")}, //css加载器
            { test: /\.scss$/, loader: ExtractTextPlugin.extract("style-loader", "css-loader")}, //sass加载器
            { test: /\.(jpg|png)$/, loader: "url?limit=8192" }, //图片加载器
            { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader', }
        ]
    },
    plugins: [
        new ExtractTextPlugin("[name].css")
    ]
};

```
这个时候需要在`index.html`中分别引入

```html
<html>

<head>
    <meta charset="utf-8">
    <link rel="stylesheet" href="dist/home/page1.css">
    <link rel="stylesheet" href="dist/home/page2.css">
</head>

<body>
    <script src="dist/home/page1.js"></script>
    <script src="dist/home/page2.js"></script>
    <div class="img"></div>
</body>

</html>
```
# 4.提取公共部分
第3部分中给出的`entry2.js`和`entry.js`是有相同的部分的，我们想要实现的是可以提取出两者的公共部分。
配置文件中增加

```javascript
var webpack = require('webpack');
//... ...
plugins: [
      new webpack.optimize.CommonsChunkPlugin('common.js')
  ]
```
如果公共部分想单列一个文件夹下，可以
```
new webpack.optimize.CommonsChunkPlugin('../common/js/common.js')
```
在html文件中引入`common.js`文件即可
# 5.实践后的项目目录
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/webpack项目文件夹.png)
