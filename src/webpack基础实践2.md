---
title: webpack基础实践2
tags: 
- FE
- javascript
- webpack
categories: 前端技术
date: 2016-09-28 00:00:00
---
本文在webpack基础实践1的基础上，主要阐述的是模块化和ES6与webpack的结合使用。
# 1.模块化
**commonJS/CMD风格**
module1.js

```javascript
var obj = {
	val:"hello from m1",
	sayHi:function(){
		document.write('hi');
	},
	sum:function(a,b){
		return a+b;
	}
};
module.exports = obj;
```
**AMD风格**
module2.js

```javascript
define(['./module1.js'],function(m1){
	return "1+2="+m1.sum(1,2);
});
```
入口文件entry.js

```javascript
var m1 = require("./module1.js");
document.write("<br>");    
document.write(m1.val);    
document.write("<br>");    
var m2 = require("./module2.js");
document.write(m2); 
```
结果显示为
```
hello from m1
1+2=3
```
当然实际项目中不建议两种风格的模块都使用，选择其中一种模块风格即可。
# 2.ES6
webpack是支持babel转化器的，所以可以将ES6代码转为ES5供现在的浏览器使用
- 1) 安装babel依赖库

```bash
npm install babel-loader --save-dev
npm install babel-core --save-dev
npm install babel-preset-es2015 --save-dev
```
- 2) 新建一个`.babelrc`文件，内容是：

```javascript
{
  "presets": [
    "es2015"
  ]
}
```
- 3) 配置`webpack.config.js`文件
```javascript
module: {loaders: [
    { test: /\.js$/,exclude: /node_modules/,loader: 'babel-loader',}
]
```
- 4) 入口文件`entry.js`中就可以使用了

```javascript
/*es6*/
require("./es6test2.js");
```
`es6test2.js`

```javascript
import Point from './es6test1';

let p = new Point(1,2);
document.write(p.toString()); 
```
`es6test1.js`

```javascript
//定义类
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
export default Point; 
```
编译完成即可
# 3.总结

## 3.1 配置文件`webpack.config.js`

```javascript
module.exports = {
    entry: "./src/home/js/entry.js", //入口文件
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }, //css加载器
            { test: /\.scss$/, loader: "style!css!sass" }, //sass加载器
            { test: /\.(jpg|png)$/, loader: "url?limit=8192" }, //图片加载器
            { test: /\.js$/, exclude: /node_modules/, loader: 'babel-loader', }
        ]
    }
};
```
## 3.2 加载的依赖库`package.json`

```javascript
{
  "name": "webpackdemo",
  "version": "1.0.0",
  "description": "webpack demo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "^6.14.0",
    "babel-loader": "^6.2.5",
    "babel-preset-es2015": "^6.14.0",
    "css-loader": "^0.24.0",
    "file-loader": "^0.9.0",
    "node-sass": "^3.8.0",
    "sass-loader": "^4.0.0",
    "style-loader": "^0.13.1",
    "url-loader": "^0.5.7",
    "webpack": "^1.13.2"
  }
}
```
## 3.3 入口文件`entry.js`
```javascript
require("./style.css");
require("./index.scss");

document.write(require("./content.js")); 
document.write("</br/>");    

var img = document.createElement("img"); 
img.src = require("./img/webpack.png"); 
document.body.appendChild(img);
document.write("</br/>");    

/*模块化*/
var m1 = require("./module1.js");
document.write("</br/>");    
document.write(m1.val);    
document.write("<br>");    
var m2 = require("./module2.js");
document.write(m2);    
document.write("<br>");    

/*es6*/
require("./es6test2.js");
```
