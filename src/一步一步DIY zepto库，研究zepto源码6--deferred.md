---
title: 一步一步DIY zepto库，研究zepto源码6--deferred
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---
接下来我们来DIY另外一个重要的模块defrred延迟对象，这当然与源码有些许的不同，然而这并不重要。

基础包上要进行扩展了，输入命令：

```bash
MODULES="zepto event ajax deferred callbacks" npm run dist
```
> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，对应文件夹v0.6.1。
> https://github.com/zrysmt/DIY-zepto

# 1.示例Demo
**示例1：**
```javascript
$.ajax({
        type: 'GET',
        // type: 'POST',
        url: '/projects.json',
        dataType: 'json',
        timeout: 300,
        success: function(data) {
            console.log(data);
        },
        error: function(xhr, type) {
            alert('Ajax error!');
        }
    }).done(function() {
        console.info("done");
    }).fail(function() {
        console.info("fail");
    }).always(function() {
        console.info("always");
    }) //then 三个参数 第一个是成功后回掉，第二个是失败，第三个是运行中
    .then(function() {
        console.info("then1");
    }, function() {
        console.info("then2");
    }, function() {
        console.info("then3");
    });
```
成功后结果:

```javascript
//done always then1
```
失败后结果:

```javascript
//alert('Ajax error!')  fail always then2
```
**示例2：**
```javascript
var wait = function(dtd) {　　　　
    var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象　　　　
    var tasks = function() {　　　　　　
        alert("执行完毕！");　　　　　　
        dtd.resolve(); // 改变Deferred对象的执行状态   　　　　
    };　　
    setTimeout(tasks, 5000);　　　　
    return dtd.promise(); // 返回promise对象
    // 返回dtd.promise  因其没有resolve和reject方法，所以在外面不能该调用这两个方法改变状态
    　　
};
```
```javascript
$.when(wait()).done(function() {
    alert("哈哈，成功了！");
})　　.fail(function() {
    alert("出错啦！");
});
```
或者：
```javascript
$.Deferred(wait).done(function() {
    alert("哈哈，成功了！");
})　　.fail(function() {
    alert("出错啦！");
});
```
# 2.整体结构
```javascript
var DeferredMod = function($) {
    var slice = Array.prototype.slice;

    function Deferred(func) {
        var tuples ,
            state = "pending", //Promoise的初始状态
            promoise = {
                state: function() {
                    //... ...
                },
                always: function() {
                    //... ...
                },
                then: function( /* fnDone [, fnFailed [, fnProgress]] */ ) {
                    //... ...
                },
                //返回obj的promise对象
                promise: function(obj) {
                    //... ...
                }
            },
            deferred = {};
        //给deferred添加切换状态方法
        $.each(tuples, function(i, tuple) {
        });
        //deferred包装成promise 继承promise对象的方法
        //调用promoise的promoise方法
        promoise.promise(deferred);
        //传递了参数func，执行
        if (func) func.call(deferred, deferred);
        //返回deferred对象
        return deferred;
    } 
    $.when = function(sub){ };
    $.Deferred = Deferred;
};

export default DeferredMod;
```
# 3.Promise规范
由于deferred是基于Promise规范，我们首先需要理清楚Promises/A+是什么。
它的规范内容大致如下
*  一个promise可能有三种状态：等待（pending）、已完成（fulfilled）、已拒绝（rejected）
*  一个promise的状态只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换
*  promise必须实现then方法（可以说，then就是promise的核心），而且then必须返回一个promise，同一个promise的then可以调用多次，并且回调的执行顺序跟它们被定义时的顺序一致
* then方法接受两个参数，第一个参数是成功时的回调，在promise由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在promise由“等待”态转换到“拒绝”态时调用。同时，then可以接受另一个promise传入，也接受一个“类then”的对象或方法，即thenable对象

伪代码实现：

```javascript
//初始化： 等待状态  pending
var Promise = {
    status: pending, //状态
    promise: function(o) {
        return {
            done: done,
            fail: fail
        }
    },
    //必须申明的then方法
    then: function(fulfilledFn, rejectedFn) {
        this.done(fulfilledFn);
        this.fail(rejectedFn);
        //返回promise对象
        return this;
    },
    //当状态切换fulfilled时执行
    done: function() { },
    //当状态切换rejected时执行
    fail: function() { },
    //切换为已完成状态
    toFulfilled: function() {
        this.status = 'fulfilled'
    },
    //切换为已拒绝状态
    toRejected: function() {
        this.status = 'rejected'
    }
}

//将函数包装成Promise对象,并注册完成、拒绝链方法
//通过then
Promise.promise(fA).then(fa1, efa1).then(fa2, efa2);
//假定fb里还调用了另一个异步FB，
//之前fA的异步回调执行到fb方法
var PA = Promise.promise(fA).then(fa, efa).then(fb, efb);
//再挂上fB的异步回调
PA.then(fB).then(fb1, efb1).then(fb2, efb2);
```
# 4.Deferred函数

