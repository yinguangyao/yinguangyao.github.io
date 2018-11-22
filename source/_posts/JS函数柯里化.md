---
title: JS函数柯里化
date: 2018-05-09 18:13:05
tags:
    - 前端
    - 函数式编程
    - javascript
categories:
    - 函数式编程
---
## js函数柯里化

&nbsp; &nbsp; &nbsp; &nbsp;最近一阵子利用上班的空暇时间看了看函数式编程相关的东西，有什么错误的地方希望能够指出来。
### 概念
&nbsp; &nbsp; &nbsp; &nbsp;用我自己的话来总结一下，函数柯里化的意思就是你可以一次传很多参数给curry函数，也可以分多次传递，curry函数每次都会返回一个函数去处理剩下的参数，一直到返回最后的结果。
### 实例
&nbsp; &nbsp; &nbsp; &nbsp; 这里还是举几个例子来说明一下：
### 柯里化求和函数
```
    // 普通方式
    var add1 = function(a, b, c){
        return a + b + c;
    }
    // 柯里化
    var add2 = function(a) {
        return function(b) {
            return function(c) {
                return a + b + c;
            }
        }
    }
```
&nbsp; &nbsp; &nbsp; &nbsp; 这里每次传入参数都会返回一个新的函数，这样一直执行到最后一次返回a+b+c的值。
&nbsp; &nbsp; &nbsp; &nbsp; 但是这种实现还是有问题的，这里只有三个参数，如果哪天产品经理告诉我们需要改成100次？我们就重新写100次？这很明显不符合开闭原则，所以我们需要对函数进行一次修改。
<!-- more -->
```
var add = function() {
    var _args = [];
    return function() {
        if(arguments.length === 0) {
            return _args.reduce(function(a, b) {
                return a + b;
            })
        }
        [].push.apply(_args, arguments);
        return arguments.callee;
    }
}
var sum = add();
sum(100, 200)(300);
sum(400);
sum(); // 1000
```
&nbsp; &nbsp; &nbsp; &nbsp; 我们通过判断下一次是否传进来参数来决定函数是否运行，如果继续传进了参数，那我们继续把参数都保存起来，等运行的时候全部一次性运行，这样我们就初步完成了一个柯里化的函数。

