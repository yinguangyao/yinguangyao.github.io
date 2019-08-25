---
title: 浅谈react diff算法实现
date: 2019-05-05 18:29:43
categories: [前端, react]
tags:
    - react
    - 前端
---
这是一篇硬核文，因此不会用生动幽默的语言来讲述，这篇文章大概更是自己心血来潮的总结吧哈哈哈哈。

有很多文章讲过react的diff算法，但要么是晦涩难懂的源码分析，让人很难读进去，要么就是流于表面的简单讲解，实际上大家看完后还是一头雾水，因此我将react-lite中的diff算法实现稍微整理了一下，希望能够帮助大家解惑。

对于react diff，我们已知的有两点，一个是会通过key来做比较，另一个是react默认是同级节点做diff，不会考虑到跨层级节点的diff（事实是前端开发中很少有DOM节点跨层级移动的）。

![image_1da3op2k3207vppgli1ff91a749.png-56.8kB][1]
## 递归更新
首先，抛给我们一个问题，那就是react怎么对那么深层次的DOM做的diff？实际上react是对DOM进行递归来做的，遍历所有子节点，对子节点再做递归。
```javascript
// 超简单代码实现
const compareTwoVnodes(oldVnode, newVnode, dom) {
    let newNode = dom

    // 如果新的虚拟DOM是null，那么就将前一次的真实DOM移除掉
    if (newVnode == null) {
        destroyVnode(oldVnode, dom)
        dom.parentNode.removeChild(dom)

    } else if (oldVnode.type !== newVnode.type || oldVnode.key !== newVnode.key) {
        // replace
        destroyVnode(oldVnode, dom)
        newNode = initVnode(newVnode, parentContext, dom.namespaceURI)
        dom.parentNode.replaceChild(newNode, dom)

    } else if (oldVnode !== newVnode || parentContext) {
        // same type and same key -> update
        newNode = updateVNode(oldVnode, newVnode, dom, parentContext)
    }
}
/** 
* 更新虚拟DOM
* 这里的type需要注意一下，如果vnode是个html元素，例如h1，那么type就是'h1'
** 如果vnode是一个函数组件，例如const Header = () => <h1>header</h1>，那么type就是函数Header
** 如果vnode是一个class组件，那么type就是那个class
*/
const updateVNode = (vnode, node) => {
    const { type } = vnode; // type是指虚拟DOM的类型
    // 如果是class组件
    if (type === VCOMPONENT) {
        return updateComponent(vnode, node)
    } else (type === VSTATELESS){
        return updateStateLess(vnode, node)
    }
    updateVChildren(vnode, node)
}
// 更新class组件（调用render方法拿到新的虚拟DOM）
const updateComponent = (vnode, node) => {
    const { type: Component } = vnode; // type是指虚拟DOM的类型
    const newVNode = new Component().render();
    compareTwoVnodes(newVNode, vnode, node);
}
// 更新无状态组件（直接执行函数拿到新的虚拟DOM）
const updateStateLess = (vnode, node) => {
    const { type: Component } = vnode; // type是指虚拟DOM的类型
    const newVNode = Component();
    compareTwoVnodes(newVNode, vnode, node);
}
const updateVChildren = (vnode, node) => {
    for (let i = 0; i < node.children.length; i++) {
        updateVNode(vnode.children[i], node.children[i])
    }
}
```
因此，我们这里以其中一层节点来讲解diff是如何做到列表的更新。
## 状态收集
假设我们的react组件渲染成功后，在浏览器中显示的真实DOM节点是A、B、C、D，我们更新后的虚拟DOM是B、A、E、D。

那我们这里需要做的操作就是，将原来DOM中已经存在的A、B、D进行更新，将原来DOM中存在，而现在不存的C移除掉，再创建新的D节点。

这样一来，问题就简化了很多，我们只需要收集到需要create、remove和update的节点信息就行了。

![diff][2]

```javascript
// oldDoms是真实DOM，newDoms是最新的虚拟DOM
const oldDoms = [A, B, C, D],
    newDoms = [B, A, E, D],
    updates = [],
    removes = [],
    creates = [];
// 进行两层遍历，获取到哪些节点需要更新，哪些节点需要移除。
for (let i = 0; i < oldDoms.length; i++) {
    const oldDom = oldDoms[i]
    let shouldRemove = true
    for (let j = 0; j < newDoms.length; j++) {
        const newDom = newDoms[j];
        if (
            oldDom.key === newDom.key &&
            oldDom.type === newDom.type
        ) {
            updates[j] = {
                index: j,
                node: oldDom,
                parentNode: parentNode // 这里真实DOM的父节点
            }
            shouldRemove = false
        }
    }
    if (shouldRemove) {
        removes.push({
            node: oldDom
        })
    }
}
// 从虚拟DOM节点来取出不要更新的节点，这就是需要新创建的节点。
for (let j = 0; j < newDoms.length; j++) {
    if (!updates[j]) {
        creates.push({
            index: j,
            vnode: newDoms[j],
            parentNode: parentNode // 这里真实DOM的父节点
        })
    }
}
```
这样，我们便拿到了想要的状态信息。
## diff
在得到需要create、update和remove的节点后，我们这时就可以开始进行渲染了。
![状态][3]

首先，我们遍历所有需要remove的节点，将其从真实DOM中remove掉。因此这里需要remove掉C节点，最后渲染结果是A、B、D。
```javascript
const remove = (removes) => {
    removes.forEach(remove => {
        const node = remove.node
        node.parentNode.removeChild(node)
    })
}
```
其次，我们再遍历需要更新的节点，将其插入到对应的位置中。所以这里最后渲染结果是B、A、D。
```javascript
const update = (updates) => {
    updates.forEach(update => {
        const index = update.index,
            parentNode = update.parentNode,
            node = update.node,
            curNode = parentNode.children[index];
        if (curNode !== node) {
            parentNode.insertBefore(node, curNode)
        }
    })
}
```
最后一步，我们需要创建新的DOM节点，并插入到正确的位置中，最后渲染结果为B、A、E、D。
```javascript
const create = (creates) => {
    creates.forEach(create => {
        const index = create.index,
            parentNode = create.parentNode,
            vnode = create.vnode,
            curNode = parentNode.children[index],
            node = createNode(vnode); // 创建DOM节点
        parentNode.insertBefore(node, curNode)
    })
}
```
虽然这篇文章写的比较简单，但是一个完整的diff流程就是这样了，可以加深对react的一些理解。


  [1]: http://static.zybuluo.com/gyyin/3ieg6ign07h9uupzczsdvwon/image_1da3op2k3207vppgli1ff91a749.png
  [2]: https://user-gold-cdn.xitu.io/2019/4/30/16a6bdd0c7daa509?imageView2/0/w/1280/h/960/format/webp/ignore-error/1
  [3]: https://user-gold-cdn.xitu.io/2019/4/30/16a6bdd0c90eb4f0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1