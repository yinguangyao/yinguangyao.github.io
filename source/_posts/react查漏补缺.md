---
title: react查漏补缺
date: 2018-11-25 11:52:01
categories: [前端, react]
tags:
    - react
    - 前端
---

最近在看react-lite源码，发觉以前对react的理解实在浮浅，这里基于对react-lite的理解记录了一些以前疏忽的点。
## createElement和component
在react里面，经过babel的解析后，jsx会变成createElement执行后的结果。
```
const Test = (props) => <h1>hello, {props.name}</h1>;
<Test name="world" />
```
`<Test name="world" />`经过babel解析后会变为createElement(Test, {name: "world})，这里的Test就是上面的Test方法，name就是Test方法里面接受的props中的name。
实际上当我们从开始加载到渲染的时候做了下面几步：
```
// 1. babel解析jsx
<Test name="world"> -> createElement(Test, {name: "world"})
// 2. 对函数组件和class组件进行处理
// 如果是类组件，不做处理，如果是函数组件，增加render方法
const props = {name: world};
const newTest = new Component(props);
newTest.render = function() {
    return Test(props);
}
// 3. 执行render方法
newTest.render();
```
## key
react中的diff会根据子组件的key来对比前后两次virtual dom（即使前后两次子组件顺序打乱，也能根据key来匹配到），所以这里的key最好使用不会变化的值，比如id之类的，最好别用index，如果有两个子组件互换了位置，那么index就会导致diff全部失效。
<!-- more -->
## cloneElement
原来对cloneElement的理解就是类似cloneElement(App, {})这种写法，现在看了实现之后才理解。原来第一个参数应该是一个reactElement，而不是一个reactComponent，应该是`<App />`，而不是App，这个也确实是我没有好好看文档。
## setState
在react中为了防止多次setState导致多次渲染带来不必要的性能开销，所以会将待更新的state放到队列中，等到合适的时机（一般是组件第一次渲染或触发事件后）后进行batchUpdate，所以在setState后无法立即拿到更新后的state，很多人说setState是异步的，setState表现确实是异步，只是里面没有用异步代码实现。
如果是给setState传入一个函数，这个函数是执行前一个setState后才被调用的，所以函数返回的参数可以拿到更新后的state。
但是如果将setState在异步方法中（setTimeout、Promise等等）调用，由于方法是异步的，会导致组件pending结束后才执行异步方法中的setState，这个时候由于组件已经不处于pending状态了，会导致setState立即执行，这时通过this.state可以拿到最新的值。
```
class App extends React.Component {
    state = {
        count: 0;
    }
    test() {
        this.setState({
            count: this.state.count + 1
        }); 
        console.log(this.state.count); // 此时为0
        this.setState({
            count: this.state.count + 1
        });
        console.log(this.state.count); // 此时为0
    }
    test2() {
        setTimeout(() => {
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count); // 此时为1
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count); // 此时为2
        })
    }
    test3() {
        Promise.resolve().then(() => {
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count); // 此时为1
            this.setState({
                count: this.state.count + 1
            });
            console.log(this.state.count); // 此时为2
        })
    }
    test4() {
        this.setState(prevState => {
            console.log(prevState.count); // 0
        return {
            count: prevState.count + 1
        };
        });
        this.setState(prevState => {
            console.log(prevState.count); // 1
            return {
                count: prevState.count + 1
            };
        });
    }
    async test4() {
        await 0;
        this.setState({
            count: this.state.count + 1
        });
        console.log(this.state.count); // 此时为1
        this.setState({
            count: this.state.count + 1
        });
        console.log(this.state.count); // 此时为2
    }
}
```
## ref
ref用到原生的标签上，可以直接在组件内部用this.refs.xxx的方法获取到真实DOM。
ref用到组件上，需要用ReactDOM.findDOMNode(this.refs.xxx)的方式来获取到这个组件对应的DOM节点。




