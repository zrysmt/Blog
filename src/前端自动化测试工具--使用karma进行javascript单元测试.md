---
title: 前端自动化测试工具--使用karma进行javascript单元测试
tags: 
- FE
- PhantomJS
- Jasmine
- Karma
- 自动化测试
categories: 前端技术
---
前面我写了一篇博客是《前端自动化测试工具PhantomJS+CasperJS结合使用教程》其中使用CasperJS不仅可以进行单元测试，还可以进行浏览器测试，是个很不错的工具，今天介绍的工具是Karma+Jasmine+PhantomJS组合的前端javascript单元测试工具。
# 1.介绍
Karma是由Google团队开发的一套前端测试运行框架，karma会启动一个web服务器，将js源代码和测试脚本放到PhantomJS或者Chrome上执行。
# 2.安装
- 包管理package.json

```
npm init
```
一路回车下去即可
- 在项目中安装karma包

```
npm i karma --save-dev
```
- karma初始化

```
karma init
```
按照下面的选择好

```bash
E:\javascript\auto-test\karma-demo>karma init

Which testing framework do you want to use ?
Press tab to list possible options. Enter to move to the next question.
> jasmine

Do you want to use Require.js ?
This will add Require.js plugin.
Press tab to list possible options. Enter to move to the next question.
> no

Do you want to capture any browsers automatically ?
Press tab to list possible options. Enter empty string to move to the next question.
> PhantomJS
>

What is the location of your source and test files ?
You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
Enter empty string to move to the next question.
> src/**/*.js
> test/**/*.js
14 10 2016 10:49:43.958:WARN [init]: There is no file matching this pattern.

>

Should any of the files included by the previous patterns be excluded ?
You can use glob patterns, eg. "**/*.swp".
Enter empty string to move to the next question.
>

Do you want Karma to watch all the files and run the tests on change ?
Press tab to list possible options.
> yes


Config file generated at "E:\javascript\auto-test\karma-demo\karma.conf.js".

```
上图是选项的示例，这里使用jasmine测试框架，PhantomJS作为代码运行的环境（也可以选择其他浏览器作为运行环境，比如Chrome，IE等）。最后在项目中生成karma.conf.js文件
- 安装jasmine-core

```bash
npm i jasmine-core --save-dev
```
# 3.demo1--ES5
目录结构
```
karma-example
    ├──  src
         ├──  index.js
    ├──  test
    ├──  package.json
```
源码：src--index.js
```javascript
function isNum(num) {
    if (typeof num === 'number') {
        return true;
    } else {
        return false;
    }
}
```
测试：test--index.js
```javascript
describe('index.js: ', function() {
  it('isNum() should work fine.', function() {
    expect(isNum(1)).toBe(true)
    expect(isNum('1')).toBe(false)
  })
})
```
运行，执行命令
```
karma start
```
命令行结果

```bash
14 10 2016 10:56:13.038:WARN [karma]: No captured browser, open http://localhost:9876/
14 10 2016 10:56:13.067:INFO [karma]: Karma v1.3.0 server started at http://localhost:9876/
14 10 2016 10:56:13.101:INFO [launcher]: Launching browser PhantomJS with unlimited concurrency
14 10 2016 10:56:13.119:INFO [launcher]: Starting browser PhantomJS
14 10 2016 10:56:16.207:INFO [PhantomJS 2.1.1 (Windows 8 0.0.0)]: Connected on socket /#JoOdYxAeCS4xvhHHAAAA with id 87859111
PhantomJS 2.1.1 (Windows 8 0.0.0): Executed 1 of 1 SUCCESS (0.009 secs / 0.004 secs)
```
# 4.demo2-ES6
安装**使用Webpack+Babel**
```
npm i  karma-webpack --save-dev
npm i  babel-loader babel-core babel-preset-es2015 --save-dev
```
源码src--index2.js
```javascript
function isNum(num) {
  if (typeof num === 'number') {
    return true;
  } else {
    return false;
  }
}
 
export {isNum};
// export default isNum;
```
测试test--index2.js

```javascript
import {isNum} from '../src/index2';
// import isNum from '../src/index2';

describe('index2.js:', () => {
  it('isNum() should work fine.', () => {
    expect(isNum(1)).toBe(true);
    expect(isNum('1')).toBe(false);
  });
});
```
修改配置文件`karma.conf.js`

```javascript
config.set({
        basePath: '',
        frameworks: ['jasmine'],
        //修改
        files: [
            // 'src/**/*.js',
            'test/**/*.js'
        ],
        exclude: [],
        preprocessors: {
            'test/**/*.js': ['webpack', 'coverage'] //新增
            //coverage为覆盖率测试，这里不再介绍
        },
        reporters: ['progress', 'coverage'],
        // 新增--覆盖率测试
        coverageReporter: {
            type: 'html',
            dir: 'coverage/'
        },
        port: 9876,
        colors: true，
        logLevel: config.LOG_INFO,
        autoWatch: true,
        browsers: ['PhantomJS'],
        singleRun: false,
        concurrency: Infinity,
        //新增
        webpack: {
            module: {
                loaders: [{
                    test: /\.js$/,
                    loader: 'babel',
                    exclude: /node_modules/,
                    query: {
                        presets: ['es2015']
                    }
                }]
            }
        }
    })
```
**参考阅读：**
- [http://karma-runner.github.io/](http://karma-runner.github.io/)
- [https://github.com/karma-runner/karma](https://github.com/karma-runner/karma)
- [前端单元测试之Karma环境搭建](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551281&idx=2&sn=a2c7e0c5ce40d3c76a77878bb059b247&chksm=8025a1f0b75228e69ba643cba44872120d8a54c5ec240c36fd37f2d8b5a24d2e980464df651e&scene=1&srcid=0921IND89Hz7S81VX0ZCtsGf#rd)