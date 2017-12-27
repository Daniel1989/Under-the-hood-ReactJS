## 介绍

### 流程图初看


[![](../images/intro/all-page-stack-reconciler-25-scale.jpg)](../images/intro/all-page-stack-reconciler.svg)

<em>介绍.0 整体流程 (点击查看大图)</em>

So.. have a look. Take your time. Of course, overall it looks complex, but in fact, it describes only two processes: mount and update. I skip unmount because it’s kind of ‘reversed mount’, so, I’ve just decided to simplify scheme. Also, in fact, **this is not 100%** match of code, but just major pieces which describe the architecture, so, it’s rather 60% of the code, but other 40% in fact almost don’t bring any value, so, I omitted them to make it simple.
你可以先花点时间大致看一下整体流程。整体上它看起来有点复杂，但事实上，它只描述两个流程：挂载和更新。由于卸载就是某种程度“挂载的逆向”，同时为了简化流程图，我没有把卸载的过程加入到流程图里。同时，这个流程图不是 **100%** 和代码匹配的，只有那些描绘了整体架构的主要代码，这些代码占全部代码的60％左右，但是，剩下的40%的代码几乎没有任何实际价值，所以，为了流程图更清晰，这些代码也被我忽略了。

也许你已经注意到了，流程图上有很多颜色，每个逻辑单元（流程图上的图形）用其父亲模块的颜色高亮了，举个列子，如果方法A的在模块B被调用，且模块B是红色，则方法A也会被标记为红色。
以下是流程图的图例，主要描述了模块的颜色和文件路径。

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-src-path.svg)

<em>介绍.1 模块颜色(点击查看大图)</em>

让我们把它们放在一个图里，看看 **模块之间的依赖关系**

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/files-scheme.svg)

<em>介绍.2 模块依赖 (点击查看大图)</em>

正如你所知道的那样，React是为了**支持多平台**而创造出来的,如手机（**ReactNative**），游览器（**ReactDOM**），**服务端渲染**，**ReactART** （使用React创建矢量图）等等。所以，真实的流程图会比当前大很多，我们可以比较下，查看下多平台支持是如何影响到流程图的。

[![](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)](https://rawgit.com/Bogdan-Lyashenko/Under-the-hood-ReactJS/7c2372e1/stack/images/intro/modules-per-platform-scheme.svg)

<em>介绍.3 平台依赖(点击查看大图)</em>

So, you can see which parts were changed, it means they have separate implementation per platform. Let’s take something simple, like ReactEventListener, obviously, its implementation will be different for different platforms! Technically, as you can imagine, these platform dependent modules should be somehow injected or connected to the current logic flow, and, in fact, there are many such injectors as well. We omit them to simplify the scheme, there is nothing special in terms of coding, just standard composition pattern.
你可以看下哪些部分被改变了，那些被改变的部分在每一个平台会有独立的实现。举个列子，ReactEventListener，它的实现在各个平台是不同的！技术上说来，这些不同平台所依赖的模块应该以某种方式或注入或链接到当前的逻辑流程，事实上，这里有很多类似的注入器。为了简化流程图，我们都忽略了这些东西，代码上这些都没有什么特别的东西，都是标准的组合模式

Let’s learn the logic flow for **React DOM in a regular browser**. It’s the most used one, and it completely covers all React’s architecture ideas, so, fair enough!
让我们开始学习**React DOM在一般游览器里的**的逻辑流。这是最经常使用的一个，而且它也完全覆盖了所有React的架构理念。

### 实例代码

What is the best way to learn the code of a framework or library? That's right, read and debug the code. Alright, we are gonna to debug **two processes**: **ReactDOM.render** and **component.setState**, which map on mount and update. Let’s check the code we can write for a start. What do we need? Probably several small components with simple renders, so it will be easier to debug.
什么是学习一个框架或者库的源码的最好方式？没错，就是一边读一边调试。我们将调试**两个流程**: **ReactDOM.reander** 和 **component.setState** ，分别对应挂载和更新。让我们确认下为了开始工作，我们需要什么样的代码？大概就是一些带简单渲染方法的小组件，这些都会让我们更方便调试

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

我们已经准备好开始了，让我们先学习流程图的第一部分。一个接一个，我们将全部分析完。

[下一页: Part 0 >>](./Part-0.md)


[主页](../../README.md)