```javascript
 function Deferred(func) {
     /*********************变量***************************************/
     //元组：描述状态、状态切换方法名、对应状态执行方法名、回调列表的关系
     //tuple引自C++/python，和list的区别是，它不可改变 ，用来存储常量集
     var tuples = [
             // action, add listener, listener list, final state
             ["resolve", "done", $.Callbacks({ once: 1, memory: 1 }), "resolved"],
             ["reject", "fail", $.Callbacks({ once: 1, memory: 1 }), "rejected"],
             ["notify", "progress", $.Callbacks({ memory: 1 })]
         ],
         state = "pending", //Promoise的初始状态
         //promise对象，promise和deferred的区别是:
         /*promise只包含执行阶段的方法always(),then(),done(),fail(),progress()及辅助方法state()、promise()等。
           deferred则在继承promise的基础上，增加切换状态的方法，resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()*/
         //所以称promise是deferred的只读副本
         promise = {
             // 返回状态
             state: function() {
                 return state;
             },
             //成功/失败均回调调用
             always: function() {
                 deferred.done(arguments).fail(arguments);
                 return this;
             },
             then: function( /* fnDone [, fnFailed [, fnProgress]] */ ) {
                 var fns = arguments;
                 //注意，这无论如何都会返回一个新的Deferred只读副本，
                 //返回Deferred(func)函数，传递一个函数作为参数func，
                 //`if (func) func.call(deferred, deferred);`执行func，这个时候defer就是deferred
                 return Deferred(function(defer) {
                     $.each(tuples, function(i, tuple) {
                         //i==0: done   i==1: fail  i==2 progress
                         var fn = $.isFunction(fns[i]) && fns[i];
                         //执行新deferred done/fail/progress
                         deferred[tuple[1]](function() {
                             //直接执行新添加的回调 fnDone fnFailed fnProgress
                             var returned = fn && fn.apply(this, arguments);
                             if (returned && $.isFunction(returned.promise)) {
                                 //转向fnDone fnFailed fnProgress返回的promise对象
                                 //注意，这里是两个promise对象的数据交流
                                 //新deferrred对象切换为对应的成功/失败/通知状态，传递的参数为 returned.promise() 给予的参数值    
                                 returned.promise()
                                     .done(defer.resolve)
                                     .fail(defer.reject)
                                     .progress(defer.notify);
                             } else {
                                 var context = this === promise ? defer.promise() : this,
                                     values = fn ? [returned] : arguments;
                                 defer[tuple[0] + "With"](context, values); //新deferrred对象切换为对应的成功/失败/通知状态
                             }
                         });
                     });
                     fns = null;
                 }).promise();  // 返回deferrred.promise  因其没有resolve和reject方法，所以在外面不能该调用这两个方法改变状态
             },
             //返回obj的promise对象
             promise: function(obj) {
                 return obj != null ? $.extend(obj, promise) : promise;
             }
         }
 }
```
- 这里需要重点注意的是：promise和deferred的关系：

promise只包含执行阶段的方法always(),then(),done(),fail(),progress()及辅助方法state()、promise()等。

deferred则在继承promise的基础上（第5部分：`promise.promise(deferred)`），增加切换状态的方法，resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()

所以称promise是deferred的只读副本

---

- 赋予resolve、reject、notify真正的含义是在`then`中

```javascript
//切换的状态是resolve成功/reject失败
//添加首组方法做预处理，修改state的值，使成功或失败互斥，锁定progress回调列表
if (stateString) {
   list.add(function() {
      state = stateString;
      //i^1  ^异或运算符  0^1=1 1^1=0，成功或失败回调互斥，调用一方，禁用另一方
   }, tuples[i ^ 1][2].disable, tuples[2][2].lock);
}//因为回调函数带有memory，add后立刻执行
```

