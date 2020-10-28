# `JS`策略模式

## 概念

定义：定义一系列算法，把它们一个个封装起来，并使它们可以相互替换。

目的：将算法的使用与算法的实现分离开来。

## `JS`中的策略模式

对于基于类的语言，我们会让strategy对象从各个策略类中创建而来。而在`js`中函数也是对象，所以我们可以直接将`strategy`直接定义为函数。

```javascript
var strategies = {
	'S': function(salary) {	// S策略对象
		return salary * 4;	// 内部算法，算法的实现
	},
    'A': function(salary) {	// A策略对象
        return; salary * 3;
    }
}
```

当用户对`Context`发起请求的时候，`Context`总是把请求委托给这些策略对象中间的一个进行计算。

```javascript
var calculateBonus = function(level, salary) {
	return strategies[level](salary);	// 算法的使用
}
```

## 缓动动画案例

```javascript
// 缓动算法 算法的实现
// 参数：动画已耗时间，小球原始位置，小球目标位置，动画持续总时间
// 返回值：动画元素应该处在的当前位置
var tween = {
    linear: function (t, b, c, d) {	// linear策略对象
        return (c * t) / d + b;		// linear算法
    },
    easeIn: function (t, b, c, d) {
        return c * (t /= d) * t + b;
    },
};
```

给页面放置一个盒子

```html
<div style="position: absolute; background: blue" id="div">我是div</div>
```

定义一个动画类

```javascript
var Animate = function (dom) {
    this.dom = dom;				// 进行运动的dom节点
    this.startTime = 0;			// 动画开始时间
    this.startPos = 0;			// 动画开始位置
    this.endPos = 0;			// 动画结束位置
    this.propertyName = null;	// dom节点需要改变的css属性名
    this.easing = null;			// 所使用的缓动动画
    this.duration = null;		// 动画持续时间
};
```

动画开始运动的方法

```javascript
// 参数: 要改变的css属性名, 小球运动的目标位置, 动画持续时间, 缓动算法
Animate.prototype.start = function (propertyName, endPos, duration, easing) { 
        this.startTime = new Date();
        this.startPos = this.dom.getBoundingClientRect()[propertyName];
        this.propertyName = propertyName;
        this.endPos = endPos;
        this.duration = duration;
        this.easing = tween[easing];

        var self = this;
        var timeId = setInterval(function () {
            if (self.step() === false) {
                clearInterval(timeId);
            }
        }, 19);
};
```

计算小球当前位置同时调用函数更新CSS属性值

```javascript
Animate.prototype.step = function () {
    var t = +new Date();
    if (t >= this.startTime + this.duration) {
        this.update(this.endPos);
        return false;
    }
    // 调用缓动算法，算法的使用
    var pos = this.easing(
        t - this.startTime,
        this.startPos,
        this.endPos - this.startPos,
        this.duration
    );
    this.update(pos);
};
```

更新CSS属性值

```javascript
Animate.prototype.update = function (pos) {
	this.dom.style[this.propertyName] = pos + "px";
};
```

测试

```javascript
var div = document.getElementById("div");
var animate = new Animate(div);
animate.start("left", 500, 1000, "linear");
```

