---
title: 实现一个bind函数
date: 2018-05-30 22:26:14
tags:
- polyfill
- 前端
- 编程
categories: [前端, polyfill]
---
目前的打算还是继续深入前端基础知识，所以打算从polyfill开始做起。
## bind函数 ##
bind函数最常见的用法是绑定函数的上下文，比如在setTimeout中的this一般都是指向window，如果我们想改变上下文，这里可以使用bind函数来实现。
```
var a = 10;
var test = function() {
    console.log(this.a);
}
// 如果直接执行test，最终打印的是10.
var bindTest = test.bind({a: "111"})
bindTest(); // 111
```
从上面这个例子可以看出来，bind函数改变了test函数中this的指向。
除此之外，bind函数还有两个特殊的用法，一个是柯里化，一个是绑定构造函数无效。
<!-- more -->
### 柯里化 ###
bind函数的柯里化其实是不完全的，其实只做了一次柯里化，看过MDN的polyfill实现后也就理解了。
```
var test = function(b) {
    return this.a + b;
}
// 如果直接执行test，最终打印的是10.
var bindTest1 = test.bind({a: 20});
bindTest1(10); // 30
// 这里的bind是个柯里化的函数
var bindTest2 = test.bind({a: 20}, 10);
bindTest2(); // 30;
```
### 构造函数无效 ###
其实准确的来说，bind并不是对构造函数无效，只是对new的时候无效，如果直接执行构造函数，那么还是有效的。
```
var a = 10;
var Test = function(a) {
    console.log(this.a);
}
var bindTest = Test.bind({a: 20});
bindTest(); // 20
// 在new的时候，Test中的this并没有指向bind中的对象
new bindTest(); // undefined
```
## 实现一个bind ##
我们可以先实现一个简易版本的bind，再不断完善。由于是在函数上调用bind，所以bind方法肯定存在于Function.prototype上面，其次bind函数要有改变上下文的作用，我们想一想，怎么才能改变上下文？没错，就是call和apply方法。

然后还要可以柯里化，还好这里只是简单的柯里化，我们只要在bind中返回一个新的函数，并且将前后两次的参数收集起来就可以做到了。
```
Function.prototype.bind = function() {
    var args = arguments;
    // 获取到新的上下文
    var context = args[0];
    // 保存当前的函数
    var func = this;
    // 获取其他的参数
    var thisArgs = Array.prototype.slice.call(args, 1);
    var returnFunc = function() {
        // 将两次获取到的参数合并
        Array.prototype.push.apply(thisArgs, arguments)
        // 使用apply改变上下文
        return func.apply(context, thisArgs);
    }
    return returnFunc;
}
```
这里实现了一个简单的bind函数，可以支持简单的柯里化，也可以改变上下文作用域，但是在new一个构造函数的时候还是会改变上下文。

这里我们需要考虑一下，怎么做才能让在new的时候无效，而其他时候有效？

所以我们需要在returnFunc里面的apply第一个参数进行判断，如果是用new调用构造函数的时候应该传入函数本身，否则才应该传入context，那么该怎么判断是new调用呢？

关于在new一个构造函数的时候，这中间做了什么，建议参考这个问题：[在js里面当new了一个对象时，这中间发生了什么？][1]

所以我们很容易得出，由于最终返回的是returnFunc，所以最终是new的这个函数，而在new的过程中，会执行一遍这个函数，所以这个过程中returnFunc里面的this指向new的时候创建的那个对象，而那个新对象指向returnFunc函数。

但是我们希望调用后的结果只是new的func函数，和我们正常new func一样，所以这里猜想，在returnFunc中，一定会将其this传入func函数中执行，这样才能满足这几个条件。
```
Function.prototype.bind = function() {
    var args = arguments || [];
    var context = args[0];
    var func = this;
    var thisArgs = Array.prototype.slice.call(args, 1);
  	var returnFunc = function() {
      Array.prototype.push.apply(thisArgs, arguments);
      // 最关键的一步，this是new returnFunc中创建的那个新对象，此时将其传给func函数，其实相当于做了new操作最后一步（执行构造函数）
      return func.apply(this instanceof returnFunc ? this : context, thisArgs);
    }
    return returnFunc
}
function foo(c) {
    this.b = 100;
    console.log(c);
    return this.a;
}

var func =  foo.bind({a:1});
var newFunc = new func() // undefined
```
但是这样还是不够的，如果foo函数原型上面还有更多的方法和属性，这里的newFunc是没法获取到的，因为foo.prototype不在newFunc的原型链上面。
所以这里我们需要做一些改动，由于传入apply的是returnFunc的一个实例（this），所以我们应该让returnFunc继承func函数，最终版是这样的。
```
Function.prototype.bind = function() {
    var args = arguments || [];
    var context = args[0];
    var func = this;
    var thisArgs = Array.prototype.slice.call(args, 1);
    var returnFunc = function() {
      Array.prototype.push.apply(thisArgs, arguments);
      // 最关键的一步，this是new returnFunc中创建的那个新对象，此时将其传给func函数，其实相当于做了new操作最后一步（执行构造函数）
      return func.apply(this instanceof func ? this : context, thisArgs);
    }
    returnFunc.prototype = new func()
    return returnFunc
}
```
这样我们就完成了一个bind函数，这与MDN上面的polyfill实现方式大同小异，这里可以参考一下MDN的实现：[Function.prototype.bind()][2]

参考链接：
+ [MDN：Function.prototype.bind()
][2]
+ [手写bind()函数，理解MDN上的标准Polyfill][3]


  [1]: https://segmentfault.com/q/1010000006670906
  [2]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
  [3]: https://blog.csdn.net/u010552788/article/details/50850453