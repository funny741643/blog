# 简单了解setState
setState简单理解便是React用于管理state的一个重要的方法
## setState官方用法：
1. 语法1： setState(updater[, callback])
	* updater：函数类型，返回一个更新后的 state 中的状态对象，它会和 state 进行浅合并。
    * callback： 可选，回调函数。
```
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

2. 语法2： setState(stateChange[, callback])
    * stateChange: 对象类型，会将传入的对象浅层合并到新的 state 中。
    * callback： 可选，回调函数。
```
// 官方不推荐这种方法，因为 this.props 和 this.state 可能会异步更新，所以不要依赖他们的值来更新下一个状态，下面会详细说明。
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
关于setState的第二个参数： 

第二个参数是一个回调函数，该函数会在setState函数调用完成并且组件开始重渲染的时候被调用，我们可以用该函数来监听渲染是否完成。

## 使用setState的一些关键点

### react为什么不是直接在this.state中更改呢？
```
// 更新状态
this.setState({count: count + 1})
// 无意义
this.state.count = count + 1
```
这是因为state到底只是一个对象，单纯去修改一个对象的值是无意义的，而驱动UI的更新这才是我们最需做的。我们可以想想如果我们只是更改了this.state的值，而没有让React组件重新绘制，那又有什么意义呢？

这时候我们便需要一个函数既可以更新state的状态，又可以驱动组件的更新过程，这个函数便是setState。

### setState并不会立即改变this.state的值
```
function incrementMultiple() {
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
  this.setState({count: this.state.count + 1});
}
```
在我们看来当incrementMultiple被调用时，组件状态的count值应该被增加了3次，每一次调用setState都会增加一次count的值，实际上count的值只被增加了一次，这是因为调用this.setState时，并没有立即更改this.state，所以this.setState只是在反复设置同一个值而已。

### setState何时修改this.state的值
**setState会通过引发一次组件的更新过程来引发重新绘制**
setState调用引起的React的更新生命周期函数4个函数
* shouldComponentUpdate
* componentWillUpdate
* render
* componentDidUpdate

当shouldComponentUpdate和componentWillUpdate被调用时this.state还没有得到更新，直到render被调用时，this.state才得到更新。

或者到shouldComponentUpdate返回false时，这时候更新过程就被中断了，render函数也不会被调用了。但是React并不会放弃掉对this.state的更新的，所以虽然不调用render，依然会更新this.state。

### 多次setState函数调用产生的效果会合并
当你的state包括几个独立的变量
```
constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```
你可以分别调用setState单独更新他们的状态
```
 componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
}
```
连续调用了两次this.setState，但是只会引发一次更新生命周期，不是两次，因为React会将多个this.setState产生的修改放在一个队列里，攒在一起，觉得差不多了再引发一次更新过程。因为如果每一个this.setState都引发一个更新过程的话，那就太浪费了！

### 可以向setState里面传递一个函数updater
这个函数会接收到两个参数，第一个是当前的state值，第二个是当前的props，这个函数应该返回一个对象，这个对象代表想要对this.state的更改。但是对这个对象的计算方法有些改变，它不再依赖于this.state，而是依赖于输入参数state。

现在对上面重新写一下上面增加state上count的例子：
```
function increment(state, props) {
  return {count: state.count + 1};
}
```
可以看到，同样是把状态中的count加1，但是状态的来源不是this.state，而是输入参数state。
```
function incrementMultiple() {
  this.setState(increment);
  this.setState(increment);
  this.setState(increment);
}
```
对于多次调用函数式setState的情况，React会保证调用每次increment时，state都已经合并了之前的状态修改结果。

简单说，加入当前this.state.count的值是0，第一次调用this.setState(increment)，传给increment的state参数是0，第二调用时，state参数是1，第三次调用是，参数是2，最终incrementMultiple的效果，真的就是让this.state.count变成了3。

值得一提的是，在increment函数被调用时，this.state并没有被改变，依然，要等到render函数被重新执行时（或者shouldComponentUpdate函数返回false之后）才被改变。

# setState异步与同步的问题
在上面文章中，我们可以了解到setState是异步更新组件状态的，即在调用完setState后，React组件需要进行重新渲染，在render之后，this.state的状态值才有所改变。但是setState会不会进行同步更新组件，即当setState函数返回的时候，this.state已经体现了状态的改变。这个答案是可以的。

> **在React中，如果是由React引发的事件处理（比如通过onClick引发的事件处理），调用setState不会同步更新this.state，除此之外的setState调用会同步执行this.state。**

“除此之外”指的是什么呢？就是指绕过React通过addEventListener直接添加的事件处理函数（如原生js绑定事件），还有通过setTimeout/setInterval产生的异步调用。

如果我们规范使用React基本不会触及“除此之外”的情况。我们大部分开发中用到的都是React封装的事件，比如onChange、onClick、onTouchMove等，这些事件处理程序中的setState都是异步处理的。

## 举个栗子
```
constructor() {
  this.state = {
    count: 10
  }
  this.handleClickOne = this.handleClickOne.bind(this)
  this.handleClickTwo = this.handleClickTwo.bind(this)
}

render() {
  return (
    <button onClick={this.hanldeClickOne}>clickOne</button>
    <button onClick={this.hanldeClickTwo}>clickTwo</button>
    <button id="btn">clickTwo</button>
  )
}

handleClickOne() {
  this.setState({ count: this.state.count + 1})
  console.log(this.state.count)
}

```
输出： 10

由此可以看出React封装的事件处理程序中的setState是异步更新state的。

```
componentDidMount() {
  document.getElementById('btn').addEventListener('clcik', () => {
    this.setState({ count: this.state.count + 1})
    console.log(this.state.count)
  })
}
```
输出： 11

```
handleClickTwo() {
  setTimeout(() => {
    this.setState({ count: this.state.count + 1})
    console.log(this.state.count)
  }, 10)  
}
```
输出：11

这两种方式绕过来React,通过js的事件绑定程序 addEventListener 和使用setTimeout/setInterval的情况下，setState是同步更新state的。

## React是怎样控制异步和同步的呢？

在 React 的 setState 函数实现中，会根据一个变量 **isBatchingUpdates** 判断是直接更新 this.state 还是放到队列中延时更新，而 isBatchingUpdates 默认是 false，表示 setState 会同步更新 this.state；但是，有一个函数 **batchedUpdates**，该函数会把 isBatchingUpdates 修改为 true，而当 React 在调用事件处理函数之前就会先调用这个 batchedUpdates将isBatchingUpdates修改为true，这样由 React 控制的事件处理过程 setState 不会同步更新 this.state。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27fd5e7bd0b345409bd736d3ce81ae47~tplv-k3u1fbpfcp-zoom-1.image)

## React为何将setState设置为异步更新state
可以参考一下[setState为什么不会同步更新组件状态](https://zhuanlan.zhihu.com/p/25990883)这篇文章

# 参考文章
[setState：这个API设计到底怎么样](https://zhuanlan.zhihu.com/p/25954470)

[setState何时同步更新状态](https://zhuanlan.zhihu.com/p/26069727)

[React 中setState更新state何时同步何时异步？](https://www.jianshu.com/p/799b8a14ef96)