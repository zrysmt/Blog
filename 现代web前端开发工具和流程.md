---
title: 现代web前端开发工具和流程
tags: 
- FE
- javascript
categories: 前端技术
---

# 1.版本控制
- **SVN** 
- **GIT**
推荐使用git，git安装和图形化界面tortoiseGit安装，[git与github联系](http://www.cnblogs.com/peterzd/archive/2012/04/22/2465230.html)不在本文的讨论范围，请自行搜索。
在github中新建一个项目
在本地使用图形Git-->git clone
或者使用命令：
```bash
git clone  git://github.com/someone/some_project.git
```
文件夹就是我们的项目文件夹。
# 2.前端自动化--gulp
当然目前比较流行的webpack,其配置过程大体一致，这里不去赘述。
## 2.1 package.json初始化（包管理文件）
```bash
npm init
```
一路回车确定
```javascript
{
  "name": "cjeb",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/zrysmt/cjeb.git"
  },
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/zrysmt/cjeb/issues"
  },
  "homepage": "https://github.com/zrysmt/cjeb#readme",
  "devDependencies": {  }
}
```
如果有package.json文件
```bash
npm install
```
## 2.2 安装局部gulp
```bash
npm install --save-dev gulp
```
下载的包会存放在项目的node_modules文件夹下
包依赖会加入到`package.json`中:
```javascript
"devDependencies": {
    "gulp": "^3.9.1"
  }
```
## 2.3 新建gulpfile.js文件
实例见`3.2 在`gulpfile.js`中配置``.
## 2.4 在命令窗口就可以执行gulp命令
实例见`3.3 命令执行`.
# 3.SASS
'CSS预处理器'，它的基本思想是，用一种专门的编程语言，进行网页样式设计，然后再编译成正常的CSS文件。
## 3.1 安装
```bash
npm install --save-dev gulp-sass
npm install --save-dev gulp-watch
```
`gulp-sass`是用来将SASS转化为CSS的，`gulp-watch`是用来观察文件修改的变化
我们来看`package.json`文件的变化
```javascript
  "devDependencies": {
      "gulp":"^3.9.1",
      "gulp-sass":"^2.3.1",
      "gulp-watch":"^4.3.6",
  }
```
## 3.2 在`gulpfile.js` 中配置
```javascript
'use strict';
var gulp = require('gulp');
var sass = require('gulp-sass');
var path = {
    sass_isstudy:'./modules/istudy/sass/',
};
gulp.task('default', function() {
    // 将你的默认的任务代码放在这
});

gulp.task('sass', function() {
    return gulp.src(path.sass_isstudy+'*.scss')
        .pipe(sass.sync().on('error', sass.logError))
        .pipe(gulp.dest('./modules/istudy/css'));
});

gulp.task('sass:watch', function() {
     gulp.watch(path.sass_isstudy+'*.scss', ['sass']);
     // gulp.watch('./modules/istudy/sass/*.scss').on('change',livereload);
});   //监控sass变化
```
## 3.3 命令执行
```bash
gulp sass:watch
```
执行上诉命令，在sass文件修改、保存后，gulp就会将sass文件转化为css文件

# 4.模块化编程
具体参见文章【javascript模块化编程】
## 4.1 **ES5时代**
依seajs为例：
CMD(Custom Module Definition)通用模块加载
### 4.1.1 seajs
**引入seajs文件**
```html
<script type="text/javascript" src="../../common/jsext/sea-debug.js"></script>
```
**seajs 的简单配置**
```javascript
seajs.config({
  base: "../sea-modules/",
  alias: {
    "jquery": "jquery/jquery/1.10.1/jquery.js"
  }
})
// 加载入口模块
seajs.use("../static/hello/src/main");//入口
```
**定义模块：**
```javascript
// 所有模块都通过 define 来定义
define(function(require, exports, module) {

  // 通过 require 引入依赖
  var $ = require('jquery');
  var Spinning = require('./spinning');

  // 通过 exports 对外提供接口
  exports.doSomething = ...

  // 或者通过 module.exports 提供整个接口
  module.exports = ...

});
```
### 4.1.2 seajs-text
另外可以使用seajs-text加载html文件或者tpl片段，seajs-css加载css文件
```html
<script src="path/to/sea.js"></script>
<script src="path/to/seajs-text.js"></script>

<script>
define("main", function(require) {

 // You can require `.tpl` file directly
 var tpl = require("./data.tpl")
//或者html
var html =require("./a.html");
$('.some_class').append(html);
})
</script>
```
### 4.1.3 seajs-css

```html
<script src="path/to/sea.js"></script>
<script src="path/to/seajs-css.js"></script>

<script>

// seajs can load css file after loading css plugin.
seajs.use("path/to/some.css");
//很多时候可以使用require的方式
require("path/to/some.css");
</script>
```
## 4.2 **ES6时代**
```javascript
//bar.js
function hello(who){
    return "hello "+who;
}

export {hello};
```
```javascript
//foo.js
import {hello} from "bar";

var n = "zry";
function awe(){
    console.log(bar.hello(n).toUpperCase());
}

export {awe};
```
```javascript
//baz.js
import {bar} from "bar";
import {foo} from "foo";

console.log(bar.hello('smt'));//hello smt

foo.awe();//HELLO ZRY
```
当然现在需要使用babel转成es5，并且要使用打包工具browserify webpack rollup 等才能直接在现在的浏览器上运行。

# 5.组件化
组件化的思路是将一个模块独立开来，比如要写一个选择器按钮，将其分为三层：
- 数据层：用来决定按钮个数以及按钮是否选择
- 表现层：按钮使用现有的ui组件
- 逻辑层：按钮事件等逻辑处理
组件化会单独写一个博客分析。

**参考阅读：**

- [Git 常用命令详解](http://blog.csdn.net/sunboy_2050/article/details/7529022)
- [SASS用法指南-阮一峰](http://www.ruanyifeng.com/blog/2012/06/sass.html)
- [SASS入门](http://www.w3cplus.com/)
- [阮一峰-es6入门]（http://es6.ruanyifeng.com/）

