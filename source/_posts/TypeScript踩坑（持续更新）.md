---
title: TypeScript踩坑（持续更新）
date: 2018-11-22 19:26:05
categories: [前端, TypeScript]
tags:
    - TypeScript
    - 前端
---

## 配置
### 为JS编写类型声明文件(d.ts)
很多第三方库已经有自己的类型声明文件，比如@types/react，@types/react-native，这些需要单独安装，而例如mobx-react和mobx这种会自带类型文件，不需要单独安装。

我们最近有个新项目，需要照顾到不同同学，有的愿意用TS，有的不想用TS，为了照顾到双方，所有的公共模块都是JS写的，所以需要单独为TS写类型声明文件，具体语法请参考TS官网的文档，这里只是讲一些坑。
#### 集中管理，相对路径导入
为项目中的JS写类型文件的时候，需要先引入对应的文件，然后以导入的路径为名字声明一个模块，最后在需要用到这个类型文件的地方用///来引入相对路径。
目录结构如下：
```
- @types
    - BasePage.d.ts
- src
    - frame
        - BasePage.js
    - page
        - hotelList
            - index.tsx
```
类型声明文件：
```
// BasePage.d.ts
import BasePage from '../src/frame/BasePage'
declare module "../src/frame/BasePage" {
    export default class BasePage{}
}
```
引入类型文件：
```
// index.tsx
/// <reference path="../../../@types/BasePage.d.ts" />
```
如果是想设置全局的类型文件，可以在tsconfig.json的paths字段里面指定对应的路径，这样就不需要单独用reference引入了。
#### 自动导入
上面那种方法虽然可以将types文件集中管理，但是有个很麻烦的地方就是需要在引入BasePage模块的地方手动引入d.ts文件，这个真的很繁琐，这里有个更好的方法。
```
- src
    - frame
        - BasePage.js
        - BasePage.d.ts
    - page
        - hotelList
            - index.tsx
```
index.tsx文件直接import导入BasePage就行了，不需要再专门引入BasePage.d.ts，这里两者命名一样，所以会自动识别BasePage.d.ts，但是BasePage.d.ts的语法也变化了一些。
```
// BasePage.d.ts
// 注意：这里不需要再声明declare module "BasePage"了，否则会识别不了
export default class BasePage{}
```
<!-- more -->
## 语法
### 1. Element implicitly has an 'any' type because type 'Test' has no index signature
```
class PageFlag {
    updatePageFlag(name: string, value: boolean) {
        this[name] = value;
    }
}
```
这里我希望能够更新PageFlag中的数据，但是又不想对所有的属性一一列举出来，但是由于没有设置this[name]的类型，导致了报错，这里有几种解决办法。

#### (1) 修改tsconfig.json里的noImplicitAny为false。

#### (2) 给this设置类型，但是不好的地方就是失去了类型判断的意义，如下：

```
// 例2
interface IParams {
    [propName: string]: any
}
class PageFlag {
    updatePageFlag = (name: string, value: boolean) => {
        (<IParams>this)[name] = value
    }
}
```

#### (3) 手动列举所有属性
虽然这样比较麻烦，但是一眼就能看出来PageFlag有哪些属性，数据比较清晰。
```
type pageFlag = "showLoading" | "showMask" | "showCalendar";
class PageFlag {
    showLoading: boolean = false;
    showMask: boolean = false;
    showCalendar: boolean = false;
    
    updatePageFlag = (name: pageFlag, value: boolean) => {
       this[name] = value
    }
}
```
## export from
有些文件夹下面有很多文件，所以我喜欢增加一个index.ts文件来直接export from其他文件，这样在其他地方引入的时候可以直接import from目录，类似如下：
```
// 我们有个components文件夹，下面有很多组件文件（都是export default导出的），我们可以components下创建index.ts文件，里面这么写（下）：
export Hotel from './Hotel'
export * as HotelList from './HotelList'
export Header from './Header'
```
但是这两种export from的方式在TS里面都会编译报错，可是我在ES6里面明明写的好好的啊！！！
后来在google上找到了一个链接，原来这两种export from的方式都只是提案，如果在ES6中则需要额外添加@babel/plugin-proposal-export-namespace-from 插件来支持，TS中不支持这些写法。
![image.png-872kB][1]
但是感觉这个更像野路子，也许未来会支持，遂放弃，最后发现了另外一种写法，可以完美解决这个问题。
```
export {default as Hotel} from './Hotel'
export {default as HotelList} from './HotelList'
export {default as Header} from './Header'
```
顺便说一下，export from其实还有下面两种写法，但是这两种写法都是需要模块export导出，而不是export default导出的。
```
export { Hotel } from './Hotel'
export * from './Hotel
```

  [1]: https://image-static.segmentfault.com/387/530/3875307783-5bf6970920056_articlex