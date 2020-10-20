## 背景React15

### react核心思想：

内存中维护一颗虚拟DOM树，数据变化时（setState），自动更新虚拟DOM，得到一颗新树，然后diff新老虚拟DOM树，找到有变化的部分，得到一个change(patch)，将这个patch加入队列，最终批量更新这些path到DOM中。简单说就是：diff + patch。

### react 执行render()和setState()进行渲染时主要有两个阶段：

调度阶段 （Reconciler）: 用新数据生成一颗新树，遍历虚拟dom，diff新老virtual dom树，搜集具体的UI差异，找到需要更新的元素，放到更新队列中。
渲染阶段（Renderer）: 遍历更新队列，通过调用宿主环境的API，实际更新渲染对应元素。宿主环境，比如dom,native等。

### 3种实例

1.DOM， 对应真实的DOM节点
2.Element 描述UI，通过React.createElement()得到
3.Instance React维护的虚拟DOM, 根据Element创建，对组件和DOM节点的抽象表示，维护组件内部状态和与DOM树的关系

### 优化实践

react本身为了提高页面渲染性能，推出了一些最佳实践
1.vdom 减少对dom的直接操作
2.无状态组件 减少组件内部状态和复杂度
3.shouldComponentUpdate 减少diff的次数
4.immutable 减少diff的成本
但以上都是针对js执行而提出的方法，具体到浏览器渲染，如何避免长时间的线程占用并没有给出好的建议。

## why Fiber的来源

Fiber之前的Reconciler阶段采用的是Stack Reconciler， 其自顶向下遍历vdom tree, 递归组件执行任务，过程无法中断。
假设有一个层级很复杂的组件，在顶层组件内执行setState, 那么调用栈可能会很长。由于调用栈过长，中间可能还有一些复杂操作，这些任务无法中断，就导致主线程被长时间阻塞。由于浏览器里渲染和js执行共一个主线程，在对响应要求高的场景，比如手势，动画等，就容易造成卡顿，延迟等现象，从而影响用户体验。

Fiber的出现就是为了解决这个问题。
Fiber 的解决思路：把渲染更新过程拆分成多个子任务，每次只做一小部分，做完看是否还有剩余时间，如果有继续下一个任务；如果没有，挂起当前任务，将时间控制权交给主线程，等主线程不忙的时候在继续执行。

## what Fiber是什么

计算机通常追踪程序执行的方式是使用调用堆栈。执行函数时，不断的把堆栈帧加入到堆栈中，一个堆栈帧就表示一个要执行的工作。
在处理UI时，如果执行太多的工作，就可能导致动画丢帧。

新版本的浏览器实现了有助于解决这个问题的API：requestIdleCallback 和 requestAnimationFrame.
requestIdleCallback 调度在空闲期间调用的低优先级函数
requestAnimationFrame调度在下一个动画帧上调用的高优先级函数。

问题是，为了使用这些API，就需要一种方法将渲染工作分解为增量单元。如果仅依赖于调用堆栈，它将继续工作直到堆栈为空，无法中断。
为了实现增量渲染的调度，就必须重新实现这个堆栈帧，以便可以将堆栈帧保留在内存中，然后按照自己的调度算法执行他们。同时由于这些堆栈栈是我们手动处理的，我们还可以加入并发或者错误边界等功能。

因此Fiber就是重新实现的堆栈帧，本质上Fiber也可以理解为是一个虚拟的堆栈帧，将可中断的任务拆分成多个子任务，通过按照优先级来自由调度子任务，分段更新，从而将之前的同步渲染改为异步渲染。
react内部有自己的优先级判断逻辑，比如动画，用户交互等任务优先级就明显要高。



### Fiber的目标

- 将耗时长可中断的任务拆分成多个子任务
- 对正在做的工作调整优先级，可以重做，复用上次的结果

### Fiber特性

- 增量渲染，把一个渲染任务拆分成多个子任务，平均到多个渲染帧中执行，每次只做一小段，做完后就把时间控制权上交给主线程
- 在渲染更新时，能够暂停，复用任务
- 不同类型的任务具有不同的优先级
- 并发方面的其他能力
- 错误边界

## how Fiber实现

简单来说就是，**时间分片 + 链表结构** 。
fiber就是维护每一个分片的数据结构。



### fiber && fiber tree

react中没有明确的Virtual Dom， 可以把fiber理解为我们习惯上的虚拟Dom概念。

react在render中第一次渲染时，利用React.createElement会创建一棵Element树，同时会利用Element中的数据创建Fiber tree。不同的Element类型对应不同类型的Fiber Node。在后续的更新过程中（setState），每次重新渲染都会重新创建Element, 但是fiber不会，fiber只会使用对应的Element中的数据来更新自己必要的属性，

