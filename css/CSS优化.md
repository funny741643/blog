# CSS性能优化

## 内联首屏关键CSS （Critical CSS）

**首次有效渲染**：页面的首要内容出现在屏幕的时间。

**内联css和外部css区别**：

* 内联css能够使浏览器开始页面渲染的时间提前，其在**HTML下载完成后就能渲染了**
* 使用外部CSS文件时，需要在HTML文档下载完成后才知道所要引用的CSS文件，然后才下载它们。

**是否可以内联所有的CSS**：

因为初始拥塞窗口存在限制（通常是14.6kb,压缩后大小），如果内联css后的文件超出了这一限制，系统就需要在服务器和浏览器之间进行更多次的往返，这样并不能提前页面渲染时间。

**内联css的缺点**：

内联之后的css不会进行缓存，每次都会重新下载。但如果我们将内联后的文件大小控制在14.6kb之内，似乎并不是什么大问题。

## 异步加载CSS

CSS会阻塞渲染，在CSS文件请求、下载、解析完成之前，浏览器将不会渲染任何已处理的内容。有时，这种阻塞是必须的，因为我们并不希望在所需的CSS加载之前，浏览器就开始渲染页面。将首屏关键CSS内联后，剩余的CSS内容的阻塞渲染就不是必需的了，可以使用外部CSS，并且**异步加载**。

### CSS如何实现异步加载

* Js动态创建样式表link元素，并插到DOM中。

```javascript
const myCSS = document.createElement( "link" );
myCSS.rel = "stylesheet";
myCSS.href = "mystyles.css";
// 插入到header的最后位置
document.head.insertBefore( myCSS, document.head.childNodes[ document.head.childNodes.length - 1 ].nextSibling );
```

* 将link元素的media属性设置为用户浏览器不匹配的媒体类型

如`media="print"`，甚至可以是完全不存在的类型`media="noexist"`。对浏览器来说，如果样式表不适用于当前媒体类型，其优先级会被放低，会在不阻塞页面渲染的情况下再进行下载。

当然，这么做只是为了实现CSS的异步加载，别忘了在文件加载完成之后，将`media`的值设为`screen`或`all`，从而让浏览器开始解析CSS。

```html
<link rel="stylesheet" href="mystyles.css" media="noexist" onload="this.media='all'">
```

* 通过`rel`属性将`link`元素标记为`alternate`可选样式表，也能实现浏览器异步加载。

```html
<link rel="alternate stylesheet" href="mystyles.css" onload="this.rel='stylesheet'">
```

* rel="preload"

  现在，[rel=”preload”](https://www.w3.org/TR/preload/)5这一Web标准指出了如何异步加载资源，包括CSS类资源。

  ```html
  <link rel="preload" href="mystyles.css" as="style" onload="this.rel='stylesheet'">
  ```

  注意，`as`是必须的。忽略`as`属性，或者错误的`as`属性会使`preload`等同于`XHR`请求，浏览器不知道加载的是什么内容，因此此类资源加载优先级会非常低。

**preload和上两种方法的不同点**：

使用preload，比使用不匹配的`media`方法能够更早地开始加载CSS。

## 文件压缩

性能优化时有一个最容易想到，也最常使用的方法，那就是文件压缩，这一方案往往效果显著。

文件的大小会直接影响浏览器的加载速度，这一点在网络较差时表现地尤为明显。相信大家都早已习惯对CSS进行压缩，现在的构建工具，如webpack、gulp/grunt、rollup等也都支持CSS压缩功能。压缩后的文件能够明显减小，可以大大降低了浏览器的加载时间。

## 去除无用的CSS

如果压缩后的文件仍然超出了预期的大小，我们可以试着**找到并删除代码中无用的CSS**。

一般情况下，会存在这两种无用的CSS代码：一种是不同元素或者其他情况下的重复代码，一种是整个页面内没有生效的CSS代码。

## 有选择的使用选择器

css选择器的匹配是从右向左的，这一策略导致了不同种类的选择器之间的性能也存在差异。相比于`#markdown-content-h3`，显然使用`#markdown .content h3`时，浏览器生成渲染树（render-tree）所要花费的时间更多。因为后者需要先找到DOM中的所有`h3`元素，再过滤掉祖先元素不是`.content`的，最后过滤掉`.content`的祖先不是`#markdown`的。试想，如果嵌套的层级更多，页面中的元素更多，那么匹配所要花费的时间代价自然更高。

**使用选择器时的关注点**

* 保持简单，不要使用嵌套过多过于复杂的选择器。
* 通配符和属性选择器效率最低，需要匹配的元素最多，尽量避免使用。
* 不要使用类选择器和ID选择器修饰元素标签，如`h3#markdown-content`，这样多此一举，还会降低效率。
* 不要为了追求速度而放弃可读性与可维护性。

**为什么css选择器是从右向左匹配的**？

> CSS中更多的选择器是不会匹配的，所以在考虑性能问题时，需要考虑的是如何在选择器不匹配时提升效率。从右向左匹配就是为了达成这一目的的，通过这一策略能够使得CSS选择器在不匹配的时候效率更高。这样想来，在匹配时多耗费一些性能也能够想的通了。

## 减少使用昂贵的属性

在浏览器绘制屏幕时，**所有需要浏览器进行操作或计算的属性相对而言都需要花费更大的代价**。当页面发生重绘时，它们会降低浏览器的渲染性能。所以在编写CSS时，我们应该尽量减少使用昂贵属性，如`box-shadow`/`border-radius`/`filter`/透明度/`:nth-child`等。

## 减少重排和重绘

### 重排

重排会导致浏览器重新计算整个文档，重新构建渲染树，这一过程会降低浏览器的渲染速度。如下所示，有很多操作会触发重排，我们应该避免频繁触发这些操作。

1. 改变`font-size`和`font-family`
2. 改变元素的内外边距
3. 通过JS改变CSS类
4. 通过JS获取DOM元素的位置相关属性（如width/height/left等）
5. CSS伪类激活
6. 滚动滚动条或者改变窗口大小

此外，我们还可以通过[CSS Trigger](https://csstriggers.com/)15查询哪些属性会触发重排与重绘。

值得一提的是，某些CSS属性具有更好的重排性能。如使用`Flex`时，比使用`inline-block`和`float`时重排更快，所以在布局时可以优先考虑`Flex`。

### 重绘

当元素的外观（如color，background，visibility等属性）发生改变时，会触发重绘。在网站的使用过程中，**重绘是无法避免的**。不过，浏览器对此做了优化，它会将多次的重排、重绘操作合并为一次执行。不过我们仍需要**避免不必要的重绘**，如页面滚动时触发的hover事件，可以在滚动的时候禁用hover事件，这样页面在滚动时会更加流畅。

此外，我们编写的CSS中动画相关的代码越来越多，我们已经习惯于使用动画来提升用户体验。我们在编写动画时，也应当参考上述内容，减少重绘重排的触发。除此之外我们还可以通过[硬件加速](https://www.sitepoint.com/introduction-to-hardware-acceleration-css-animations/)16和[will-change](https://drafts.csswg.org/css-will-change/)17来提升动画性能。

## 不要使用@import

使用@import引入CSS会影响浏览器的并行下载。**使用@import引用的CSS文件只有在引用它的那个css文件被下载、解析之后，浏览器才会知道还有另外一个css需要下载，这时才去下载，然后下载后开始解析、构建render tree等一系列操作**。这就导致浏览器无法并行下载所需的样式文件。

