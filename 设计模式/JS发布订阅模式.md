# 发布订阅模式

订阅模式又叫观察者模式，它定义对象间的一种一对多的依赖关系

## 发布订阅者模式通用实现

```javascript
  var event = {
      clientList: {},
      listen: function (key, fn) {
          if (!this.clientList[key]) {
              this.clientList[key] = [];
          }
          this.clientList[key].push(fn);
      },
      trigger: function () {
          var key = Array.prototype.shift.apply(arguments);
          var fns = this.clientList[key];
          if (!fns || fns.length === 0) return false;
          for (var i = 0, fn; (fn = fns[i++]); ) {
              fn.apply(this, arguments);
          }
      },
      remove: function (key, fn) {
          fns = this.clientList[key];
          if (!fns) {
              return false;
          }
          if (!fn) {
              fns && (fns.length = 0);
          }
          for (var i = fns.length - 1, fn; i >= 0; i--) {
              var _fn = fns[i];
              if (_fn === fn) {
                  fns.splice(i, 1);
              }
          }
      },
  };
var installEvent = function (obj) {
    for (var i in event) {
        obj[i] = event[i];
    }
};
```

