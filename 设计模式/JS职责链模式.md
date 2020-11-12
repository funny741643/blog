# 职责链模式

## 定义

使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系，将这些对象连成一条链，并沿这条链传递请求，直到有一个对象处理它为止。

## 商场购物案例

```javascript
// 多个对象处理请求
var order500 = function (orderType, pay, stock) {
    if (orderType === 1 && pay === true) {
        console.log("500元定金预购，获得100元购物券");
    } else {
        return "nextSuccessor";
    }
};
var order200 = function (orderType, pay, stock) {
    if (orderType === 2 && pay === true) {
        console.log("200元定金预购， 获得50元购物券");
    } else {
        return "nextSuccessor";
    }
};
var orderNormal = function (orderType, pay, stock) {
    if (orderType === 3 && stock > 0) {
        console.log("普通购买");
    } else {
        console.log("无货");
    }
};
```

```javascript
var Chain = function(fn) {
    this.fn = fn;
    this.successor = null;
}
Chain.prototype.setNextSuccessor = function(nextSuccessor) {
    this.successor = nextSuccessor;
}
Chain.prototype.passRequest = function() {
 	var ret = this.fn.apply(this, arguments);
    if(ret === 'nextSuccessor') {
        this.successor && this.successor.passRequest.apply(this.successor, arguments);
    }
    return ret;
}


var ChainOrder500 = new Chain(order500);
var ChainOrder200 = new Chain(order500);
var ChainOrderNormal = new Chain(orderNormal);

ChainOrder500.setNextSuccessor(ChainOrder200);
ChainOrder200.setNextSuccessor(ChainOrderNormal);
ChainOrder500.passRequest(1, false, 0);
```

## 用AOP实现职责链

```javascript
Function.prototype.after = function(fn) {
	var _self = this;
    return function() {
        var ret = _self.apply(this, arguments);
        if (ret === "nextSuccessor") {
        	return fn.apply(this, arguments);
    	} 
        return ret;
    }   
}
var order = order500.after(order200).after(orderNormal);
order(1, false, 0);
```



