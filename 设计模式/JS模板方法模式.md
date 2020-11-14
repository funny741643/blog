# 模板方法

## 认识

在抽象父类中封装子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法。

## 煮咖啡案例

```javascript
var Beverage = function () {};
Beverage.prototype.boilWater = function () {
    console.log("把水煮沸");
};
Beverage.prototype.brew = function () {
    console.log("子类需要重写父类的brew方法");
};
Beverage.prototype.pourInCup = function () {
    console.log("子类需要重写父类的pourInCup方法");
};
Beverage.prototype.addCondiments = function () {
    console.log("子类需要重写父类的addCondiments方法");
};
Beverage.prototype.customerWantsCondiments = function () {
    return true;
};
Beverage.prototype.init = function () {
    this.boilWater();
    this.brew();
    this.pourInCup();
    if (this.customerWantsCondiments()) {
        this.addCondiments();
    }
};

var CoffeeWithHook = function () {};
CoffeeWithHook.prototype = new Beverage();
CoffeeWithHook.prototype.brew = function () {
    console.log("用沸水煮咖啡");
};
CoffeeWithHook.prototype.pourInCup = function () {
    console.log("把咖啡倒进杯子里");
};
CoffeeWithHook.prototype.addCondiments = function () {
    console.log("加糖和牛奶");
};
CoffeeWithHook.prototype.customerWantsCondiments = function () {
    return window.confirm("请问需要调料吗?");
};
var coffeeWithHook = new CoffeeWithHook();
coffeeWithHook.init();
```

## 好莱坞原则重构

允许底层组件将自己挂钩到高层组件中，而高层组件会决定什么时候，以何种方式去使用这些底层组件。

当我们用模板方法编写一个程序时，就意味着子类放弃了对自己的控制权，而是改为父类通知子类，哪些方法应该在什么时候被调用。作为子类，只负责提供一些设计上的细节。

```javascript
// 好莱坞原则下重写Beverage
var Beverage = function (param) {
    var boilWater = function () {
        console.log("把水煮沸");
    };
    var brew = param.brew || function () {
        console.log("子类需要重写父类的brew方法");
    };
    var pourInCup = param.pourInCup || function () {
        console.log("子类需要重写父类的pourInCup方法");
    };
    var addCondiments = param.addCondiments || function () {
        console.log("子类需要重写父类的addCondiments方法");
    };
    var F = function () {};
    F.prototype.init = function () {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    };
    return F;
};
```

