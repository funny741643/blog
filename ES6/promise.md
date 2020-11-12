# Promise的使用

Promise本意“承诺”，是一个“承诺将来会执行”的对象

基本用法

案例1

```javascript
let p = new Promise(function(resolve,reject){
    console.log('立即执行');
    setTimeout(function(){
        resolve('成功');
    },100);
});

p.then(function(data){
    console.log(data);
},function(err){
    console.log(err);
})

复制代码
```

### new Promise(executor)

​    创建Promise对象时，executor执行器会同步执行（在Promise构造函数返回Promise对象之前就会被执行），executor是一个函数，参数resolve执行成功的回调函数，reject执行失败的回调函数，函数内部通常调用用来执行一些异步操作，操作完成后，调用resolve触发promise的成功状态，或者调用reject触发promise的失败状态。

### Promise.prototype.then()

​    then 用来注册Promise状态确定后的回调；then参数 ：处理成功返回结果的函数，处理错误结果的函数 ；then方法返回一个Promise，因此可以继续调用then，即实例化后的Promise对象可以进行链式调用

案例2

```javascript
//读取文件1.txt ,1.txt中存放着文本2.txt，再读取2.txt文件的内容
let fs = require('fs');
function read(path){
    return new Promise(function(resolve,reject){
        fs.readFile(path,'utf8',function(err,data){
            if(err){
                reject(err);
            }
        resolve(data)
        })
    })
}

read('1.txt').then(data=>{
    console.log(data);
    //return data;// 如果返回了一个普通值，会将这个值作为下一次then的成功会回调
    return read(data);
},err=>{
    console.log(err);
}).then(data=>{
    console.log('next then :',data);
},err=>{
    console.log(err);
})复制代码
```

### Promise状态

1、pending:默认状态,就是初始化Promise时，调用executor执行器函数后的状态。

2、fulfilled：完成状态/成功状态 ，异步操作调用resolve()后的状态  

3、rejected:失败状态，异步操作调用reject()后的状态 

 状态转化：pending ->fulfilled ,pending ->rejected 这个状态转化是单向的，不可逆转 

​         也就是一旦进入成功就成功了，一旦进入失败就失败了

**链式调用**返回一个Promise，该Promise的状态主要根据第一个then方法返回的值确定

 1、如果第一个then()方法中返回了一个参数值，那么返回的Promise将会变成接收状态。

 2、如果第一个then()方法中抛出了一个异常，那么返回的Promise将会变成拒绝状态。

 3、如果第一个then()方法调用resolve()方法，那么返回的Promise将会变成接收状态。

 4、如果第一个then()方法调用reject()方法，那么返回的Promise将会变成拒绝状态。

 5、如果第一个then()方法返回了一个未知状态(pending)的Promise新实例，那么返回的新    Promise就是未知状态。 

6、如果第一个then()方法没有明确指定的resolve(data)/reject(data)/return data时，那么返回的新Promise就是接收状态，可以一层一层地往下传递。 

​    不管第一个then走的成功还是失败，都会将返回值（没有返回值，就返回undefined）作为下一个then的结果 

​    除非Promise.then()方法内部抛出异常throw new Error()或者是明确置为rejected态，否则它返回的Promise的状态都是fulfilled态，并且它的状态不受它的上一级的状态的影响

案例3

```javascript
// 如果resolve 里面放的是promise 会将这个promise执行后的结果向下传递
let p1=new Promise(function(resolve,reject){
    resolve(100);
});

let p2 =new Promise(function(resolve,reject){
    resolve(p1);
}); 
let p3 =new Promise(function(resolve,reject){
    resolve(p2);
}); 
p3.then(function(data){
    console.log(data);
});复制代码
```

### **catch**

​    案例2 中每个then都写了错误回调，可以省略掉，在最后写入catch方法 但如果上面有写自己的错误回调，就调用自己的，没写就调用catch

案例4

```javascript
read('1.txt').then(data=>{
    console.log(data);
    return read(data); //返回一个promise
}).then(data=>{
    console.log('next then :',data);
},err=>{
    console.log(err);
}).catch(e=>{
    console.log(e);
})复制代码
```

### Promise.all(promises) 

all 接收一个参数,它必须是可以迭代的，比如数组,它通常用来处理一些并发的异步操作, 是多个promise对象的结果集合

案例5：同时读取两文件，将两个文件的内容拼接在一起

```javascript
//Promise.all方法执行后返回的依旧是promise
//它是并发执行的，返回结果按照执行顺序，即ele2是read('2.txt')的返回结果
//all里面的都执行成功了，才表示成功，有一个失败了，就失败
Promise.all([read('1.txt'),read('2.txt')]).then(([ele1,ele2])=>{
console.log(ele1,ele2);
},err=>{
console.log(err);
});复制代码
```

### Promise.race() 

Promise.race() 和all基本同，race“赛跑”的意思，race的状态变化不是全部受参数内的状态影响，一旦参数内有一个值的状态发生的改变，那么该Promise的状态就是改变的状态

```javascript
// race赛跑 如果先成功了 那就成功了 如果先失败了 那就失败了
Promise.race([read('1.txt'),read('2.txt')]).then(([ele1,ele2])=>{
   console.log(ele1,ele2);
},err=>{
   console.log(err);
});复制代码
```

### Promise.resolve() / Promise.reject()

参数可以是普通值或Promise对象

```javascript
let p = Promise.resolve([1,2]);
p.then(data=>{
    console.log(data);
})
Promise.reject([1,2,3]).then(null,function(err){
    console.log('err',err)
}); 
let p2 = Promise.reject(['fail']);
let p3 = Promise.resolve(p2).then(data=>{
    console.log('不执行');
},err=>{
    console.log('resolve接收rejected状态的promise',err);
});
```