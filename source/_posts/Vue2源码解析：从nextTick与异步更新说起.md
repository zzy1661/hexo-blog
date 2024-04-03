---
 title: "Vue2源码解析：从nextTick与异步更新说起"
 date: 2021-11-22
 tags: [前端]
 categories: [前端笔记]
---

这是我参与11月更文挑战的第22天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095 "https://juejin.cn/post/7023643374569816095")

Vue为什么要设计nextTick
-----------------

vue或react这种框架，最大的性能消耗来自于diff计算，而修改data数据，或者setState,这些触发更新的操作，都意味着一次diff的计算，如果没有任何的优化，频繁地进行数据的修改，很容易造成页面的卡顿。

因此，两个框架都不约而同地设计了更新合并。曾有一阵子，react中关于setState是同步还是异步的面试题几乎成为了一道必问的问题，而vue中nextTick相关的面试题也屡见不鲜。

本期将从nextTick开始，探究Vue中如何进行更新合并。

nextTick做了什么
------------

```js
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

nextTick接收一个函数参数，并且将这个函数放入了`callbacks`，之后调用了 `timerFunc()`,前者是一个空数组，后者其实就是一个微任务。

```javascript
const callbacks = []
...
let timerFunc
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
  }
}
```

`timerFunc`会调用`flushCallbacks`,这个函数会浅拷贝`callbacks`，并依次执行里面的函数

```ini
function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
```

因此nextTick其实就是将回调函数放入在当前EventLoop的微任务队列。

那为什么要这么设计呢？

> 在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

我们可以推测，**修改数据和dom的更新是异步的**，因此要获取更新后的dom，也需要等待异步。那么这个异步是如何实现的呢？

我们先来看Vue从beforeCreate到created做了什么。

Vue init执行顺序
------------

\_init

callHook(vm, 'created')

initLifecycle

initEvents

initRender

callHook(vm, 'beforeCreate')

initInjections

initProvide

initState

defineReactive

observe(data, true)

ob = new Observer(value)

this.walk(value)

new Vue()

这里需要关注的是initState中的defineReactive

### defineReactive

这个方法里进行了大家熟知的`Object.defineProperty`操作：

```ts
export function defineReactive (
  obj: Object, key: string,val: any,customSetter?: ?Function,shallow?: boolean
) {
    ...
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
     ...
    },
    set: function reactiveSetter (newVal) {
      ...
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

Vue更新执行顺序
---------

修改data数据后，触发这里的set，set进行了数据的更改，然后调用了`dep.notify()`

```css
notify () {
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
```

这些subs便是一个个Watcher，经过依赖收集，这些watcher会被加入subs数组

```ts
 class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;
}
```

### Watcher

watcher有一个重要的成员vm，代表的是组件实例，它的update调用的是`queueWatcher`

```ts
class Watcher {
    vm: Component;
    ...
    update () {
        ...
         queueWatcher(this)
  }
}
```

### queueWatcher

```tsx
function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      queue.push(watcher)
    } else {
      let i = queue.length - 1
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
        ...
      nextTick(flushSchedulerQueue)
    }
  }
}
```

在最后又遇到了`nextTick`，可以看到从这里开始了异步更新。

异步更新的得与失
--------

要做性能优化就必定要做更新合并，要做更新合并，首要的问题是如何确定更新已经结束了？

异步更新的思路是认为：对data的修改是在宏任务队列中完成的，当进入微任务队列时，这些修改操作必定已经完成了，接下来只要一次性完成更新即可。

这种设计思路清晰，代码也比较简单，缺点是会造成数据和dom的不一致（毕竟数据改完后，dom还得等微任务后才会更新完），因此不得不借助nextTick

那么有没有同步更新完成更新合并呢？有，比如React16

点击后修改数据，并触发更新：

进入更新生命周期

根据这个新state执行一次性更新

合并多次setState生成的副本得到完整的state

委托事件

执行console.log(state)

执行setState data1

得到state副本，开启更新流程

执行setState data1

得到state副本，已在更新流程中

onClick

console.log(state)

setState data1

setState data2

触发onClick

同步的方式下，state和dom是保持统一的，dom不更新，state也不变。

但也有几个缺点，比如反直觉，setState的命名与逻辑不一致了，还不如叫`requestStateChange`。另外状态的修改，必须是在能够监控到的地方发生，比如说生命周期，自定义的委托事件。

当然这两种设计都能达到更新合并这一主要目标，只有取舍，没有好坏。

Vue是如何收集依赖的？
------------

当然再回到上面，有一个问题还没有解释：subs数组是何时放入了watcher。后续文章将从这个问题继续解析源码。