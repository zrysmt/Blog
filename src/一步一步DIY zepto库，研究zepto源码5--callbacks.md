---
title: 一步一步DIY zepto库，研究zepto源码5-- callbacks
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---
Callbacks API用来管理回调函数，也作为deferred延迟对象的基础部分，今天就一起来探寻它的源码（对应src下的callbacks.js）。

> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，对应文件夹v0.5.1。
> https://github.com/zrysmt/DIY-zepto

注：要在[github源代码中](https://github.com/madrobby/zepto)自己编译的话，要在基础包`命令：npm run dist`上要进行扩展了，输入命令：

```bash
MODULES="zepto event ajax  callbacks" npm run dist
# on Windows
> SET MODULES=zepto event ajax  callbacks
> npm run dist
```
# 1.示例Demo
```javascript
 var foo = function(value) {
     console.log('foo:' + value);
 };
 var bar = function(value) {
     console.log('bar:' + value);
 };
```
**示例1：**
```javascript
var callbacks = $.Callbacks();
callbacks.add(foo);
callbacks.fire(['hello', 'wo', '123']);
callbacks.add(bar);
callbacks.fire('中');
 /*foo:hello,wo,123
   foo:中
   bar:中*/
```
标记：
 *  once： 回调只能触发一次
 *  memory 记录上一次触发回调函数列表时的参数，之后添加的函数都用这参数立即执行
 *  unique  一个回调函数只能被添加一次        
 *  stopOnFalse 当某个回调函数返回false时中断执行

**示例2：**
```javascript
var callbacks = $.Callbacks({
    memory: true
});
callbacks.add(foo);
callbacks.fire(['hello', 'wo', '123']);
callbacks.add(bar);
/*foo:hello,wo,123
  bar:hello,wo,123*/
```
**示例3：**
```javascript
var callbacks = $.Callbacks({
    memory: true,
    once: true //只能执行一次
});
callbacks.add(foo);
callbacks.fire(['hello', 'wo', '123']);
callbacks.add(bar);
callbacks.fire(['hello', 'wo', '123']);
/*foo:hello,wo,123
  bar:hello,wo,123*/
```
# 2.整体结构
```javascript
var Callbacks = function($) {
    $.Callbacks = function(options) {
        Callbacks = {/*add remove has empty等方法*/};
        return Callbacks;
    };
};
export default Callbacks;
```
# 3.源码
当然你也可以结合下一部分的过程分析，来理解源码。
## 3.1 几个重要变量
```javascript
options = $.extend({}, options);

var memory, // Last fire value (for non-forgettable lists)
    fired, // Flag to know if list was already fired    //是否回调过
    firing, // Flag to know if list is currently firing  //回调函数列表是否正在执行中
    firingStart, // First callback to fire (used internally by add and fireWith) //第一回调函数的下标
    firingLength, // End of the loop when firing   //回调函数列表长度？
    firingIndex, // Index of currently firing callback (modified by remove if needed)
    list = [], // Actual callback list     //回调数据源： 回调列表
    stack = !options.once && [], // Stack of fire calls for repeatable lists//回调只能触发一次的时候，stack永远为false
```
## 3.2 fire函数-- 回调底层函数
```javascript
/**
 * 触发   回调底层函数
 */
fire = function(data) {
    memory = options.memory && data;
    fired = true;
    firingIndex = firingStart || 0;
    firingStart = 0;
    firingLength = list.length;
    firing = true; //正在回调
    //遍历回调列表，全部回调函数都执行，参数是传递过来的data
    for (; list && firingIndex < firingLength; ++firingIndex) {
        //如果 list[ firingIndex ] 为false，且stopOnFalse（中断）模式
        //list[firingIndex].apply(data[0], data[1])  这是执行回调
        //data经过封装，[context,arg] 第一个参数为上下文
        if (list[firingIndex].apply(data[0], data[1]) === false && options.stopOnFalse) {
            memory = false; //中断回掉执行
            break;
        }
    }
    firing = false; //回调执行完毕
    if (list) {
        //stack里还缓存有未执行的回调
        if (stack) { //options.once存在的时候，不执行下面的一行
            stack.length && fire(stack.shift()); //执行stack里的回调
        } else if (memory) {
            list.length = 0; //memory 清空回调列表
        } else {
            Callbacks.disable(); //其他情况如  once 禁用回调
        }
    }
}
```
## 3.3 Callbacks对象
回调函数管理：添加add() 移除remove()、触发fire()、锁定lock()、禁用disable()回调函数。它为Deferred异步队列提供支持。

原理：通过一个数组保存回调函数，其他方法围绕此数组进行检测和操作

```javascript
Callbacks = {
    add: function() {
        if (list) {
            var start = list.length,
                add = function(args) {
                    $.each(args, function(_, arg) {//$.each处理数组
                        if (typeof arg === "function") {
                            //非unique，或者是unique，但回调列表未添加过
                            if (!options.unique || !Callbacks.has(arg)) list.push(arg);
                        } else if (arg && arg.length && typeof arg !== 'string') {
                            //是数组/伪数组，添加，重新遍历   
                            add(arg);
                        }
                    });
                };
            add(arguments); //添加进列表
            if (firing) { //如果列表正在执行中，修正长度，使得新添加的回调也可以执行
                firingLength = list.length;
            } else if (memory) {
                firingStart = start; //memory 模式下，修正开始下标
                fire(memory); //立即执行所有回调
            }
        }
        return this;
    },
    //从回调列表里删除一个或一组回调函数
    remove: function() {
        if (list) { //回调列表存在才可以删除
            $.each(arguments, function(_, arg) {
                var index;
                while ((index = $.inArray(arg, list, index)) > -1) {
                    list.splice(index, 1); //执行删除
                    // Handle firing indexes
                    if (firing) {
                        //避免回调列表溢出
                        if (index <= firingLength) --firingLength; //在正执行的回调函数后，递减结尾下标
                        if (index <= firingIndex) --firingIndex; //在正执行的回调函数前，递减开始下标
                    }
                }
            });
        }
        return this;
    },
    //检查指定的回调函数是否在回调列表中

    has: function(fn) {
        return !!(list && (fn ? $.inArray(fn, list) > -1 : list.length));
    },
    //清空回调函数
    empty: function() {
        firingLength = list.length = 0;
        return this;
    },
    //禁用回掉函数
    disable: function() {
        list = stack = memory = undefined;
        return this;
    },
    //是否禁用了回调函数
    disabled: function() {
        return !list;
    },
    //锁定回调函数
    lock: function() {
        stack = undefined;
        //非memory模式下，禁用列表
        if (!memory) Callbacks.disable();
        return this;
    },
    //是否是锁定的
    locked: function() {
        return !stack;
    },
    //用上下文、参数执行列表中的所有回调函数
    fireWith: function(context, args) {
        // 未回调过，非锁定、禁用时
        if (list && (!fired || stack)) {
            args = args || [];
            args = [context, args.slice ? args.slice() : args];
            if (firing) {
                stack.push(args); //正在回调中,存入stack
            } else {
                fire(args); //否则立即回调,外层fire函数
            }
        }
        return this;
    },
    fire: function() {
        //执行回调
        return Callbacks.fireWith(this, arguments);
    },
    //回调列表是否被回调过
    fired: function() {
        return !!fired;
    }
};
```
# 4.过程分析
对于第一部分的示例：
```javascript
var callbacks = $.Callbacks();
callbacks.add(foo);
callbacks.fire(['hello', 'wo', '123']);
```
+ add方法（CallBacks.add方法）
  - add函数(`add(arguments)`)
    $.each可以处理参数是数组的形式
    - 非数组  直接存放到list中`list.push(arg)`;
    - 数组    递归  `add(arg)`
  - 如果memory存在，为ture，离开执行 (`fire(memory)`)
  - 返回this，可以链式调用
+ fire方法(CallBacks.fire方法)
  - Callbacks.fireWith(this, arguments)
    - 未回调过，非锁定、禁用时
      - 处理参数，包装成数组`args = [context, args.slice ? args.slice() : args]
      - 如果正在执行回调（`firing = true`）  `stack.push(args);//正在回调中,存入stack`
      - 否则 调用外层fire函数，立即执行。
+ 外层fire函数
  - 参数处理
   ```javascript
   memory = options.memory && data;
   fired = true;
   firingIndex = firingStart || 0;
   firingStart = 0;
   firingLength = list.length;
   firing = true; //正在回调
   ```
  - 遍历回调函数执行 for循环
    - 如果 list[ firingIndex ] 为false，且stopOnFalse（中断）模式
      立即中断`memory = false; break;`
    - 否则,立刻执行
    `list[firingIndex].apply(data[0], data[1])`
  - 处理结束  firing = false; //回调执行完毕
  - option.once为false，即stack为true
    stack.length && fire(stack.shift()); //执行stack里的回调
  - option.memory存在  则清空list【此时已经执行完结束了】
  - 其余情况，包括option.once为true，在遍历回调函数中已经执行过了，这里禁用回掉即可
    `Callbacks.disable();`

> 全部代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，本博文对应文件夹v0.5.x。
> https://github.com/zrysmt/DIY-zepto

参考阅读：
- [Zepto源码分析-callbacks模块](http://www.cnblogs.com/mominger/p/4369469.html)