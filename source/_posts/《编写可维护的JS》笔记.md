---
title: 《编写可维护的JS》笔记
date: 2018-07-07 23:28:19
tags:
    - 读书笔记
    - 前端
categories:
    - 电影
---

这里只记录了对自己有用的部分，更详细的内容请看《编写可维护的Javascipt》原书。
## 编程风格
### 格式化

 1. 一行代码长度一般不超过80个字符
 2. JS语句在换行的时候一般需要两个缩进
 3. 在语义不相关的JS语句之间空行
 4. 避免没有意义的函数/变量命名，如foo、tmp等等
 5. 使用大写来定义常量

### 注释
 1. 多行注释使用/* */
 2. hack注释
<!-- more -->
### 语句和表达式

 1. 语句最好不要不带花括号
 2. 严格模式下使用with语句会报错
 3. 尽量避免在循环中使用continue
 4. for in循环的时候最好使用hasOwnProperty判断，除非你想查找原型链
 5. 严格模式（use strict;）最好不要用在全局中，如果想在多个函数使用，可以放到立即执行函数中
 6. 如果布尔值和数字比较，会把布尔值转换为数字，如果一个值是对象，那会调用对象的valueOf方法，如果没有valueOf，那就调用toString方法，得到原始类型值后再比较，通常不建议使用==和!=，推荐===和!==
 7. 避免使用Function和eval，尽管eval可以用在某些JSON操作中，Function则可以用在lodash/underscore的template中
 8. 避免使用原始包装类型，如new String

## 编程实践
### UI层的松耦合

 1. 避免在JS中写HTML，可以考虑使用客户端/服务端模板来代替

### 避免使用全局变量

 1. 尽量不要去创建全局变量，函数尽量不要依赖全局变量，这样会让你的测试更容易测试
 2. 尽量使用单变量（对象），将需要定义的全局变量都放到它的属性下面
```javascript
// 创建命名空间
var yourGlobal = {
    namespace: function(ns) {
        var paths = ns && ns.split(".") || [],
            self = this
        paths.reduce(function(result, currentAttr) {
            self[result] = self[result] || {}
            return self[result][currentAttr] = self[result][currentAttr] || {}
        })  
    }
}
// yourGlobal.a.b就是一个命名空间
yourGlobal.namespace("a.b")
yourGlobal.a.b.c = 0
console.log(yourGlobal.a.b)
```

### 事件处理

 1. 事件触发的时候隔离应用逻辑，将应用逻辑从事件处理程序中抽离出来，便于测试
 2. 不要分发事件对象event，应用逻辑不应该依赖于event对象，如果需要event中的某属性，那么直接传入这个属性，不应该传入event对象，这样也便于测试

### 避免空比较

 1. 避免空比较，多使用typeof来判断类型，除非可以确定期望值的类型
 2. 检查引用值类型的时候使用instanceof方法

### 将配置数据从代码中抽离
### 抛出自定义错误

 1. 针对会引发错误的场景，将错误抛出（throw），有利于调试

### 不是你的对象不要动

 1. 不是你创建的对象，那么就不要修改，比如原生对象、DOM对象、BOM对象和类库的对象
 2. 对待已存在的对象应该不覆盖方法、不新增方法、不删除方法
 3. 如果不得不修改，可以继承或复制这个对象得到一个自己的对象，再对自己的这个对象进行操作
 4. 使用[门面模式][1]，门面模式的用意是为子系统提供一个集中化和简化的沟通管道，而不能向子系统加入新的行为
 5. 禁止扩展对象Object.preventExtension()，禁止为对象增加属性，但是可以删除和修改已有的属性，Object.isExtensible函数可以检查是否不可扩展
 6. 密封对象Object.seal()，禁止删除和增加属性，但是可以修改已有的属性，Object.isSealed函数可以检查是否密封
 7. 冻结对象Object.freeze()，禁止删除、增加和修改对象

### 浏览器嗅探

 1. 不能根据一个特性去推断另一个特性，比如支持getElementsByTagName就一定支持getElementById
 2. 避免使用浏览器推断和特性推断，尽量使用特性检测

### 自动化
### 文件和目录结构

 1. 保持一个文件只有一个对象（组件？类？）
 2. 相关的文件用目录分组
 3. 保持第三方代码独立（npm了解一下？）
 4. 目录结构（build、src、test）
### 校验
这几章使用的工具比较过时了，建议直接学习webpack
### 压缩
 5. 现代浏览器都支持http压缩，并且会发送一个http头作为请求的一部分来指名压缩类型的，比如：Accept-Encoding: gzip, deflate，服务器在http请求头中发现这些信息，会明白浏览器是支持通过gzip或者deflate对压缩后的文件执行解压缩的，会返回Content-Encoding: gzip来指名压缩的类型
 6. apache2和nginx内置了gzip作为http压缩
### 文档化
### 自动化测试
书上的内容也比较老的，建议使用jest和mocha
### 组装到一起
 1. 编制打包计划
 2. 开发版本的构建
 3. 集成版本的构建
 4. 发布版本的构建
 5. 使用CI系统

  [1]: http://www.cnblogs.com/skywang/articles/1375447.html