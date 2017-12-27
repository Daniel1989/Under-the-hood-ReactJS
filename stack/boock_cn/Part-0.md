## Part 0

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)

<em>0.0 Part 0 ()</em>

### ReactDOM.render
Alright, let’s start with a call of ReactDOM.render.
让我们从一个ReactDOM.render的调用开始。

入口方法是ReactDOM.render方法，我们的应用在这里把DOM渲染进DOM。为了调试更方便，我创建了一个简单组件`<ExampleApplication/>`. 第一件，**JSX将会被转成React元素**. 代码很简单，基本上就是一个简单结构的简单对象，这些都只是作为组件render方法的返回，没有其他功能。一些简单的关键字你应该很熟悉了，比如props,key,ref. 属性类型是指用JSX描述的标记对象。在我们的例子中，就是类`ExampleApplication`, 当然，它也可以Button 标签的button字符串形式。在React元素的创建期间，React会把`defaultProps`和props属性（如果有指定）进行合并，同时验证其类型，更多详细信息请查看源码
(`src\isomorphic\classic\element\ReactElement.js`)

### ReactMount
You can see the module called `ReactMount` (01), it contains the logic of components mounting. Actually, there is no logic inside `ReactDOM`, it is just an interface to work with `ReactMount`, so when you call `ReactDOM.render` you technically call `ReactMount.render`. What is all that mounting about?
现在看一些`ReactMount`这个模块（01），组件挂载的逻辑都在这里。事实上，`ReactDOM`中没有什么逻辑，它只不过是`ReactMount`的一个接口声明，所以，当你调用`ReactDOM.render`时，从技术上来说，你真正调用是`ReactMount.render`方法。挂载具体是指什么呢？
>挂载，是指通过创建组件代表的DOM元素并插入｀容器｀的初始化React组件的过程。

至少源码中的注视是这么解释的。那么，这到底是什么意思呢？
看一下下图的转换：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)

<em>0.1 JSX 到 HTML (点击查看大图)</em>

React needs to **transform your component(s) description into HTML** to put in into a document.
How to get there? Right, it needs to handle all **props, events listeners, nested components**, and logic. It’s needed to granulate your high-level description (components)  to really low-level data (HTML) which can be put into a web-page. That is all that mounting is about.
React需要 **将组件描述转成到HTML**然后插入到页面文档中。如何做的呢？这个过程需要处理所有的**props,事件监听器，内嵌组件**,逻辑。这需要保证你的高层级的描述(组件)转成层级的数据(HTML)后，能够顺利插入到整个web页面。这就是挂载要做的全部事情。


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)

<em>0.1 JSX 到 HTML, 2 (点击查看大图)</em>

Alright, let’s continue. But… it’s interesting fact time! Yes, let’s add some interesting things during our journey, to have more ‘fun’.
让我们继续。为了让我们探索源码的旅程更有意思，加入一些有意思的事情。

> 有趣的事实：确保所有的滚动都是被监控的(02)
> 有趣的是，在根组件的第一次渲染期间，React会初始化滚动条监听器并且缓存滚动条的值，这样做的目的是为了应用代码可以获取这些值而不触发重排。由于游览器渲染的实现，一些DOM值不是静态的，每次你从代码里使用这些值，他们都会重新计算，这当然会影响到性能。事实上，老的游览器不支持pageX,pageY,而React试图优化这些。正如你所见，为了制造一个使用更快的工具需要使用到一大堆技术，这个就是一个很好的列子。

### 实例化React组件

Look at the scheme, there is an instance creation by number (03). Well, it's too early create an instance of `<ExampleApplication />` here, in fact, we instantiate `TopLevelWrapper` (internal React class).
Let’s check out the next scheme at first.
看一下流程图，这里有一个创建的实例（图中为03）。 对于创建`<ExampleApplication />`，这里还有点早，这里，我们实例化`TopLevelWrapper`（内部React类）.让我们先看下下面的流程图：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)

<em>0.3 JSX 到 VDOM (点击查看大图)</em>

你可以看到这里有三个阶段，JSX通过React元素会转化成以下内部React组件之一：`ReactCompositeComponent`(对应我们自定义的组件),`ReactDOMComponent`（对应HTML标签）,
`ReactDOMTextComponent`（对于文本节点）。我们将忽略
`ReactDOMTextComponent`，然后将精力集中到前两个类型

你也许听过**虚拟DOM**  , 虚拟DOM是一种在React内部在不直接接触DOM情况下，计算不同或其他任务时的DOM表示。这个是React快的原因。但是，事实上，在React源码内部，并没有称为'虚拟DOM'的文件和类。很有趣，是不是？这是因为虚拟DOM只是一种概念，一种操作真实DOM的手段。有人说，虚拟DOM表示React元素，但是在我看来，这个不完全正确。我认为虚拟DOM
表示的是这三种类型：`ReactCompositeComponent`, `ReactDOMComponent`, `ReactDOMTextComponent`.后面你会看到我为什么这么说。

让我们完成我们的实例过程。我们还是很有兴趣知道我们会创建什么实例。 我们会创建一个`ReactCompositeComponent`的实例，但是，这不是因为我们将`<ExampleApplication/>`放到`ReactDOM.render`中。React会从`TopLevelWrapper`开始渲染组件树。这是一个空的包装类，它的`render`（组件的渲染方法）将会在稍后返回`<ExampleApplication/>`。
```javascript
//src\renderers\dom\client\ReactMount.js#277
TopLevelWrapper.prototype.render = function () {
  return this.props.child;
};

```
目前只有`TopLevelWrapper`被创建了。但是这里有个有意思的地方！
>  有意思的事情：验证DOM内嵌
>几乎每一次当内嵌组件渲染时，他们会被一个用来验证HTML，名为`validateDOMNesting`的模块验证。DOM内嵌验证就是说确认`child->parent`标签的继承体系。举个列子，如果父标签是
`<select>`，子标签只能是以下之一`option`,`optgroup`,或者`#text`。这个规则是在这里被定义的：https://html.spec.whatwg.org/multipage/syntax.html#parsing-main-inselect. 你可能看到过这个模块的产出，它产生类似如下的错误:
<em> &lt;div&gt; cannot appear as a descendant of &lt;p&gt; </em>.


### 好了, 我们完成了 *Part 0*.

让我们回想下我们学到的内容。再看一次流程图，然后将不重要的信息移除掉，流程图变成如下：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)

<em>0.4 Part 0 简化版本 (点击查看大图)</em>

然后让我们移除多余的空格并排下版：
[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)

<em>0.5 Part 0 简化&重构 (点击查看大图)</em>

很好，事实上，这就是目前发生的全部了。我们可以完成*Part 0*部分的学习了，从这学到的东西会在最后`挂载`流程图里使用：
[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)

<em>0.6 Part 0 必要信息 (点击查看大图)</em>

完毕！


[下一页: Part 1 >>](./Part-1.md)

[<< 上一页: Intro](./Intro.md)


[主页](../../README.md)