### 通用柯里化函数
&nbsp; &nbsp; &nbsp; &nbsp; 这里只是一个求和的函数，如果换成求乘积呢？我们是不是又需要重新写一遍？仔细观察一下我们的add函数，如果我们将if里面的代码换成一个函数执行代码，是不是就可以变成一个通用函数了？
```
var curry = function(fn) {
    var _args = [];
    return function() {
        if(arguments.length === 0) {
            return fn.apply(fn, _args);
        }
        [].push.apply(_args, arguments);
        return arguments.callee;
    }
}
var multi = function() {
    return [].reduce.call(arguments, function(a, b) {
        return a + b;
    })
}
var add = curry(multi);
add(100, 200, 300)(400);
add(1000);
add(); // 2000

```
&nbsp; &nbsp; &nbsp; &nbsp; 在之前的方法上面，我们进行了扩展，这样我们就已经实现了一个比较通用的柯里化函数了。
&nbsp; &nbsp; &nbsp; &nbsp;  也许你想问，我不想每次都使用那个丑陋的括号结尾怎么办？
```
var curry = function(fn) {
	var len = fn.length,
		args = [];
	return function() {
		Array.prototype.push.apply(args, arguments)
		var argsLen = args.length;
		if(argsLen < len) {
			return arguments.callee;
		}
		return fn.apply(fn, args);
	}
}
var add = function(a, b, c) {
	return a + b + c;
}

var adder = curry(add)
adder(1)(2)(3)
```
&nbsp; &nbsp; &nbsp; &nbsp; 这里根据函数fn的参数数量进行判断，直到传入的数量等于fn函数需要的参数数量才会返回fn函数的最终运行结果，和上面那种方法原理其实是一样的，但是这两种方式都太依赖参数数量了。
&nbsp; &nbsp; &nbsp; &nbsp; 我在简书还看到别人的另一种递归实现方法，其实实现思路和我的差不多吧。
```
// 简单实现，参数只能从右到左传递
function createCurry(func, args) {

    var arity = func.length;
    var args = args || [];

    return function() {
        var _args = [].slice.call(arguments);
        [].push.apply(_args, args);

        // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
        if (_args.length < arity) {
            return createCurry.call(this, func, _args);
        }

        // 参数收集完毕，则执行func
        return func.apply(this, _args);
    }
}
```
&nbsp; &nbsp; &nbsp; &nbsp; 这里是对参数个数进行了计算，如果需要无限参数怎么办？比如下面这种场景。
```
add(1)(2)(3)(2);
add(1, 2, 3, 4, 5);
```
&nbsp; &nbsp; &nbsp; &nbsp;这里主要有一个知识点，那就是函数的隐式转换，涉及到toString和valueOf两个方法，如果直接对函数进行计算，那么会先把函数转换为字符串，之后再参与到计算中，利用这两个方法我们可以对函数进行修改。
```
var num = function() {
}
num.toString = num.valueOf = function() {
	return 10;
}
var anonymousNum = (function() { // 10
	return num;
}())
```
&nbsp; &nbsp; &nbsp; &nbsp; 经过修改，我们的函数最终版是这样的。
```
var curry = function(fn) {
	var func = function() {
		var _args = [].slice.call(arguments, 0);
		var func1 = function() {
			[].push.apply(_args, arguments)
			return func1;
		}
		func1.toString = func1.valueOf = function() {
			return fn.apply(fn, _args);
		}
		return func1;
	}
	return func;
}
var add = function() {
	return [].reduce.call(arguments, function(a, b) {
		return a + b;
	})
}

var adder = curry(add)
adder(1)(2)(3)
```
&nbsp; &nbsp; &nbsp; &nbsp;  那么我们说了那么多，柯里化究竟有什么用呢？
#### 预加载
&nbsp; &nbsp; &nbsp; &nbsp; 在很多场景下，我们需要的函数参数很可能有一部分一样，这个时候再重复写就比较浪费了，我们提前加载好一部分参数，再传入剩下的参数，这里主要是利用了闭包的特性，通过闭包可以保持着原有的作用域。
```
var match = curry(function(what, str) {
  return str.match(what);
});

match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null
```
&nbsp; &nbsp; &nbsp; &nbsp; 上面例子中，使用hasSpaces函数来保存正则表达式规则，这样可以有效的实现参数的复用。
#### 动态创建函数
&nbsp; &nbsp; &nbsp; &nbsp; 这个其实也是一种惰性函数的思想，我们可以提前执行判断条件，通过闭包将其保存在有效的作用域中，来看一种我们平时写代码常见的场景。
```
 var addEvent = function(el, type, fn, capture) {
     if (window.addEventListener) {
         el.addEventListener(type, function(e) {
             fn.call(el, e);
         }, capture);
     } else if (window.attachEvent) {
         el.attachEvent("on" + type, function(e) {
             fn.call(el, e);
         });
     } 
 };
```
&nbsp; &nbsp; &nbsp; &nbsp; 在这个例子中，我们每次调用addEvent的时候都会重新进行if语句进行判断，但是实际上浏览器的条件不可能会变化，你判断一次和判断N次结果都是一样的，所以这个可以将判断条件提前加载。
```
var addEventHandler = function(){
    if (window.addEventListener) {
        return function(el, sType, fn, capture) {
            el.addEventListener(sType, function(e) {
                fn.call(el, e);
            }, (capture));
        };
    } else if (window.attachEvent) {
        return function(el, sType, fn, capture) {
            el.attachEvent("on" + sType, function(e) {
                fn.call(el, e);
            });
        };
    }
}
var addEvent = addEventHandler();
addEvent(document.body, "click", function() {}, false);
addEvent(document.getElementById("test"), "click", function() {}, false);
```
&nbsp; &nbsp; &nbsp; &nbsp; 但是这样做还是有一种缺点，因为我们无法判断程序中是否使用了这个方法，但是依然不得不在文件顶部定义一下addEvent，这样其实浪费了资源，这里有一种更好的解决方法。
```
var addEvent = function(el, sType, fn, capture){
    if (window.addEventListener) {
        addEvent =  function(el, sType, fn, capture) {
            el.addEventListener(sType, function(e) {
                fn.call(el, e);
            }, (capture));
        };
    } else if (window.attachEvent) {
        addEvent = function(el, sType, fn, capture) {
            el.attachEvent("on" + sType, function(e) {
                fn.call(el, e);
            });
        };
    }
}
```
&nbsp; &nbsp; &nbsp; &nbsp; 在addEvent函数里面对其重新赋值，这样既解决了每次运行都要判断的问题，又解决了必须在作用域顶部执行一次造成浪费的问题。
#### React
&nbsp; &nbsp; &nbsp; &nbsp; 在回家的路上我一直在想函数柯里化是不是可以扩展到更多场景，我想把函数换成react组件试试？我想到了高阶组件和redux的connect，这两个确实是将柯里化思想用到react里面的体现。我们想一想，如果把上面例子里面的函数换成组件，参数换成高阶函数呢？
```
    var curry = function(fn) {
	var func = function() {
		var _args = [].slice.call(arguments, 0);
		var func1 = function() {
			[].push.apply(_args, arguments)
			return func1;
		}
		func1.toString = func1.valueOf = function() {
			return fn.apply(fn, _args);
		}
		return func1;
	}
	return func;
}

var hoc = function(WrappedComponent) {
	return function() {
		var len = arguments.length;
		var NewComponent = WrappedComponent;
		for (var i = 0; i < len; i++) {
			NewComponent = arguments[i](NewComponent)
		}
		return NewComponent;
	}
}
var MyComponent = hoc(PageList);
curry(MyComponent)(addStyle)(addLoading)
```
这个例子是对原来的PageList组件进行了扩展，给PageList加了样式和loading的功能，如果想加其他功能，可以继续在上面扩展（注意addStyle和addLoading都是高阶组件），但是写法真的很糟糕，一点都不coooooool，我们可以使用compose方法，underscore和loadsh这些库中已经提供了。
```
var enhance = compose(addLoading, addStyle);
enhance(MyComponent)
```
### 总结
&nbsp; &nbsp; &nbsp; &nbsp; 其实关于柯里化的运用核心还是对函数闭包的灵活运用，深刻理解闭包和作用域后就可以写出很多灵活巧妙的方法。