一个Fiber Node可以认为是一个对象，它表示组件需要做的工作。一个Element对应一个或多个Fiber Node。

上面提到Fiber要做增量更新，所以就要额外维护一些上下文信息，所以react 扩展出了 fiber tree 的概念，更新过程就是根据输入的数据以及当前fiber tree，构造出新的fiber tree(workInProgress tree)。上面我们提到了Instance, Fiber基于此进行了扩展，添加了一些其他概念：

```
{
    // 搜集diff差异结果，每个workInProgress tree节点都有一个effect list
    // 当前节点更新完毕，会向上merge effect list
    effect

    // reconcile过程中的快照，工作过程树，类似于“草稿”，用户不可见
    workInProgress

    // fiber tree 与vdom tree类似，描述增量更新需要的上下文信息
    fiber
}
```

```
// 假设有一个<Card />组件

// FiberNode 结构如下：
{
    // 定义fiber节点类型，类组件指向构造函数，dom元素指向标签名称
    type: Card,

    // Fiber类型，将React Element映射成对应的Fiber类型，用于说明协调过程中需要完成的工作
    // HostRoot|HostComponent|ClassComponent|FunctionComponent...
    tag: 1,

    // 不同tag代表不同类型的副作用
    effectTag: 1,
    firstEffect: null,
    lastEffect: null,

    // 单链表结构，方便遍历fiber树上有副作用的节点
    nextEffect: FiberNode|null,

    // 第一个子fiber
    child: FiberNode|null,

    // 指向父fiber，表示当前节点处理完毕后，应该向谁提交自己的结果effect list
    return: FiberNode|null

    // 兄弟fiber
    slibing: FiberNode|null,

    // 当前父fiber中的位置
    index: 0,

    // fiber实例对象，指向当前组件实例
    stateNode: Card,

    // setState待更新状态，回调，DOM更新的队列
    updateQueue: null,

    // 当前UI的状态，反映了UI当前在屏幕上的表现状态
    memoizedState: {},

    // 前次渲染中用于决定UI的props
    memoizedProps: {},

    // 即将应用于下一次渲染更新的props
    pendingProps: {},

    // 和组件Element中的key,ref一致
    key: null,
    ref: null,

    // fiber更新时基于当前fiber克隆出的镜像，更新时记录两个fiber diff的变化；更新结束后alternate替换之前的fiber成为新的fiber节点
    alternate: {},

    // 标记子树上待更新任务的优先级 （最新版的react做了变更，改由过期时间实现，时间越大，setState越频繁，优先级就越高）
    pendingWorkPriority: number
}
```

从上述fiber数据结构，可以看出fiber tree 是一个链表结构，通过child, slibing, return完成结构关联。

### current tree && workInProgress tree

在第一次渲染(didMount)之后，React将得到一个 Fiber 树，它反映了当前UI的工作状态。这棵树通常被称为 current 树（当前树）。当 React 开始处理更新时，它会构建一个所谓的 workInProgress 树（工作过程树），它反映了要刷新到屏幕的未来状态。

所有的工作都在 workInProgress 树的 Fiber 节点上执行。当 React 遍历 current 树时，对于每个现有 Fiber 节点，React 会创建一个构成 workInProgress 树的备用节点，该节点会使用 render 方法返回的 ReactElement 中的数据来创建。处理完更新并完成所有相关工作后，React 将工作完成的备用树workInProgress刷新到屏幕上。一旦这个 workInProgress 树在屏幕上输出，它就会变成 current 树。

workInProgress 树可以理解为一个工作快照，或者“工作草稿”，一般用户不可见。对React来说就是不会显示更新渲染的中间过程，React先处理所有组件，然后将其一次性更新到屏幕上。

### 副作用 && 副作用列表

更新完成后可能要调用声明周期方法，更新ref，或者执行其他方法等等，这些都称之为“副作用”。副作用定义了在组件更新后需要为组件实例完成的 相关工作。不同类型的组件其副作用各不相同。比如一个DOM元素的副作用和类组件的副作用就不一样。

副作用列表：收集具有副作用的Fiber节点，从而后续能够快速遍历线性列表，执行副作用。firstEffect指针指向列表的开始位置，不同节点间通过nextEffect串联顺序。一般React会按从子节点到父节点的顺序逐个执行副作用。

## Fiber Reconciler

reconciler过程分为两个阶段:

### Reconciliation（也叫render）

> 目的：确定需要在UI中更新的内容
> 代码实质：得到标记了副作用的Fiber节点树。副作用描述了在下一个commit阶段需要完成的工作。

这一过程可中断。事实上React通过时间分片的方式来处理一个或多个Fiber节点，从而赋予对正在做的工作以暂停，恢复，撤销重做的能力。这一阶段的工作对用户始终不可见。

