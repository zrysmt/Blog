---
title: javascript设计模式【上】
tags: 
- javascript
- 设计模式
categories: 前端技术
---
参考《javascript设计模式》[美]Addy Osmani一书，下面介绍使用javascript经常会使用的主要设计模式。本博文是使用ES5语法【上】，还有一个【下】篇，ES6语法会单独写个博客。
这里主要是一下几个设计模式：

- Constructor Pattern 构造模式
- Module Pattern 模块化模式
- Revealing Module Pattern 揭露模块化模式 
- Singleton Pattern 单例模式
- Observer Pattern  观察者模式
- Mediator Pattern 中介者模式
- Prototype Pattern 原型模式
- Command Pattern 命令行模式
- Facade Pattern 外观模式
- Factory Pattern 工厂模式
- Mixin Pattern 混入模式
- Decorator Pattern 装饰者模式
- Flyweight Pattern 享元模式

> 所有代码挂在我的[github](https://github.com/zrysmt/javascript-design-pattern)上，包含有ES5和ES6语法实现的内容。
> https://github.com/zrysmt/javascript-design-pattern

# 1.Module Pattern 模块化模式
模块化很好理解，目前很多提供模块化的库如require.js(AMD),sea.js(CMD),现在我们就看看怎样自己编写的代码能够支持模块化。
## 1.1 对象字面量表示法
```javascript
var myModule = {
    myProperty: "someValue",
    myConfig: {
        useCaching: true,
        language: "en"
    },
    // a very basic method
    myMethod: function() {
        console.log("Where in the world is Paul Irish today?");
    },
};
// Outputs: Where in the world is Paul Irish today?
myModule.myMethod();
```
## 1.2 Module(模块)模式
- 私有--IIFE模拟

```javascript
var testModule = (function() {
    var counter = 0;
    return {
        incrementCounter: function() {
            return counter++;
        },
        resetCounter: function() {
            console.log("counter value prior to reset: " + counter);
            counter = 0;
        }
    };
})();
// Usage:

testModule.incrementCounter();
testModule.resetCounter(); //1
```
- Module(模块)模式变化--混入

```javascript
var myModule = (function(jQ, _) {
    function privateMethod1() {
        jQ(".container").html("test");
    }
    function privateMethod2() {
        console.log(_.min([10, 5, 100, 2, 1000]));
    }
    return {
        publicMethod: function() {
            privateMethod1();
        }
    };
}(jQuery, _));

myModule.publicMethod();
```
- Module(模块)模式变化--引出
```javascript
var myModule = (function() {
    // Module object
    var module = {},
        privateVariable = "Hello World";
    function privateMethod() {
        // ...
    }
    module.publicProperty = "Foobar";
    module.publicMethod = function() {
        console.log(privateVariable);
    };

    return module;
}());
```
# 2.Revealing Module Pattern 揭露模块化模式
其实简单说就是将要暴露的接口返回（return）出去
```javascript
var myRevealingModule = function() {
    var privateVar = "Ben Cherry",
        publicVar = "Hey there!";

    function privateFunction() {
        console.log("Name:" + privateVar);
    }

    function publicSetName(strName) {
        privateVar = strName;
    }

    function publicGetName() {
        privateFunction();
    }

    return {
        setName: publicSetName,
        greeting: publicVar,
        getName: publicGetName
    };
}();
myRevealingModule.setName("Paul Kinlan");
```
# 3.Singleton Pattern 单例模式
确保实例化或者说是创建对象的时候只实例化/创建一次。

```javascript
var mySingleton = (function() {
    // Instance stores a reference to the Singleton
    var instance;
    function init() {
        // Private methods and variables
        function privateMethod() {
            console.log("I am private");
        }
        var privateVariable = "Im also private";
        var privateRandomNumber = Math.random();
        return {
            // Public methods and variables
            publicMethod: function() {
                console.log("The public can see me!");
            },
            publicProperty: "I am also public",
            getRandomNumber: function() {
                return privateRandomNumber;
            }
        };

    };
    return {
        getInstance: function() {
            if (!instance) {
                instance = init();
            }
            return instance;
        }
    };
})();

// Usage:
var singleA = mySingleton.getInstance();
var singleB = mySingleton.getInstance();
console.log( singleA.getRandomNumber() === singleB.getRandomNumber() ); // true
```
- 静态方法
静态方法的解释不太容易说清楚，但可以从它的特点和用处来说明：
1）静态方法不会被继承
2）静态方法不用实例化（不用new）能够用直接调用（[类名/对象名].[静态方法名]）
ES6在方法前加上`static`关键字即可，使用ES6实现的见另外一篇博客，或者直接在github中查看我的源代码。使用ES5语法实现见下面：

```javascript
var SingletonTester = (function() {
    function Singleton(options) {
        options = options || {};
        // set some properties for our singleton
        this.name = "SingletonTester";
        this.pointX = options.pointX || 6;
        this.pointY = options.pointY || 10;
    }
    var instance;
    var _static = {
        name: "SingletonTester",
        getInstance: function(options) {
            if (instance === undefined) {
                instance = new Singleton(options);
            }
            return instance;
        }
    };
    return _static;
})();

var singletonTest = SingletonTester.getInstance({
    pointX: 5
});

console.log(singletonTest.pointX);// Outputs: 5
```
# 3.Observer Pattern  观察者模式

观察者一共有四个组件：
* Subject: maintains a list of observers, facilitates adding or removing observers
目标对象（类似接口，不具体实现，只有方法名）
* Observer: provides a update interface for objects that need to be notified of a Subject's changes of state
观察者对象 主要是update方法（类似接口，不具体实现，只有方法名）
* ConcreteSubject: broadcasts notifications to observers on changes of state, stores the state of ConcreteObservers
具体目标对象，继承（实现）目标对象（实现接口）
* ConcreteObserver: stores a reference to the ConcreteSubject, implements an update interface for the Observer to ensure state is consistent with the Subject's
具体观察者，继承（实现）观察者（实现接口）

具体观察者模式的代码请移步到我的[github](https://github.com/zrysmt/javascript-design-pattern)中，这里就不单独列出来了。
其实我们现在用的最多的是它的变体-发布-订阅模式
简单解释下该模式，比如我们订阅了某些微信公众号，然后就等着别人发布信息，我们就能立刻接受到信息了。
```javascript
var pubsub = {};
(function(q) {
    var topics = {},//存放所有订阅者
        subUid = -1;
    //发布
    q.publish = function(topic, args) {

        if (!topics[topic]) {
            return false;
        }

        var subscribers = topics[topic],
            len = subscribers ? subscribers.length : 0;

        while (len--) {
            subscribers[len].func(topic, args);
        }

        return this;
    };
    //订阅
    q.subscribe = function(topic, func) {
        if (!topics[topic]) {
            topics[topic] = [];
        }

        var token = (++subUid).toString();
        topics[topic].push({
            token: token,
            func: func
        });
        return token;
    };
    //取消订阅
    q.unsubscribe = function(token) {
        for (var m in topics) {
            if (topics[m]) {
                for (var i = 0, j = topics[m].length; i < j; i++) {
                    if (topics[m][i].token === token) {
                        topics[m].splice(i, 1);
                        return token;
                    }
                }
            }
        }
        return this;
    };
}(pubsub));
//测试
var messageLogger = function(topics, data) {
    console.log("Logging: " + topics + ": " + data);
};
var subscription = pubsub.subscribe("inbox/newMessage", messageLogger);
pubsub.publish("inbox/newMessage", "hello world!");
// or
pubsub.publish("inbox/newMessage", ["test", "a", "b", "c"]);
// or
pubsub.publish("inbox/newMessage", {
    sender: "hello@google.com",
    body: "Hey again!"
});
```
# 4.Mediator Pattern 中介者模式
该模式和发布订阅模式非常像，这里就不再重复了。
# 5.Command Pattern 命令行模式
命令行模式就是类似控制台输入命令的方式。说白点就是我们只使用一个方法，第一个参数是我们实际调用的方法，后面的参数是作为该调用方法的参数。
```javascript
(function() {
    var CarManager = {
        requestInfo: function(model, id) {
            return "The information for " + model + " with ID " + id + " is foobar";
        },
        buyVehicle: function(model, id) {
            return "You have successfully purchased Item " + id + ", a " + model;
        },
        arrangeViewing: function(model, id) {
            return "You have successfully booked a viewing of " + model + " ( " + id + ") ";
        }
    };
    CarManager.execute = function(name) {
        return CarManager[name] && CarManager[name].apply(CarManager, [].slice.call(arguments, 1));
    };

    console.log(CarManager.execute("arrangeViewing", "Ferrari", "14523"));
    console.log(CarManager.execute("requestInfo", "Ford Mondeo", "54323"));
    console.log(CarManager.execute("buyVehicle", "Ford Escort", "34232"));
})();
```

> 所有代码挂在我的[github](https://github.com/zrysmt/javascript-design-pattern)上，包含有ES5和ES6语法实现的内容。
> https://github.com/zrysmt/javascript-design-pattern