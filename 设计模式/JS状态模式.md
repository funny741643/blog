#  状态模式

## 思想目标

将事务的每种状态封装为状态类，将跟该对象类有关的行为方式封装在该类的内部。

状态模式的关键是区分事物的内部状态，事物的内部状态的改变往往会带来事物的行为发生改变

## 传统面向对象语言对其的基本实现

1. 设置不同的内部类，并进行封装其对应的行为
2. 建立上下文类，用于保存当前状态和每个内部状态，将请求委托给当前的状态类
3. 建立上下文类的原型方法，如初始化和设置当前状态

开关灯案例

```javascript
// 关闭灯光的状态类
var OffLightState = function (light) {
    this.light = light; // 保存状态实例
};
// 状态类对应的行为
OffLightState.prototype.buttonWasPressed = function () {
    console.log("微光开启");
    this.light.setState(this.light.weakLightState);
};
var WeakLightState = function (light) {
    this.light = light; // 保存状态实例
};
WeakLightState.prototype.buttonWasPressed = function (light) {
    console.log("强光开启");
    this.light.setState(this.light.strongLightState);
};
var StrongLightState = function (light) {
    this.light = light; // 保存状态实例
};
StrongLightState.prototype.buttonWasPressed = function () {
    console.log("灯光关闭");
    this.light.setState(this.light.offLightState);
};

// Light类也可称之为context即上下文
var Light = function () {
    this.offLightState = new OffLightState(this);
    this.weakLightState = new WeakLightState(this);
    this.strongLightState = new StrongLightState(this);
    this.button = null;
};

Light.prototype.init = function () {
    // 保存light对象的引用
    var button = document.createElement("button"),
        self = this;
    button.innerHTML = "开关";
    this.button = document.body.appendChild(button);

    this.currState = this.offLightState; //设置当前状态
    this.button.onclick = function () {
        // 将请求委托给当前的状态对象
        self.currState.buttonWasPressed();
    };
};
// 设置当前的状态
Light.prototype.setState = function (newState) {
    this.currState = newState;
};

var light = new Light();
light.init();
```

## Js实现对象模式

1. 设置状态机FSM，FSM保存各个状态
2. 建立上下文类，设置当前状态，并将请求委托给FSM里对应的状态。

```javascript
 // 使用javascript实现状态模式
var Light = function() {
    this.currState = FSM.off;   //设置当前状态
    this.button = null;
}
Light.prototype.init = function() {
    var button = document.createElement('buttom');
    var self = this;
    button.innerHTML = "已关灯";
    this.button = document.body.appendChild(button);

    this.button.onclick = function() {
        // 将请求委托给状态机FSM
        self.currState.buttonWasPressed.call(self);
    }
}

var FSM = {
    off: {
        buttonWasPressed: function() {
            console.log('关灯');
            this.button.innerHTML = "下次按我是开灯";
            this.currState = FSM.on;
        }
    },
    on: {
        buttonWasPressed: function() {
            console.log('开灯');
            this.button.innerHTML = "下次按我是关灯";
            this.currState = FSM.off;
        }
    }
}

var light = new Light()
light.init()
```

## 使用delegate函数实现

```javascript
var delegate = function (client, delegation) {
    return {
        buttonWasPressed: function () {
            // 将客户的操作委托给delegation, 绑定this为当前Light的实例对象
            return delegation.buttonWasPressed.apply(client, arguments);
        },
    };
};
var FSM = {
    off: {
        buttonWasPressed: function () {
            console.log("关灯");
            this.button.innerHTML = "下一次按我是开灯";
            this.currentState = this.onState;
        },
    },
    on: {
        buttonWasPressed: function () {
            console.log("开灯");
            this.button.innerHTML = "下一次按我是关灯";
            this.currentState = this.offState;
        },
    },
};
var Light = function () {
    this.offState = delegate(this, FSM.off);
    this.onState = delegate(this, FSM.on);
    this.currentState = this.offState;
    this.button = null;
};
Light.prototype.init = function () {
    var self = this;
    var button = document.createElement("button");
    button.innerHTML = "已关闭";
    this.button = document.body.appendChild(button);
    this.button.onclick = function () {
        self.currentState.buttonWasPressed();
    };
};
var light = new Light();
light.init();
```

## 实例场景

* hover动作的显示，悬浮，隐藏状态转换
* tcp建立连接，监听，关闭连接状态转化