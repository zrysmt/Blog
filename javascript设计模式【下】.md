---
title: javascript设计模式【下】
tags: 
- javascript
- 设计模式
categories: 前端技术
---
参考《javascript设计模式》[美]Addy Osmani一书，下面介绍使用javascript经常会使用的主要设计模式。本博文是使用ES5语法【下】，还有一个【上】篇，ES6语法会单独写个博客。
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

# 6.Facade Pattern 外观模式
外观模式也是只暴露一个很简单的方法，然后该方法在内部执行，调用内部的其他方法，jquery使用了很多这种模式，如$().css、$.ajax()等
```javascript
 var module = (function() {
     var _private = {
         i: 5,
         get: function() {
             console.log("current value:" + this.i);
         },
         set: function(val) {
             this.i = val;
         },
         run: function() {
             console.log("running");
         },
         jump: function() {
             console.log("jumping");
         }
     };
     return {
         facade: function(args) {
             _private.set(args.val);
             _private.get();
             if (args.run) {
                 _private.run();
             }
         }
     };
 }());
 // Outputs: "current value: 10" and "running"
 module.facade({ run: true, val: 10 });
```
# 7.Factory Pattern 工厂模式
怎么解释呢？工厂模式就是创建一个大型的制作工厂，然后其它的对象从这个工厂中产生。
```javascript
//工厂
function VehicleFactory() {}
// Our default vehicleClass is Car 默认的是Car制造工厂
VehicleFactory.prototype.vehicleClass = Car;

VehicleFactory.prototype.createVehicle = function(options) {
    if (options.vehicleType === "car") {
        this.vehicleClass = Car;
    } else {
        this.vehicleClass = Truck;
    }
    return new this.vehicleClass(options);
};
```
两个具体的工厂里面的制作空间
```javascript
function Car(options) {
    // some defaults
    this.doors = options.doors || 4;
    this.state = options.state || "brand new";
    this.color = options.color || "silver";
}
// A constructor for defining new trucks
function Truck(options) {
    this.state = options.state || "used";
    this.wheelSize = options.wheelSize || "large";
    this.color = options.color || "blue";
}
```
使用
```javascript
var carFactory = new VehicleFactory();
var car = carFactory.createVehicle({
    vehicleType: "car",
    color: "yellow",
    doors: 6
});

console.log(car instanceof Car);// Outputs: true
```
# 8.Mixin Pattern 混入模式
简单解释下，混入就是将一个对象的方法复制给另外一个对象。
```javascript
var Car = function(settings) {
    this.model = settings.model || "no model provided";
    this.color = settings.color || "no colour provided";
};

var Mixin = function() {};
Mixin.prototype = {
    driveForward: function() {
        console.log("drive forward");
    },
    driveBackward: function() {
        console.log("drive backward");
    },
    driveSideways: function() {
        console.log("drive sideways");
    }
};

function augment(receivingClass, givingClass) {
    if (arguments[2]) {
        for (var i = 2, len = arguments.length; i < len; i++) {
            receivingClass.prototype[arguments[i]] = givingClass.prototype[arguments[i]];
        }
    } else {
        for (var methodName in givingClass.prototype) {
            if (!Object.hasOwnProperty(receivingClass.prototype, methodName)) {
                receivingClass.prototype[methodName] = givingClass.prototype[methodName];
            }
        }
    }
}

// 只混入两个方法，Mixin的方法复制给Car
augment(Car, Mixin, "driveForward", "driveBackward");
// Create a new Car
var myCar = new Car({
    model: "Ford Escort",
    color: "blue"
});
myCar.driveForward();
myCar.driveBackward();
```
# 9.Decorator Pattern 装饰者模式
装饰模式是只针对一个基本的对象，添加一些修饰。如下面的是对MacBook，加内存（Memory函数装饰）增加75美元，雕刻（Engraving函数装饰）增加200美元，买保险（Insurance函数装饰）增加250美元。
```javascript
function MacBook() {
    this.cost = function() {
        return 997; };
    this.screenSize = function() {
       return 11.6; };
}
// Decorator 1
function Memory(macbook) {
    var v = macbook.cost();
    macbook.cost = function() {
        return v + 75;
    };
}
// Decorator 2
function Engraving(macbook) {
    var v = macbook.cost();
    macbook.cost = function() {
        return v + 200;
    };
}
// Decorator 3
function Insurance(macbook) {
    var v = macbook.cost();
    macbook.cost = function() {
        return v + 250;
    };

}
var mb = new MacBook();
Memory(mb);
Engraving(mb);
Insurance(mb);
```
# 10.Flyweight Pattern 享元模式
享元模式我感觉就是共享一些数据或者方法，有一个工厂可以管理
* Flyweight 
  享元对象（类似于接口），提供的可以共享的属性/方法；
* Concrete Flyweight 
  具体享元对象，实现接口，具体实现享元对象的方法；
* Flyweight Factory 
  享元工厂对象，创建并管理flyweight对象

实现接口的方法，由于js没有，这里我们就模拟下。
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
```
- 享元对象

```javascript
// Flyweight object 享元对象
var CoffeeOrder = {
    // Interfaces 接口
    serveCoffee: function(context) {},
    getFlavor: function() {}
};
```
- 具体享元对象

```javascript
// Implements CoffeeOrder
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
```
- 辅助器

```javascript
function CoffeeOrderContext(tableNumber) {
    return {
        getTable: function() {
            return tableNumber;
        }
    };
}
```
- 享元工厂对象

```javascript
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
```
- 测试

```javascript
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
    takeOrders("Frappe", 897);
    takeOrders("Cappuccino", 97);
    takeOrders("Cappuccino", 97);
    takeOrders("Frappe", 3);
    takeOrders("Xpresso", 3);
    takeOrders("Cappuccino", 3);
    takeOrders("Xpresso", 96);
    takeOrders("Frappe", 552);
    takeOrders("Cappuccino", 121);
    takeOrders("Xpresso", 121);

    for (var i = 0; i < ordersMade; ++i) {
        flavors[i].serveCoffee(tables[i]);
    }
    console.log(" ");
    console.log("total CoffeeFlavor objects made: " + flavorFactory.getTotalCoffeeFlavorsMade());
}
```

> 所有代码挂在我的[github](https://github.com/zrysmt/javascript-design-pattern)上，包含有ES5和ES6语法实现的内容。
> https://github.com/zrysmt/javascript-design-pattern