---
- 提供的[`deferred.promise()`](http://www.css88.com/jqapi-1.9/deferred.promise/)方法的作用是，在原来的Deferred 对象上返回另一个 Deferred 对象，即受限制的 Promise 对象，受限制的 Promise 对象只开放与改变执行状态无关的方法（比如done()方法和fail()方法），屏蔽与改变执行状态有关的方法（比如resolve()方法和reject()方法），从而使得执行状态不能被改变
         
# 5.给deferred添加切换状态方法

```javascript
//给deferred添加切换状态方法
$.each(tuples, function(i, tuple) {
    var list = tuple[2], //$.Callback
        stateString = tuple[3]; //状态 一共2个：resolved  rejected

    //扩展promise的done、fail、progress为Callback的add方法，使其成为回调列表
    //简单写法：
    // promise['done'] = $.Callbacks( "once memory" ).add
    // promise['fail'] = $.Callbacks( "once memory" ).add  
    // promise['progress'] = $.Callbacks( "memory" ).add
    // 使用的时候  .done(func)  func就添加到了回调函数中
    promise[tuple[1]] = list.add;

    //切换的状态是resolve成功/reject失败
    //添加首组方法做预处理，修改state的值，使成功或失败互斥，锁定progress回调列表
    if (stateString) {
        list.add(function() {
            state = stateString;
            //i^1  ^异或运算符  0^1=1 1^1=0，成功或失败回调互斥，调用一方，禁用另一方
        }, tuples[i ^ 1][2].disable, tuples[2][2].lock);
    }//因为回调函数带有memory，add后立刻执行.包括上面的 promise[tuple[1]]
    //添加切换状态方法 resolve()/resolveWith(),reject()/rejectWith(),notify()/notifyWith()
    deferred[tuple[0]] = function() {
        //使用list.fireWith 调用
        deferred[tuple[0] + "With"](this === deferred ? promise : this, arguments);
        return this;
    };
    //使用 deferred.resolveWith()就使用fireWith调用了回调函数
    deferred[tuple[0] + "With"] = list.fireWith;
});
//deferred包装成promise 继承promise对象的方法
//调用promise的promise方法
promise.promise(deferred);
//传递了参数func，执行
if (func) func.call(deferred, deferred);

//返回deferred对象
return deferred;
}
```
# 6.`$.when`
```javascript
/**
     * 主要用于多异步队列处理。
       多异步队列都成功，执行成功方法，一个失败，执行失败方法
       也可以传非异步队列对象
     * @param sub
     * @returns {*}
     */
$.when = function(sub) {
    var resolveValues = slice.call(arguments), //队列数组 ，未传参数是[]
        len = resolveValues.length, //队列个数
        i = 0,
        remain = len !== 1 || (sub && $.isFunction(sub.promise)) ? len : 0, //子deferred计数
        deferred = remain === 1 ? sub : Deferred(), //主def,如果是1个fn，直接以它为主def，否则建立新的Def
        progressValues, progressContexts, resolveContexts,
        updateFn = function(i, ctx, val) {
            return function(value) {
                ctx[i] = this;
                val[i] = arguments.length > 1 ? slice.call(arguments) : value; // val 调用成功函数列表的参数
                if (val === progressValues) {
                    deferred.notifyWith(ctx, val); // 如果是通知，调用主函数的通知，通知可以调用多次
                } else if (!(--remain)) { //如果是成功，则需等成功计数为0，即所有子def都成功执行了，remain变为0
                    deferred.resolveWith(ctx, val); //调用主函数的成功
                }
            };
        };

    if (len > 1) {
        progressValues = new Array(len);
        progressContexts = new Array(len);
        resolveContexts = new Array(len);
        for (; i < len; ++i) {
            if (resolveValues[i] && $.isFunction(resolveValues[i].promise)) {
                resolveValues[i].promise()
                    .done(updateFn(i, resolveContexts, resolveValues)) //每一个成功
                    .fail(deferred.reject) //直接挂入主def的失败通知函数,当某个子def失败时，
                    //调用主def的切换失败状态方法，执行主def的失败函数列表   
                    .progress(updateFn(i, progressContexts, progressValues));
            } else {
                --remain; //非def，直接标记成功，减1
            }
        }
    }
    //都为非def，比如无参数，或者所有子队列全为非def，直接通知成功，进入成功函数列表
    if (!remain) deferred.resolveWith(resolveContexts, resolveValues);
    return deferred.promise();
};
```
示例可以见第一部分示例2

> 全部代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，本博文对应文件夹v0.6.x。
> https://github.com/zrysmt/DIY-zepto

参考阅读：

- [Zepto源码分析-deferred模块](http://www.cnblogs.com/mominger/p/4411632.html)
- [jQuery的deferred对象详解](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object.html)
- [deferred.resolveWith()](http://www.css88.com/jqapi-1.9/deferred.resolveWith/)
