---
title: javascript设计模式 使用ES6语法
tags: 
- javascript
- 设计模式
categories: 前端技术
---
参考《javascript设计模式》[美]Addy Osmani一书，下面介绍使用javascript经常会使用的主要设计模式。本博文ES6语法的博客，还有使用ES5语法的【上】【下】两篇。
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
```javascript
let privateName = Symbol('privateName');//利用Symbol做成私有的变量
//直接用class类
class MyModule {
    set container(value) {
        this.value = value;
    }
    get container() {
        return this.value;
    }
    init() {
        this.value = 'ES6 module';
    }
    [privateName](){
      console.log("hi");
    }
}
export default Module;
```
# 2.Singleton Pattern 单例模式
确保实例化或者说是创建对象的时候只实例化/创建一次。

- 一般的单例模式

```javascript
let instance = null;
class mySingleton {
    constructor() {
        if (!instance) instance = this;
        return instance;
    }
    publicMethod() {
        console.log("The public can see me!");
    }
}

let singleton1 = new mySingleton();
```
- 静态方法和单例模式

静态方法的解释不太容易说清楚，但可以从它的特点和用处来说明：
1）静态方法不会被继承
2）静态方法不用实例化（不用new）能够用直接调用（[类名/对象名].[静态方法名]）

```javascript
class mySingleton {
    static getInstance() {
        if (!mySingleton.instance) {
            mySingleton.instance = new mySingleton();
        }
        return mySingleton.instance;
    }
    publicMethod() {
        console.log("The public can see me!");
    }
}
var cache = mySingleton.getInstance();
```
# 3.Observer Pattern  观察者模式
观察者一共有四个组件：

*   Subject: maintains a list of observers, facilitates adding or removing observers
    目标对象（类似接口，不具体实现，只有方法名）
*   Observer: provides a update interface for objects that need to be notified of a Subject’s changes of state
    观察者对象 主要是update方法（类似接口，不具体实现，只有方法名）
*   ConcreteSubject: broadcasts notifications to observers on changes of state, stores the state of ConcreteObservers
    具体目标对象，继承（实现）目标对象（实现接口）
*   ConcreteObserver: stores a reference to the ConcreteSubject, implements an update interface for the Observer to ensure state is consistent with the Subject’s
    具体观察者，继承（实现）观察者（实现接口）
    
```javascript
/*Subject 目标*/
class Subject {
    addObserver() {
        throw new Error("This method must be overwritten!");
    }
    removeObserver() {
        throw new Error("This method must be overwritten!");    
    }
    notify() {
        throw new Error("This method must be overwritten!");
    }
}

class Observer {
    update() {
        throw new Error("This method must be overwritten!");
    }
}
//=============================================================
//具体的对象
class ControlCheckbox extends Subject {
    constructor() {
        super();
        this.observers = [];
    }
    addObserver(observer){
        this.observers.push(observer);
    }
    notify(context) {
        let observerCount = this.observers.length;
        for (let i = 0; i < observerCount; i++) {
           this.observers[i].update(context);
        }
    }
}


//具体的观察者
class AddedCheckboxs extends Observer{
    constructor(subject){
        super();
        console.log(subject);
        this.subject = subject;
        // this.subject.addObserver(this);
    }
    update(context){
        this.checked = context;
    }
}

//main test
let addBtn = document.getElementById("addNewObserver"),
    container = document.getElementById("observersContainer"),
    controlCheckboxDom = document.getElementById("mainCheckbox");
let controlCheckbox = new ControlCheckbox();
controlCheckboxDom.onclick = function(){
    controlCheckbox.notify(controlCheckboxDom.checked);//通知了变化
}

addBtn.onclick = function(){
    var check = document.createElement("input");
    check.type = "checkbox";
    //新增的每一个都应该实现观察者
    console.info(controlCheckbox.observers);//查看是否添加上
    check.update = AddedCheckboxs.prototype.update;
    controlCheckbox.addObserver(check);//添加到观察者列表上去
    container.appendChild(check);
}
```
其实我们现在用的最多的是它的变体-发布-订阅模式
简单解释下该模式，比如我们订阅了某些微信公众号，然后就等着别人发布信息，我们就能立刻接受到信息了
```javascript
//发布-订阅模式
class Pubsub {
    constructor(){
        this.subUid = 0; //订阅的id值
        this.topics = {};//存放所有订阅者
    }
    publish(topic, args) {
        if (!this.topics[topic]) {
            return false;
        }

        let subscribers = this.topics[topic],
            len = subscribers ? subscribers.length : 0;

        while (len--) {
            subscribers[len].func(topic, args);
        }

        return this;
    }

    subscribe(topic, func) {
        if (!this.topics[topic]) {
            this.topics[topic] = [];
        }

        let token = (++this.subUid).toString();
        this.topics[topic].push({
            token: token,
            func: func
        });
        return token;
    }
    unsubscribe(token) {
        for (let m in this.topics) {
            if (this.topics[m]) {
                for (let i = 0, j = this.topics[m].length; i < j; i++) {
                    if (this.topics[m][i].token === token) {
                        this.topics[m].splice(i, 1);
                        return token;
                    }
                }
            }
        }
        return this;
    }
}

//usage
let messageLogger = function(topics, data) {
    console.log("Logging: " + topics + ": " + data);
};
let pubsub = new Pubsub();
let subscription = pubsub.subscribe("inbox/newMessage", messageLogger);

pubsub.publish("inbox/newMessage", "hello world!");
```
# 4.Mediator Pattern 中介者模式
该模式和发布订阅模式非常像，这里就不再重复了

