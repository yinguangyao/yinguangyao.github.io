---
title: redux状态管理
date: 2018-10-05 17:47:30
categories:
    - 前端
    - react
tags:
    - redux
    - react
    - 前端
---
react作为现今最火热的界面库，很多项目都采用了react构建界面，但是react自身也有许多不足，比如状态管理和组件通信等等，随着项目越来越大，只靠react很难满足需求，所以很多状态管理库由此诞生。

## 全局state
如果是比较小的个人项目，完全可以不使用任何状态库，可以在容器（顶层）组件里面使用一个类似redux store的大state，将这个state通过props和context传给子组件，可以将setState方法从容器组件传下去，或者将子组件的修改state方法放到容器组件中，这样就可以轻松实现状态管理。

## redux
redux是我们公司老项目中最常用的状态库，redux遵守dispatch -> action -> reducer -> store -> view的一套操作流程，好处是状态可预测，数据流动更加清晰，组件之间通信也更加方便，缺点就是需要在多个文件里面写很多action和reducer，在小项目中就会得不偿失，但是在大型项目中有利于以后维护。

## relite
relite是我们部门的工业聚大神写的一个类redux库，总体思想和redux一致，区别只是在于写法上的不同，relite中剔除了reducer的概念，将action和reducer合一，以原来的action.type来命名reducer函数，每个函数最终都会返回整个store，这样无需写一串长长的switch case语句，但也带来了一系列问题，比如由于返回整个store，这样很难和细粒度的PureComponent一起使用，底层性能优化也比不上react-redux。

## mobx
mobx是我们打算在下期项目中使用的状态管理库，目前还没有使用经验，不过看过了文档，已经有了一些了解。
mobx和flux走了完全不同的一条路，mobx更偏向vue等响应式库，通过getter、setter来实现对数据的监听，而mobx-react会提供observer方法来给组件订阅数据的变化，这样带来的好处就是可以做到更细粒度的控制组件渲染，这样会带来更高效的性能，可以做到只渲染子组件而不渲染父组件，这在react和redux里面是无法做到的。

如果是小型项目，也许你不需要使用状态管理库，如果操作不好，那么可以考虑一下relite，如果是中型项目，可以考虑使用mobx，如果是大型项目，那么建议使用redux。