将每个fiber节点作为最小工作单位，通过自顶向下逐个遍历fiber node，构造workInProgress tree(一颗新的fiber tree), 得到patch结果。
这一过程总是从顶层的HostRoot节点开始遍历，但React会跳过那些已经处理过的Fiber节点，直到找到未完成工作或者需要处理的节点。源码中的入口函数是renderRoot。

具体过程如下：

```
1. 如果当前节点不需要更新，直接clone 子节点， 跳到步骤5；如果需要更新，则修改tag，记录更新类型
2. 更新当前节点状态，context，props，state等
3. 调用shouldComponentUpdate, 如果返回值为false，则跳转步骤5
4. 调用组件实例的render方法，得到一个新的子节点，同时为子节点创建Fiber Node（会尽量复用现有fiber，子节点的增删也发生在这里）
5. 如果没有产生child fiber, 该工作单元结束，把effect list归并到return上，并把当前FiberNode的sibling作为下一个工作单元。如果有child fiber，将child指向作为下一个工作单元。
6. 检查有没有剩余时间，如果有继续执行下一个工作单元；如果没有，等到下一次主线程空闲时再开始执行下一个工作单元
7. 如果没有下一个工作单元，回到workInProgress tree 根节点，reconciliation节点结束，进入pendingCommit状态。
```



从上述过程可以看出，1-6的实际执行逻辑其实是一个work loop，每执行完一次loop，都要检查有没有剩余时间，进行控制权的交换。
由于每做完一个loop，都要把effect list向上归并到return，因此等到loop结束时，workInProgress根节点上的effect list就是收集到的所有effect。

**构建workInProgress tree的过程就是diff的过程**。
通过requestIdleCallback来调度执行一个任务，每完成一个任务，都回来检查下有没有优先级更高的任务。每完成一个任务，都要把时间控制权交换给主线程，直到下一个requestIdleCallback回调再继续构建workInProgress tree.(requestIdleCallback本身有兼容性问题，react团队通过MessageChannel + requestAnimationFrame实现了requestIdleCallback的效果)。

PS.Fiber之前的Reconciler被叫做Stack Reconciler，就是因为这些调度上下文信息是由系统堆栈来保存的，以便和Fiber Reconciler区分开。

这一阶段执行的生命周期方法有：
componentWillMount、
componentWillReceiveProps、
shouldComponentUpdate、
componentWillUpdate

由于Reconciliation阶段是可中断的，因此处在这一阶段的生命周期钩子函数可能被多次调用，存在副作用，从而引起bug。所以版本16之后会逐渐废除掉这些API（不包括scu函数）

但 componentWillReceiveProps 和 componentWillUpdate 在实际业务场景中比较有用，所以16新增了两个API static getDerivedStateFromProps 和 getSnapshotBeforeUpdate 用以解决相同场景下的业务问题。

### Commit

> 目的：更新UI，对DOM应用上一个过程得到的patch结果。
> 代码实质：已经得到了标记了副作用的的Fiber节点树，通过遍历副作用列表，根据副作用类型执行具体的副作用，包括DOM更新，生命周期函数调用，ref更新等一系列用户可见的UI变化。

副作用类型:

```
import {
  NoEffect,
  PerformedWork,
  Placement,            // 挂载，didMount
  Update,               // 更新, didUpdate
  Snapshot,             // getSnapshotBeforeUpdate，更新之前设置快照
  PlacementAndUpdate,
  Deletion,             // 卸载，willUnmount
  ContentReset,
  Callback,
  DidCapture,
  Ref,
  Incomplete,
  HostEffectMask,
  Passive,
} from 'shared/ReactSideEffectTags';
```



进入commit阶段时，react从上一阶段得到了两棵树和一个副作用列表。current树反应当前屏幕上UI的状态，finishedWork反映未来需要映射到屏幕上UI的状态。副作用列表来描述需要实际做的操作，比如dom更新，增删，调用生命周期函数等等。因此严格来说，副作用列表应该是finishedWork树的子集。

这一阶段的工作会导致用户可见的变化，比如DOM更新。因此该过程不可中断，必须一直执行直到更新完成。

根据副作用类型，执行工作：

```
1.副作用类型为Snapshot, 则执行getSnapshotBeforeUpdate生命周期；Deletion类型，执行componentWillUnmount生命周期
2.执行DOM更新
3.将 finishedWork 树设置为 current
4.副作用为Placement类型执行componentDidMount生命周期；Update类型执行componentDidUpdate生命周期
5.其他钩子
```



这一阶段执行的生命周期方法有：
getSnapshotBeforeUpdate、
componentDidMount、
componentDidUpdate、
componentWillUnmount

思考：为什么commit(patch)不能拆分？这样意义不大，容易导致react内部维护的dom状态和实际不一致，影响体验