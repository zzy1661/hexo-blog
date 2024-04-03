---
 title: "Redux5源码解析：store与reducer"
 date: 2022-01-21
 tags: [前端]
 categories: [前端笔记]
---

「这是我参与2022首次更文挑战的第4天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

Redux可以说是一个典型的小而精的lib，源码量及api数量都不多，但设计却十分巧妙。本系列将深入Redux5源码，探究其实现与设计。

store
-----

`createStore`用来创建store，可以接收三个参数： `createStore(reducer, [preloadedState], [enhancer])`

preloadedState是初始state，enhancer是中间件

源码如下：

```js
function createStore(
  reducer,
  preloadedState?,
  enhancer?
){
      if (typeof enhancer !== 'undefined') {

        return enhancer(createStore)(
          reducer,
          preloadedState
        )
      }
     ...
     const store = {
        dispatch: dispatch as Dispatch<A>,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable
    }
    return store

}
```

store就是一个由dispatch、subscribe、 getState和 replaceReducer组成的对象

### getState与currentState

Redux将state维护在currentState变量中，并提供getState获取state，禁止直接获取

```js
 let currentState = preloadedState 
 ...
 function getState() {
    if (isDispatching) {
      throw new Error(/*dispatching时禁止读取 */)
    }

    return currentState 
 }

```

### reducer、 dispatch与subscribe

dispatch的时候会将state与action交给reducer处理，并将currentState更新问reducer返回的state。之后调用通过subscribe注册的listener

```js
let currentReducer = reducer
function subscribe(listener) {
   
    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() { ...}
 }

function dispatch(action: A) {
    
    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

### 其他属性

*   `replaceReducer`:支持动态更换reducer
*   `[$$observable]: observable`可以将Redux与其他observable/reactive的lib搭配

reducer
-------

reducer是一个接收action和state，并返回一个新的state的函数。除了createStore，与之相关的另一个顶层api是`combineReducers`。虽然Redux不支持多store,但通过这个api，我们能够以namespace切分state。

用法如下：

```js
// reducers/todos.js
export default function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([action.text])
    default:
      return state
  }
}
// reducers/counter.js
export default function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state#### `reducers/index.js`
  }
}
// reducers/index.js
import { combineReducers } from 'redux'
import todos from './todos'
import counter from './counter'

export default combineReducers({
  todos,
  counter
})
// App.js
import { createStore } from 'redux'
import reducer from './reducers/index'

const store = createStore(reducer)
console.log(store.getState())
// {
// counter: 0,
// todos: []
// }
```

### 核心源码

```js
export default function combineReducers(reducers: ReducersMapObject) {
  ...
  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: AnyAction
  ) {
    let hasChanged = false
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    // 这里调用了所有的reducer
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
     
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || finalReducerKeys.length !== Object.keys(state).length
    return hasChanged ? nextState : state
  }
}

```

**从源码看出这里有个坑：**

当dispatch的时候，Redux并不能判断应该由哪个reducer执行，因此一股脑让所有的reducer都执行了一遍。通过判断返回的state与原本的state是否相同，确定是返回原先的state还是新的state。这样可以避免触发多余的listener。

**因此reducer中如果没有action匹配，那么务必要`return state`,返回原本的state,而不能返回一个对象字面量或新的state**。

bindActionCreators
------------------

这个api用来简化dispatch和action，可以理解为柯里化。

比如组件需要进行这样一个调用：

```js
dispatch({
    type: 'REMOVE_TODO',
    id
})
```

那么这个组件既要能够获取到dispatch，又要知道action的类型，但这些往往都不是子组件应该关心的。

这时候就可以使用bindActionCreators，将action和dispatch封装起来。

```js
const TodoActionCreators = {
     removeTodo(id) {
          return {
                type: 'REMOVE_TODO',
                id
          }
    }
}
 const { removeTodo } = useMemo(
    () => bindActionCreators(TodoActionCreators, dispatch),
    [dispatch]
  )
```

### 核心源码

```js
function bindActionCreator<A extends AnyAction = AnyAction>(
  actionCreator: ActionCreator<A>,
  dispatch: Dispatch
) {
  return function (this: any, ...args: any[]) {
      //核心源码就这一句
    return dispatch(actionCreator.apply(this, args))
  }
}

export default function bindActionCreators(
  actionCreators: ActionCreator<any> | ActionCreatorsMapObject,
  dispatch: Dispatch
) {
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  const boundActionCreators: ActionCreatorsMapObject = {}
  for (const key in actionCreators) {
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}

```

里面的核心就一行`dispatch(actionCreator.apply(this, args))`。

### react-redux

Redux本身只提供了store的创建和管理，后续系列将从react-redux源码看它与react是如何结合的。

未完待续~