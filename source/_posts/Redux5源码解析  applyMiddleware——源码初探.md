---
 title: "Redux5源码解析: applyMiddleware——源码初探"
 date: 2022-01-19
 tags: [前端,Redux]
 categories: [前端笔记]
---

「这是我参与2022首次更文挑战的第2天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

Redux可以说是一个典型的小而精的lib，源码量及api数量都不多，但设计却十分巧妙。本系列将深入Redux5源码，探究其实现与设计。

applyMiddleware
---------------

Redux中的核心api`createStore`可以支持三个参数:`createStore(reducer, [preloadedState], [enhancer])`，第三个参数是store增强器，而Redux自带的唯一个`enhancer`便是`applyMiddleware`,因此在解析`createStore`之前，先来看一看`applyMiddleware`

### 源码实现

```js
function applyMiddleware(
  ...middlewares
) {
  return (createStore) =>(
      reducer,
      preloadedState
    ) => {
      const store = createStore(reducer, preloadedState)
      let dispatch = () => {
        throw new Error(...)
      }

      const middlewareAPI = {
        getState: store.getState,
        dispatch: (action, ...args) => dispatch(action, ...args)
      }
      const chain = middlewares.map(middleware => middleware(middlewareAPI))
      dispatch = compose(...chain)(store.dispatch)

      return {
        ...store,
        dispatch
      }
    }
}

```

从源码可以看出，`applyMiddleware`的参数，即每一个中间件都是一个函数，并且这些函数都被`compose`组合了起来。这些中间件函数的类型定于如下

```typescript
export interface Middleware<
  _DispatchExt = {}, 
  S = any,
  D extends Dispatch = Dispatch
> {
  (api: MiddlewareAPI<D, S>): (
    next: D
  ) => (action: D extends Dispatch<infer A> ? A : never) => any
}
```

这些中间件函数，接收的参数是`getState`(虽然也有dispatch，但其实不可用),返回的也是一个函数：`(next)=>(action)=>any`，先称它为fn吧。`chain`数组中的每个成员都是这样的一个fn。

fn函数能通过闭包获取到`getState`，因此可以记录state的快照。而`next`参数其实是一个`dispatch`(初始是`store.dispatch`，之后是前一个fn返回的(action)=>any。这样就巧妙的实现了一个**洋葱圈**)这些fn函数又被`compose`组合为一个新的`dispatch`,从`applyMiddleware`中返回出来，成为store增加的dispatch。

因此当store调用`dispatch`的时候，fn可以通过`action`拿到`dispatch`的参数，通过`getState`获取前后的state，实现自己的中间件逻辑。

#### 举个例子

以官网的log插件为例，可以更好地理解源码的运行过程。

```js
import { createStore, applyMiddleware } from 'redux'
import todos from './reducers'
function logger({ getState }) {
    return next => action => {
        console.log('will dispatch', action)

        const returnValue = next(action)
        console.log('state after dispatch', getState())

        return returnValue
    }
}

const store = createStore(todos, ['Use Redux'], applyMiddleware(logger))

store.dispatch({
    type: 'ADD_TODO',
    text: 'Understand the middleware'
})
// will dispatch: { type: 'ADD_TODO', text: 'Understand the middleware' }
// state after dispatch: [ 'Use Redux', 'Understand the middleware' ]
```

### 未完待续

本文浅尝applyMiddleware的源码，梳理其中几个函数的关系。但applyMiddleware的独特魅力在于以极少的代码便实现了一个洋葱圈模型。

下一篇将解剖**洋葱圈**，深入探究其实现原理。