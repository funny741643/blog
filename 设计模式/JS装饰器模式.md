# 装饰器模式

## 思想目标：

装饰器模式主要用于动态的为对象增添一些额外的职责，且不会影响从这个类中派生的对象

## 传统面向对象语言

```javascript
var Plane = function() {}
Plane.prototype.fire = function() {
    console.log('发射子弹')
}
var MissileDecorator = function(plane) {
    this.plane = plane
}
MissileDecorator.prototype.fire = function() {
    this.plane.fire();
    console.log('发射激光炮')
}
var plane = new Plane();
var missilePlane = new MissileDecorator(plane);
missilePlane.fire();	// 发射子弹，发射激光炮
```

## JS实现

```javascript
var plane = {
	fire: function() {
        console.log('发射子弹')
    }
}
var missilePlane = function() {
    console.log("发射激光炮")
}
var fire1 = plane.fire;
plane.fire = function() {
    fire1();
    missilePlane();
}
plane.fire()
```

## 用AOP装饰函数

```javascript
Function.prototype.before = function(beforefn) {
	var _self = this;
    return function() {
        beforefn.apply(this, arguments);
        return _self.apply(this, arguments)
    }
}
var plane = {
	fire: function() {
        console.log('发射子弹')
    }
}
var missilePlane = function() {
    console.log("发射激光炮")
}
var fire = plane.fire.before(missilePlane);
fire()
```

```javascript
Function.prototype.after = function(afterfn) {
	var _self = this;
    return function() {
        var ret = _self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;
    }
}
var plane = {
	fire: function() {
        console.log('发射子弹')
    }
}
var missilePlane = function() {
    console.log("发射激光炮")
}
var fire = plane.fire.after(missilePlane);
fire()
```

## 应用

* Ajax请求前配置请求参数
* 表单提交前进行表单验证