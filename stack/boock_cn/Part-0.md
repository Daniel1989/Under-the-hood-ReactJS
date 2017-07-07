## 第0部分

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0.svg)

<em>0.0 预览图(点击看大图)</em>

### ReactDOM.render
好了，让我们从ReactDOM.render的调用开始学习。

ReactDom.render就是我们的入口点，我们的程序就是从这里开始把内容渲染进DOM的。为了更方便调试，我创建了简单的组件`<ExampleApplication/>`。现在，程序第一件发生的事情就是**JSX将会被转化成React元素**. 这些元素真的很简单，基本上就是一些带简单结构的对象。这些元素只是代表了组件render方法返回结果，这之上就没有其他任何内容了。一些字段对你来说已经很熟悉了，比如props, key, ref.属性类型代表了用JSX描述的标记语言对象。在我们的例子中，就是类`ExampleApplication`，当然了，标记语言对象也可以就只是一个Button标签的字符串表示等等。同时，在React元素的创建过程中，React会将`defaultProps`和props（如果在代码中有定义的话）合并并验证prop的类型。更多细节请查看源码
(`src\isomorphic\classic\element\ReactElement.js`)

### ReactMount
正如你所见，模块调用`ReactMount`(01),它包含了组件挂载的逻辑。事实上，在`ReactDOM`中并没有什么内在逻辑，它只不过是一个调用`ReactMount`的接口，所以，当你调用`ReactDOM.render`时，技术上来说，你真正调用的是`ReactMount.render`。那么挂载中有什么过程呢？
> 挂载就是初始化一个React组件的过程，其中它会创建组件代表的DOM元素并插入到参数提供的`container`中。

至少代码里的注释就是这样描述的。那么，它真正是意思该如何理解呢？我们来看下一个转化过程，如下：


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-small.svg)

<em>0.1 JSX到HTML(点击看大图)</em>

React需要**转化你组件的描述成HTML**，然后插入到文档里。那么这是如何工作的呢？没错，react需要处理所有的**props, events listeners, nested components**, 还有所有逻辑。
整个过程会将你高层次内容（组件）转成低层次的数据（HTML），以使得能够插入到web页面中。以上就是挂载时发生的所有的事。详细流程如下：


[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/mounting-scheme-1-big.svg)

<em>0.1 JSX转HTML，扩充版（点击看大图）</em>

我们继续。嗯。。。，接下去就是有趣的时间了，让我们加一些有趣的东到代码中让我们的react之旅有更多乐趣。

> 有趣的事：确保所有滚动事件都被监听（02）

> 有意思的事：在根组件的首次渲染期间，React会初始化滚动条同时缓存滚动条的值，这是为了应用代码能在不触发页面重绘的基础上访问到滚动条的相关值。由于游览器的渲染实现，一些DOM值不是固定的，而是在每次代码使用的过程中会重新计算它们，当然，这会影响到性能。然而，这个只会对不支持pageX，pageY属性的老游览器有影响，而React就是为了优化这些。正如你所见，制作一个快速的工具要求一大堆技术的使用，而滚动条就是其中一个很好的列子。

### 实例化React组件

看一下模式图，图3是一个实例化的过程图。看的出来，真正实例化一个
`<ExampleApplication/>`之前有一些其他步骤，事实上，我们会先实例化`TopLevelWrapper`（内部React类）.
先看下图：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/jsx-to-vdom.svg)

<em>0.3 JSX 到 虚拟DOM (点击看大图)</em>

这里主要是三个步骤，JSX通过React元素会被转成以下React组件类型之一：
`ReactCompositeComponent`(我们自定义的组件)
`ReactDOMCommponent`(HTML标签)
`ReactDOMTextComponent`(文本节点)
我们暂且忽略`ReactDOMTextComponent`，主要分析下前两个。

内部组件？听起来很有意思。你应该已经听说过**虚拟DOM**,虚拟DOM是一种DOM节点的表现形式，主要被React用来在不直接访问DOM节点的前提下计算节点差异等等。这也是React“快”的原因之一。但是事实上呢，在React源码内部，根本没有额外的称为虚拟DOM的文件或类。
很有趣，是不是？这是因为虚拟DOM只是一种概念，一种操作真实DOM节点的方法。所以呢，有人说虚拟节点代表了React元素，但是在我看来并不是100%的正确，我任务虚拟DOM只是代表了这三个类：
`ReactCompositeComponent`,`ReactDOMComponent`,
`ReactDOMTextComponent`
稍后你就会看到我为什么这样说。

好了，让我们完成我们等实例化过程。我们对我们究竟实例化了什么东西很感兴趣。我们会创建一个`ReactCompositeComponent`的实例，但事实上，我们并不是真正创建，而是把`<ExampleApplication/>`放入到`ReactDOM.render`方法中。React从`TopLevelWrapper`开始渲染组件树。基本上，它就是一个包装器，它的`render`(组件的render方法)会在
稍后返回`<ExampleApplication/>`，代码如下
```javascript
//src\renderers\dom\client\ReactMount.js#277
TopLevelWrapper.prototype.render = function () {
  return this.props.child;
};

```
所以，到目前为止，只有`TopLevelWrapper`被创建了。我们继续，哈，
这里的代码值得研究下：
> 有趣的事：验证DOM内嵌
> 几乎每次内嵌组件在渲染时，为了验证是否符合html验证，他们都会被一个dedicated模块验证，这个过程叫做`内嵌DOM验证`. DOM内嵌验证意味着`子 -> 父`标签继承关系的确认。举个列子，比如说父亲标签是`<select>`，子标签就必须只能是以下中的一个`option`,`optgroup`,`#text`.
这些规则详细定义在https://html.spec.whatwg.org/multipage/syntax.html#parsing-main-inselect.
你也许已经看到过这个模块的工作流程，它的错误警告会展示如下:
<em> &lt;div&gt; cannot appear as a descendant of &lt;p&gt; </em>.


###好了，我们完成了 *Part 0*.
让我们总结下我们已经了解到的内容。在看一下流程图，然后移除一些不是那么重要的内容，之后大概是这样的：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-A.svg)

<em>0.4 部分 0 简化图 (点击查看大图)</em>

接着，我们在美化一下，

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-B.svg)

<em>0.5 部分 0 简化和重构(点击查看大图)</em>
很好，这就是全部了。我们已经了解了部分0的有价值的内容，所有的这些都会在完整的挂载模式图里被使用到：

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/master/stack/images/0/part-0-C.svg)

<em>0.6 部分 0 总结(点击查看大图)</em>

完成!


[下一页: 部分 1 >>](./Part-1.md)

[<< 上一页: 介绍](./Intro.md)


[首页](../../README.md)