# 5.Command Pattern 命令行模式
命令行模式就是类似控制台输入命令的方式。说白点就是我们只使用一个方法，第一个参数是我们实际调用的方法，后面的参数是作为该调用方法的参数。

```javascript
class CarManager {
    requestInfo(model, id) {
        return "The information for " + model + " with ID " + id + " is foobar";
    }
    buyVehicle(model, id) {
        return "You have successfully purchased Item " + id + ", a " + model;
    }
    arrangeViewing(model, id) {
        return "You have successfully booked a viewing of " + model + " ( " + id + ") ";
    }
    execute(name) {
        let carManager = new CarManager();
        return carManager[name] && carManager[name].apply(carManager, [].slice.call(arguments, 1));
    }
}
let carManager = new CarManager();
console.log(carManager.execute("arrangeViewing", "Ferrari", "14523"));
console.log(carManager.execute("requestInfo", "Ford Mondeo", "54323"));
console.log(carManager.execute("requestInfo", "Ford Escort", "34232"));
```
# 6.Facade Pattern 外观模式
外观模式也是只暴露一个很简单的方法，然后该方法在内部执行，调用内部的其他方法，jquery使用了很多这种模式，如$().css、$.ajax()等。

```javascript
//装饰者模式
class Facade {
    _get() {
        console.log("current value:" + this.i);
    }
    _set(val) {
        this.i = val;
    }
    _run() {
        console.log("running");
    }
    _jump() {
        console.log("jumping");
    }

    facade(args) {
        this._set(args.val);
        this._get();
        if (args._run) {
            this._run();
        }
    }
}
let fa = new Facade();
fa.facade({ run: true, val: 10 });
```

# 7.Factory Pattern 工厂模式
怎么解释呢？工厂模式就是创建一个大型的制作工厂，然后其它的对象从这个工厂中产生。

```javascript
class VehicleFactory {
    constructor() {
        this.vehicleClass = Car;
    }
    createVehicle(options) {
        if (options.vehicleType === "car") {
            this.vehicleClass = Car;
        } else {
            this.vehicleClass = Truck;
        }
        return new this.vehicleClass(options);
    }
}

class Car {
    constructor(options) {
        // some defaults
        options = options || "";
        this.doors = options.doors || 4;
        this.state = options.state || "brand new";
        this.color = options.color || "silver";
    }
}
class Truck {
    constructor(options) {
        this.state = options.state || "used";
        this.wheelSize = options.wheelSize || "large";
        this.color = options.color || "blue";
    }
}
//usage 
let carFactory = new VehicleFactory();
let car = carFactory.createVehicle({
    vehicleType: "car",
    color: "yellow",
    doors: 6
});
```

# 8.Mixin Pattern 混入模式
简单解释下，混入就是将一个对象的方法复制给另外一个对象。

```javascript
//http://es6.ruanyifeng.com/#docs/class#Mixin模式的实现
function mix(...mixins) {
  class Mix {}
  for (let mixin of mixins) {
    copyProperties(Mix, mixin);
    copyProperties(Mix.prototype, mixin.prototype);
  }
  return Mix;
}
function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor" && key !== "prototype" && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
//使用-继承即可
class DistributedEdit extends mix(Loggable, Serializable) {
  // ...
}
```
# 9.Decorator Pattern 装饰者模式
装饰模式是只针对一个基本的对象，添加一些修饰。如下面的是对MacBook，加内存（Memory函数装饰）增加75美元，雕刻（Engraving函数装饰）增加200美元，买保险（Insurance函数装饰）增加250美元。
```javascript
class MacBook {
    cost() {
        return 997;
    }
    screenSize() {
        return 11.6;
    }
}

function Memory(macbook) {
    let v = macbook.cost();
    macbook.cost = function() {
        return v + 75;
    };
}
// Decorator 2
function Engraving(macbook) {

    let v = macbook.cost();
    macbook.cost = function() {
        return v + 200;
    };
}
// Decorator 3
function Insurance(macbook) {
    let v = macbook.cost();
    macbook.cost = function() {
        return v + 250;
    };
}
let mb = new MacBook();
Memory(mb);
Engraving(mb);
Insurance(mb);
console.log(mb.cost());// Outputs: 1522
console.log(mb.screenSize());// Outputs: 11.6
```
# 10.Flyweight Pattern 享元模式
享元模式我感觉就是共享一些数据或者方法，有一个工厂可以管理

