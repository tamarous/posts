# Component的生命周期
学习React Native已经快一个星期了，在这个过程中，我体会到了React Native为何会如此流行，因为用它来构建应用程序比原生快捷方便多了：内置的fetch方法使得网络请求开箱即用，对JSON数据的良好解析能力使得传统的创建model、解析数据、从数据生成model的过程变为过去时……

与原生iOS应用程序相同，React中的Component也有生命周期这一概念，相信熟悉iOS开发的同学对于它都不陌生了……React同样提供了一些方法，允许我们在Component进入某种状态之前或之后进行额外的设置或操作。

按照各阶段所发生的事件不同，Component的生命周期可以分为三个阶段：初始化、状态更新、销毁。

## 初始化过程
` getDefaultsProps`：可以在此方法中定义一些属性值。

` getInitialState`：可以在此方法中定义一些状态值

    // ES5
    getInitialState:function() {
        return {
            ……
        }
    }
在ES6中，React现在推荐使用构造器方法来进行初始化
    
    // ES6
    constructor(props) {
        super(props);
        this.state = {
           ……
        }
    }
    
`componentWillMount`：在render方法执行之前被调用

`render`：渲染该组件中某些或者全部需要更新的子组件

`componentDidMount`：在render方法执行之后被调用

注意在`componentWillMount`中设置`this.state`是不会触发重新渲染的，而在其他函数中`this.state`的变化会触发一次渲染。
## 更新
### 状态更新
`shouldComponentUpdate`：该函数会在每一次渲染之前调用，在此时可以获得该状态时的props和state值，进而判断接下来的这一次渲染是否是必要的。如果是则返回true否则返回false

    shouldComponentUpdate(nextProps, nextState) {
    
    }

`componentWillUpdate`：当上面这个方法返回true时，该方法就会被调用，在此方法中可以为渲染做些准备。该方法中不允许对props或者state进行任何修改

`render`和`componentDidUpdate`函数和前一节类似。

### 属性更新
`componentWillRecieveProps`：当属性改变的时候这个方法就会被执行。在此方法中我们可以根据现有的和要变更的属性来更新组件的状态。

    componentWillRecieveProps(nextProps) {
    
    }

`componentWillRecieveProps`, `componentWillUpdate`, `render`, `componentDidUpdate`这些方法和前一节类似。

## 销毁
组件的销毁过程只有一个方法：`componentWillUnmount`，该方法会在组件被销毁之前执行，因此可以在此方法中进行一些清理工作。





