---
 title: "Redux5源码解析: compose"
 date: 2022-01-18
 tags: [前端]
 categories: [前端笔记]
---

「这是我参与2022首次更文挑战的第1天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

Redux可以说是一个典型的小而精的lib，源码量及api数量都不多，但设计却十分巧妙。本系列将深入Redux5源码，探究其实现与设计。

compose
-------

这是Redux中我最喜欢的一个api，虽然当初刚上手Redux的时候始终无法get其中的含义，但一看源码便瞬间明白了。

### 用法

```js
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import DevTools from './containers/DevTools'
import reducer from '../reducers'

const store = createStore(
    reducer,
    compose(applyMiddleware(thunk), DevTools.instrument())
)
```

在Redux中，这个函数常常用来增强store，比如通过[`applyMiddleware`](https://redux.js.org%2Fapi%2Fapplymiddleware "https://redux.js.org/api/applymiddleware")增加中间件。

但是compose的本意是组装，和store增强并没有关系，这是理解这个api的一个误区。真正理解这个api可以直接看源码。

### 源码实现

```javascript
export default function compose(...funcs: Function[]) {
  if (funcs.length === 0) {
    // infer the argument type so it is usable in inference down the line
    return <T>(arg: T) => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  )
}

```

compose的源码很简单：

如果传入多个函数参数，`funcs`数组会通过`reduce`组装为一个函数，并返回。而这个函数执行的效果便是将这些函数从右往左依次执行，并将返回值作为下一个函数的参数。

### 举个例子

假设我们有一下三个函数

```lua
function double(arg) {
  return arg*2
}
function triple(arg) {
  return arg*3
}
function fourfold(arg) {
  return arg*4
}
```

现在需要将他们依次执行，并获得返回值，我们可以这么写

```ini
var a2 = double(1);
var a3 = triple(a2);
var a4 = fourfold(a3)
```

或者

```less
var a4 = fourfold(triple(double(1)))
```

这两种写法，前一种易读但冗余；后一种简洁但增加了耦合性：如果其中一个函数需要改成其他的实现，改起来就会比较麻烦。

而如果用compose：

```scss
compose(fourfold,triple,double)(1)
```

不仅简洁易读，而且函数的组装变得更灵活。