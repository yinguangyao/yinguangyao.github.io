---
title: react查漏补缺
date: 2018-11-25 11:52:01
categories:
    - react
tags:
    - react
    - 前端
---

最近在看react源码，发觉以前对react的理解实在浮浅，这里记录了一些以前疏忽的点。
## key
react中的diff会根据子组件的key来对比前后两次virtual dom（即使前后两次子组件顺序打乱，也能根据key来匹配到），所以这里的key最好使用不会变化的值，比如id之类的，最好别用index，如果有两个子组件互换了位置，那么index就会导致diff全部失效。
## cloneElement
原来对cloneElement的理解就是类似cloneElement(App, {})这种写法，现在看了实现之后才理解。原来第一个参数应该是一个reactElement，而不是一个reactComponent，应该是<App />，而不是App，这个也确实是我没有好好看文档。
## setState
以前只知道setState是异步的，但是不知道其他情况下的表现，比如在setTimout或者Promise中会变成同步的，也不知道如果给setState传入一个函数，那个函数参数的prevState是可以拿到刚刚更新后的值，而非更新前的，这是一些区别。具体原因未知，还在看源码中。
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
## createElement和Component
createElement是babel解析react语法时使用的，Component则是一个组件类，两者区别在于babel会将一个类似<App name="666" />的结构解析为：
```
// 这里的App代表component type，也可以理解为就是class App
createElement(App, {
    name: "6666"
})
```
## ref
ref用到原生的标签上，可以直接在组件内部用this.refs.xxx的方法获取到真实DOM。
ref用到组件上，需要用ReactDOM.findDOMNode(this.refs.xxx)的方式来获取到这个组件对应的DOM节点。




