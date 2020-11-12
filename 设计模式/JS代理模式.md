# 代理模式

## 认识

代理模式是为一个对象提供一个代用品或占用符，以便控制对它的访问

* 保护代理：代理B帮助A过滤一些请求
* 虚拟代理：把一些开销很大的对象，延迟到真正需要它的时候才去创建
* 缓存代理：为一些开销大的运算结果提供暂时的存储，在下次运算时，如果传递进来的参数跟之前的一样，则直接返回前面存储的运算结果

## 虚拟代理实现图片预加载

```javascript
var myImage = (function() {
    var imgNode = document.createElement("img");
    document.body.appendChild(imgNode);
    
    return {
        setSrc: function(src) {
            imgNode.src = src;
        }
    }
})();

var proxyImage = (function() {
    var img = new Image();
    // 设置了src后图片开始加载，加载成功后会执行onload事件
    img.onload = function() {
        // 加载真正的图片
        myImage.setSrc(this.src);
    }
    return {
        setSrc: function(src) {
            // 预加载一张本地图片
            myImage.setSrc = "./本地图片.jpg";
            img.src = src;
        }
    }
})
```

## 缓存代理

```javascript
var mult = function() {
    console.log('开始计算乘积');
    var a = 1;
    for (var i = 0,l = arguments.length; i < l ; i++) {
        a = a * arguments[i];
    }
    return a;
}
mult( 2 , 3)
```

```javascript
var proxyMult = (function() {
	var cache = {};
    return function() {
        var args = Array.prototype.join.apply(arguments);
        if (args in cache) return cache[args];
        return cache[args] = mult.apply(this, arguments);
    }
})();
proxyMult(1,2,3,4);
proxyMult(1,2,3,4);
```

### 高阶函数动态创建代理

```javascript
var createProxyFactory = function(fn) {
	var cache = {};
    return function() {
        var args = Array.prototype.join.call(arguments, ',');
        if (args in cache) return cache[args];
        return cache[args] = fn.apply(this, arguments);
    }
}
```

### 应用

* 实现分页，同一页的数据只需向后台拉取一次



