# Generator用法详解+co

Generator函数是ES6提供的一种异步编程解决方案

> 先来看个Generator的简单的用法

```
function* read() {
    console.log(100);
    let a = yield '200';
    console.log(a);
    let b = yield 300;
    console.log(b);
    return b;
}
let it = read();
console.log(it.next('400'));  
console.log(it.next('500')); 
console.log(it.next('600')); 
console.log(it.next('700'));  
复制代码
```

> 打印结果为

```
100
{ value: '200', done: false }
500
{ value: 300, done: false }
600
{ value: '600', done: true }
{ value: undefined, done: true }
复制代码
```

首先，Generatror函数有两个特征：

- function关键字与函数名之间有一个星号
- 函数体内部使用yield语句，定义不同的内部状态

yield语句在英语里的意思就是“产出”，yield会将函数分割成好多个部分，每产出一次，就暂停一次

- Genenrator是一个生成器，调用Genenrator函数，不会立即执行函数体，只是创建了一个迭代器对象,如上例中的it就是调用read这个Generator函数得到的一个迭代器
- 迭代器有一个next方法，调用一次就会继续向下执行，直到遇到下一个yield或return
- next()方法可以带一个参数，该参数会被当做上一条yield语句的返回值,并赋值给yield前面等号前的变量
- 每遇到一个yield,就会返回一个{value:xxx,done:bool}的对象，然后暂停，返回的value就是跟在yield后面的返回值，done表示这个generator是否已经执行结束了；
- 当遇到return 时，return后的值就是value指，done此时就是true；
- 函数末尾如果没有return，就是隐含的return undefined;

上例中每一次迭代器调用next的所执行的语句可以理解为下图



![img](https://user-gold-cdn.xitu.io/2018/3/24/162558026da59a48?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 第一次执行 `it.next('400')` ，先执行了红色区域内代码，这里没有接收参数400的语句，是无效的，程序先打印出100，再执行yield，next返回的{value:xxx,done:bool}对象中的value值，即yield后面跟着的值，此时迭代没有执行完，done为false，所以接着打印出{ value: '200', done: false }
- 第二次执行`it.next('500')` ,执行红色和蓝色区间内的代码，yield 前面的等号前的变量，就是接收这次next传进来的参数的，也就是说a等于500，yield的输出同里，会输出{ value: 300, done: false }
- 第三次执行`it.next('600')`,到了蓝色和黑色区间内的代码，b接收next参数600，最后return b，会将b的值作为value,同时迭代器执行完毕，done为true,所以返回结果为 { value: '600', done: true }
- 第四次执行`it.next('700')`，已经没有接收next参数的地方，也没有return,即value为undefined,done仍然是完成，返回{ value: undefined, done: true }

Generator函数有多种理解角度。从语法上，可以把它理解成一个状态机，封装了多个内部状态。

执行Generator函数会返回一个遍历器对象，也就是说，Generator函数除了状态机，还是一个遍历器对象生成函数。

### generator用来与promise搭配使用 将异步回调看起来变成同步模式

> 先来看一个读取文件的例子

读取文件1.txt中的内容content1，content1又是一个文件的路径，继续读取文件content1中的内容content2，并返回结果

```
function read(path) {
    return new Promise(function (resolve, reject) {
        require('fs').readFile(path, 'utf8', function (err, data) {
            if (err) reject(err);
            resolve(data);
        })
    })
}
function* r() {
    let content1 = yield read('1.txt', 'utf8');
    let content2 = yield read(content1, 'utf8');
    return content2;
}
let it = r();
it.next().value.then(function(data1){
    it.next(data1).value.then(function(data2){
        console.log(data2);
        console.log(it.next(data2).value);
    })
})
复制代码
```

先将异步readFile方法promise化,即调用read方法会得到一个执行readFile方法的promise;
Generator生成器函数依次输出（yield）各文件的内容，最后return返回
使用时，先得到迭代器对象it，第一次调用next()得到的value即为读取1.txt的promise,既然是promise，调用其then方法处理成功回调，data1就是1.txt读取成功时返回的内容，将data1作为下一个next的参数，即content1这时就是data1，同理next返回的value是读取content1文件的promise，所以data2就是成功读取content1的返回值

上例可以简化read函数，利用一些库提供的promisify可以直接将函数promise化，如

```
let bluebird = require('bluebird');
let fs = require('fs');
let read = bluebird.promisify(fs.readFile);
复制代码
```

但是上面不停的调用next()方法，并容易形成嵌套，也是我们希望简化的
这里使用co库，来帮我们自动的将generator迭代

### co库

安装： npm install co 上例可以简化为

```
let bluebird = require('bluebird');
let fs = require('fs');
let co = require('co');
 
let read = bluebird.promisify(fs.readFile);
function* r() {
    let content1 = yield read('1.txt', 'utf8');
    let content2 = yield read(content1, 'utf8');
    return content2;
}
co(r()).then(function (data) {
    console.log(data)
})
复制代码
```

那么co库是怎么实现自动迭代的呢，基于上例这里简单实现一下

```
function co(it) {
    return new Promise(function (resolve, reject) {
        function step(d) {
            let { value, done } = it.next(d);
            if (!done) {
                value.then(function (data) { // 2,txt
                    step(data)
                }, reject)
            } else {
                resolve(value);
            }
        }
        step();
    });
}
复制代码
```

首先从用法可以看出，co执行会返回一个promise，用then注册成功/失败回调，所以先return 一个promise
co将迭代器it作为参数，这里每调用一次step，就执行一次next
既然是自动执行，那么promise的executor中先执行一次it.next()方法,返回value和done；value是一个pending的Promise;如果done=false，说明还没有走完，继续在value.then的成功回调中执行下一次next，即调用step方法；直到done为true，走完所有代码，调用resolve;中间有任何一次next异常，直接调用reject，停止迭代