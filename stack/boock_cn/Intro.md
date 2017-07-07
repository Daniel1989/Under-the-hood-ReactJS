## 简介

### Scheme初探


[![](../images/intro/all-page-stack-reconciler-25-scale.jpg)](../images/intro/all-page-stack-reconciler.svg)

<em>简介 －0 模式图(点击看大图)</em>

先花点时间看一下模式图。当然了，大体上流程图看起来有点复杂，但本质上，图片内容只描述了两个过程：挂载和更新。由于'卸载'在某种程度上就是挂载的反过程，我就跳过了卸载的内容，同时，我也简化了整个模式图。同时，模式图和代码没有百分百的匹配，这是因为一些重要代码就能描述了整个项目架构，这些代码大概占60%，也就是另外40%的代码在事实上不会有任务价值，所以，我也决定忽略这些代码以使得模式图看起来更简单。

也许你已经注意到模式图上面由很多颜色标注，每个逻辑单元（用形状标记的）被其在父模块颜色高亮，举个列子，如果方法A在模块B中
标记为红色，且被调用，则方法A也会被标记为红色。以下是模块图列，每个图列描述了模块颜色和文件路径。

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)

<em>简介 － 1模块颜色(点击看大图)</em>

让我们结合模式图来分析下 **模块依赖关系**.

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)

<em>简介 － 2模块依赖(点击看大图)</em>
正如你所知道的那样，React是**支持多环境**的。如手机（**ReactNative**）,游览器(**ReactDOM**)，还有**服务端渲染**和**ReactART**(使用react绘制矢量图)等等。所以，全部的代码其实是比模式图里展示的还要多。我们可以比较下，多环境支持是如何影响到模式图的。

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)

<em>简介 － 3平台依赖（点击看大图）</em>

上图中，你可以看到代码中哪些部分改变了，也就是说，改变的部分在不同平台有不同的实现。一个简单的列子，ReactEventListener，很显然，在不同的平台它的实现会不一样！技术上来说，正如你想的那样，这些平台依赖模块应该会以某种方式注入或连接到当前的逻辑过程流中，事实上，react里有很多这样的注入。我们决定都忽略他们以使得简化模式图。在代码中并没有特殊技巧去处理这些，只有标准的设计模式－－组合模式。

让我们先学习下**游览器下React DOM**的逻辑流。这个是被使用最多的一个，它也完全覆盖了所有React的架构思想。总之，用这个足够了。


### 代码示列

什么是学习框架和库源码的最好途径呢？没错，就是阅读并调试代码。接下去，我们打算调试**两个过程**：**ReactDOM.render** 和**component.setState**，它们分别对应挂载和更新这两个过程。让我们想想我们可以从什么代码可以开始下手。我们需要什么呢？也许就是一些带有简单渲染方法的简单组件，这样会让我们更方便调试。

```javascript
class ChildCmp extends React.Component {
    render() {
        return <div> {this.props.childMessage} </div>
    }
}

class ExampleApplication extends React.Component {
    constructor(props) {
        super(props);
        this.state = {message: 'no message'};
    }

    componentWillMount() {
        //...
    }

    componentDidMount() {
        /* setTimeout(()=> {
            this.setState({ message: 'timeout state message' });
        }, 1000); */
    }

    shouldComponentUpdate(nextProps, nextState, nextContext) {
        return true;
    }

    componentDidUpdate(prevProps, prevState, prevContext) {
        //...
    }

    componentWillReceiveProps(nextProps) {
        //...
    }

    componentWillUnmount() {
        //...
    }

    onClickHandler() {
        /* this.setState({ message: 'click state message' }); */
    }

    render() {
        return <div>
            <button onClick={this.onClickHandler.bind(this)}> set state button </button>
            <ChildCmp childMessage={this.state.message} />
            And some text as well!
        </div>
    }
}

ReactDOM.render(
    <ExampleApplication hello={‘world’} />,
    document.getElementById('container'),
    function() {}
);
```
现在，我们已经做好准备工作，让我们聚焦到模式图的第一部分，我们会一部分一部分的过完整个模式图。

[去下一节: Part 0 >>](./Part-0.md)


[Home](../../README.md)