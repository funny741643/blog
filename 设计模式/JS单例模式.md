# `JS`单例模式

## 概念

保证一个类仅有**一个实例**，并提供一个全局访问它的**全局访问点**。

## `js`中的单例模式

传统的单例模式实现在`js`中并不适合，因为其没有类这个概念，往往我们将全局变量对象，作为一个单例来使用。

```javascript
var a = {}
```

确实它也满足单例模式的两个特点：a对象是独一无二的，我们可以在项目的任何地方使用它。

但是其存在一个很大的缺点就是，**造成命名空间污染**。

### 降低命名空间污染的方法

* 使用命名空间

  ```javascript
  var namespace1 = {
      a: function() {
          alert(1);
      },
      b: function() {
          alert(2);
      }
  }
  ```

* 使用闭包封装私有变量

  ```javascript
  var user = (function(){
  	var _name = 'sven',_age = 29;
      return {
          getUserInfo: function() {
              return _name + '-' + _age;
          }
      }
  })();
  ```

## 惰性单例

### 概念

在需要的时候才创建，惰性单例是单例模式的一个**重点**

### 基于“类”的单例模式

```javascript
Singleton.getInstance = (function(){
    var instance = null;
    return function() {
    	if (!instance) {
            instance = new Singleton();
        }
        return instance
    }
})();
```

上述代码，便是一个基于”类“的简单的惰性单例模式，`instance`实例只有在调用`getInstance`方法时才会被创建。但其在`Javascript`语言里并行不通。

## 登录浮框案例

该案例主要介绍与全局变量结合实现惰性的单例。

现在我们需要实现一个登录浮框的功能，当我们点击登录按钮时显示它，点击关闭按钮进行退出。

1. 传统方案：创建一个盒子，先隐藏它，当我们有需求时再展示它。缺点：创建过多无用的`DOM`结点。

2. 惰性单例方案：当用户点击登录按钮时才开始创建登录浮框，并且使用一个变量保存它，如果用户再次点击登录按钮，便可以直接返回它。

   ```javascript
   var createLoginLayer = (function(){
   	var div;
       return function() {
           if(!div) {
               div = document.createElement('div');
               div.innerHTML = "我是登录浮框";
               div.style.display = 'none';
               document.body.appendChild('div');
           }
           return div;
       }
   })();
   document.getElementById('loginBtn').onclick = function() {
       var loginLayer = createLoginLayer();
       loginLayer.style.display = 'block';
   }
   ```

3. 通用的惰性单例

   上述代码的缺点： 违反了单一职责原则，将创建盒子和管理单例的逻辑放在了一起。下面我们来进行逻辑的分离，以达到我们可以建立任何`DOM`元素。

   ```javascript
   // 创建盒子的逻辑
   var createLoginLayer = function() {
       div = document.createElement('div');
       div.innerHTML = "我是登录浮框";
       div.style.display = 'none';
       document.body.appendChild('div');
       return div;
   }
   ```

   ```javascript
   // 只负责管理单例
   var getSingle = function(fn) {
   	var result;
   	return result || result = fn.apply(this, arguments);
   }
   var createSingleLoginLayer = getSingle(createLoginLayer);
   ```

   