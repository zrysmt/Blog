---
title: rollup + es6最佳实践
tags: 
- FE
- javascript
- es6
- rollup
categories: 前端技术
---
简单说下rollup就是只将调用过的模块打包，做到尽量精简的打包。

使用webpack 1.X 版本是无法利用该特性来避免引入冗余模块代码的

webpack2 已经出来好几款 beta 版本了，同样也加上了对 Tree-shaking 的支持

# 1.src中的文件
jquery.js

```javascript
// 出口
import init from './init';
init(jQuery);

export default jQuery;
```
init.js

```javascript
var init = function(jQuery){
    jQuery.fn.init = function (selector, context, root) {
        if (!selector) {
            return this;
        } else {
            var elem = document.querySelector(selector);
            if (elem) {
                this[0] = elem;
                this.length = 1;
            }
            return this;
        }
    }; 
    jQuery.fn.init.prototype = jQuery.fn;
};

export default init;
```
# 2.安装包
pakage.json 包管理
```
npm init
```
开始安装
```
npm i rollup rollup-plugin-babel babel-preset-es2015-rollup --save-dev
```
# 3.编译
## 3.1 命令行编译

```
rollup src/jquery.js --output bundle.js -f cjs
```
### 3.1.1 编译成commonjs格式的文件

```javascript
'use strict';

var init = function(jQuery){
    jQuery.fn.init = function (selector, context, root) {
        if (!selector) {
            return this;
        } else {
            var elem = document.querySelector(selector);
            if (elem) {
                this[0] = elem;
                this.length = 1;
            }
            return this;
        }
    }; 
    jQuery.fn.init.prototype = jQuery.fn;
};

init(jQuery);

module.exports = jQuery;
```
另外还有几种格式
```
amd /  es6 / iife / umd
```
### 3.1.2 **umd**
```
rollup src/jquery.js --output bundle.js -f umd
```
会报错
```
You must supply options.moduleName for UMD bundles
```
这是因为我们在`jquery.js`中
```
export default jQuery;
```
我们使用配置方式进行编译,就能指定导出的模块名`moduleName: 'jQuery'`
## 3.2 配置编译--rollup -c rollup.config.dev.js
`rollup.config.dev.js`

```javascript
import babel from 'rollup-plugin-babel';

export default {
  entry: 'src/jquery.js',
  format: 'iife',
  moduleName: 'jQuery',
  plugins: [babel() ],
  dest: 'bundle.js',
};
```
src中`.babelrc`
```
{
  presets: [
    ["es2015", { "modules": false }]
  ]
}
```
注意` { "modules": false }`一定要有，否则一直报错，错误如下所示
```bash
Error transforming E:\javascript\rollup-demo\src\jquery.js with 'babel' plugin:                                        It looks like your Babel configuration specifies a module transformer. Please                                        disable it. If you're using the "es2015" preset, consider using "es2015-rollup"                                        instead. See https://github.com/rollup/rollup-plugin-babel#configuring-babel f                                       or more information
Error: Error transforming E:\javascript\rollup-demo\src\jquery.js with 'babel'                                        plugin: It looks like your Babel configuration specifies a module transformer.                                        Please disable it. If you're using the "es2015" preset, consider using "es2015-                                       rollup" instead. See https://github.com/rollup/rollup-plugin-babel#configuring-                                       babel for more information
    at preflightCheck (E:\javascript\rollup-demo\node_modules\rollup-plugin-bab                                       el\dist\rollup-plugin-babel.cjs.js:43:102)
    at Object.transform$1 [as transform] (E:\javascript\rollup-demo\node_module                                       s\rollup-plugin-babel\dist\rollup-plugin-babel.cjs.js:104:18)
    at C:\Users\Ruyi\AppData\Roaming\npm\node_modules\rollup\src\utils\transfor                                       m.js:19:35
    at process._tickCallback (node.js:379:9)
    at Function.Module.runMain (module.js:459:11)
    at startup (node.js:138:18)
    at node.js:974:3
```
命令
```
rollup -c rollup.config.dev.js
```
## 3.3 配置编译--node rollup.config.dev.js
`rollup.config.dev.js`

```javascript
var rollup = require( 'rollup' );
var babel = require('rollup-plugin-babel');
 
rollup.rollup({
    entry: 'src/jquery.js',
    plugins: [ babel() ]
}).then( function ( bundle ) {
    bundle.write({
        format: 'umd',
        moduleName: 'jQuery', //umd或iife模式下，若入口文件含 export，必须加上该属性
        dest: 'bundle.js',
        sourceMap: true 
    });
});
```



参考阅读：
- [Issue rollup -c](https://github.com/rollup/rollup-plugin-babel/issues/72)
- [rollup.js官网 http://rollupjs.org/guide/](http://rollupjs.org/guide/)
- [## 冗余代码都走开——前端模块打包利器 Rollup.js 入门](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651550907&idx=2&sn=4acac6d96a0ce4fc61e37b798dd90fd8&scene=21#wechat_redirect)
