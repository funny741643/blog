# Async +Await

 js异步处理的方法 async await( 即Generator的语法糖)

async 是“异步”的简写，async 用于申明一个 function 是异步的，而 await 用于等待一个异步方法执行完成，await 只能出现在 async 函数中

同样用上一篇中读取文件的例子,这里改写为

```javascript
let bluebird = require('bluebird');
let fs = require('fs');
let read = bluebird.promisify(fs.readFile);
//await 命令后面的 Promise 对象，运行结果可能是 //rejected，所以最好把 await 命令放在 try...catch 代码块中。

async function r(){
    try{
        let content1 = await read('1.txt','utf8');
        let content2 = await read(content1,'utf8');
        return content2;
    }catch(e){ 
        console.log('err',e)
    }
}

r().then(function(data){
    console.log('data',data);
},function(err){
    console.log('err1',err);
})
复制代码
```

async await和generator的写法很像，就是将 Generator 函数的星号（*）替换成 async，将 yield 替换成await

但async 函数对 Generator 函数做了改进：

1、内置执行器：Generator函数的执行必须靠执行器，所以才有了 co 函数库，而 async 函数自带执行器.也就是说，async 函数的执行，与普通函数一模一样。

2、更好的语义：async 和 await，比起星号和 yield，语义更清楚了。async 表示函数里有异步操作，await 表示紧跟在后面的表达式需要等待结果。

3、更广的适用性： co 函数库约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面，可以跟 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）

> async 函数是非常新的语法功能，新到都不属于 ES6，而是属于 ES7。目前，它仍处于提案阶段，但是转码器 Babel 和 regenerator 都已经支持，转码后就能使用。

### async 的作用

async 函数负责返回一个 Promise 对象
 如果在async函数中 return 一个直接量，async 会把这个直接量通过Promise.resolve()  封装成 Promise 对象;
 如果 async 函数没有返回值,它会返回 Promise.resolve(undefined)

### await 在等待什么

> 一般我们都用await去等带一个async函数完成，不过按语法说明，await 等待的是一个表达式，这个表达式的计算结果是 Promise 对象或者其它值，所以，await后面实际可以接收普通函数调用或者直接量

如果await等到的不是一个promise对象，那跟着的表达式的运算结果就是它等到的东西；
 如果是一个promise对象，await会阻塞后面的代码，等promise对象resolve，得到resolve的值作为await表达式的运算结果
 虽然await阻塞了，但await在async中，async不会阻塞，它内部所有的阻塞都被封装在一个promise对象中异步执行

### Async Await使用场景

如上面的例子，当需要用到promise链式调用的时候，就体现出Async Await的优势；

假设一个业务需要分步完成，每个步骤都是异步的，而且依赖上一步的执行结果，甚至依赖之前每一步的结果，就可以使用Async Await来完成

```javascript
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}
function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m + n);
}
function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k + m + n);
}

async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();
复制代码
```

如果用promise来实现

```javascript
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();
复制代码
```

可见用promise，参数传递非常麻烦

下面的例子，指定多少毫秒后输出一个值。

```javascript
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value)
}

asyncPrint('hello world', 50);
复制代码
```

### 注意

await 命令后面的 Promise 对象，运行结果可能是 rejected，所以最好把 await 命令放在 try...catch 代码块中，或者await后的Promise添加catch回调

```javascript
await read('1.txt','utf8').catch(function(err){
    console.log(err);
})
复制代码
```

await 只能出现在 async 函数中,如果用在普通函数，就会报错

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  // 报错
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
复制代码
```

上面代码会报错，因为 await 用在普通函数之中了。但是，如果将 forEach 方法的参数改成 async 函数，也有问题

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  // 可能得到错误结果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
复制代码
```

上面代码可能不会正常工作，原因是这时三个 db.post 操作将是并发执行，也就是同时执行，而不是继发执行。正确的写法是采用 for 循环。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  for (let doc of docs) {
    await db.post(doc);
  }
}
复制代码
```

如果确实希望多个请求并发执行，可以使用 Promise.all 方法。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}

复制代码
```

### 总结

使用 async / await, 搭配 promise, 可以通过编写形似同步的代码来处理异步流程, 提高代码的简洁性和可读性。

Async Await 的优点： 1、解决了回调地狱的问题
 2、支持并发执行
 3、可以添加返回值 return xxx;
 4、可以在代码中添加try/catch捕获错误



作者：WZZ41998
链接：https://juejin.im/post/6844903586481209358
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。