*   Flyweight
    享元对象（类似于接口），提供的可以共享的属性/方法；
*   Concrete Flyweight
    具体享元对象，实现接口，具体实现享元对象的方法；
*   Flyweight Factory
    享元工厂对象，创建并管理flyweight对象

实现接口的方法，由于js没有，我们需要模拟。
这部分没有必要完全使用ES6语法，请参考我的上一篇博客。

```javascript
//在js模拟存虚拟的继承，类似java中的implements
Function.prototype.implementsFor = function(parentClassOrObject) {
    if (parentClassOrObject.constructor === Function) {
        this.prototype = new parentClassOrObject();
        this.prototype.consturctor = this;
        this.prototype.parent = parentClassOrObject.prototype;
    }else {
        //纯虚拟继承
        this.prototype = parentClassOrObject;
        this.prototype.constructor = this;
        this.prototype.parent = parentClassOrObject;
    }
}

// Flyweight object 享元对象
var CoffeeOrder = {
    // Interfaces 接口
    serveCoffee: function(context) {},
    getFlavor: function() {}
};
// ConcreteFlyweight object that creates ConcreteFlyweight 具体享元对象
function CoffeeFlavor(newFlavor) {
    var flavor = newFlavor;
    if (typeof this.getFlavor === "function") {
        this.getFlavor = function() {
            return flavor;
        };
    }
    if (typeof this.serveCoffee === "function") {
        this.serveCoffee = function(context) {
            console.log("Serving Coffee flavor " + flavor + " to table number " + context.getTable());
        };
    }
}
// Implement interface for CoffeeOrder 实现接口
CoffeeFlavor.implementsFor(CoffeeOrder);
// tableNumber 订单数 辅助器
function CoffeeOrderContext(tableNumber) {
    return {
        getTable: function() {
            return tableNumber;
        }
    };
}
//享元工厂对象
//创建并管理flyweight对象
function CoffeeFlavorFactory() {
    var flavors = {},
        length = 0;
    return {
        getCoffeeFlavor: function(flavorName) {
            //这是个单例模式
            var flavor = flavors[flavorName];
            if (flavor === undefined) {
                flavor = new CoffeeFlavor(flavorName);//创建flyweight对象
                flavors[flavorName] = flavor;
                length++;
            }
            return flavor;
        },
        getTotalCoffeeFlavorsMade: function() {
            return length;
        }
    };
}

// Sample usage: 测试
testFlyweight()
function testFlyweight() {
    // The flavors ordered. 已订购的flavors
    var flavors = new CoffeeFlavor(),
        // The tables for the orders. 
        tables = new CoffeeOrderContext(),
        // Number of orders made 订单数量
        ordersMade = 0,
        // The CoffeeFlavorFactory instance
        flavorFactory;
    //flavorIn 订单物的名称
    function takeOrders(flavorIn, table) {
        flavors[ordersMade] = flavorFactory.getCoffeeFlavor(flavorIn);
        //flavorFactory管理者创建好后(管理者也做了处理)返回给CoffeeFlavor
        tables[ordersMade++] = new CoffeeOrderContext(table);
    }

    flavorFactory = new CoffeeFlavorFactory();

    takeOrders("Cappuccino", 2);
    takeOrders("Cappuccino", 2);
    takeOrders("Frappe", 1);
    takeOrders("Frappe", 1);
    takeOrders("Xpresso", 1);
    for (var i = 0; i < ordersMade; ++i) {
        flavors[i].serveCoffee(tables[i]);
    }
    console.log("total CoffeeFlavor objects made: " + flavorFactory.getTotalCoffeeFlavorsMade());
}
```

> 所有代码挂在我的[github](https://github.com/zrysmt/javascript-design-pattern)上，包含有ES5和ES6语法实现的内容。
> https://github.com/zrysmt/javascript-design-pattern