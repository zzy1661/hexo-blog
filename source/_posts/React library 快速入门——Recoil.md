---
 title: "React library 快速入门——Recoil"
 date: 
 tags: [前端]
 categories: 
---

这是我参与11月更文挑战的第14天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095/ "https://juejin.cn/post/7023643374569816095/")

介绍
--

Recoil是React的一个轻量且高性能状态管理库，它的目标是解决一下几个问题：

> *   组件间的状态共享只能通过将 state 提升至它们的公共祖先来实现，但这样做可能导致重新渲染一颗巨大的组件树。
> *   Context 只能存储单一值，无法存储多个各自拥有消费者的值的集合。
> *   以上两种方式都很难将组件树的顶层（state 必须存在的地方）与叶子组件 (使用 state 的地方) 进行代码分割。

快速上手
----

在介绍它的api之前我们先看一下应用在项目中应用Recoil是有多么简单：

*   首先是引入`RecoilRoot`并将其放在根组件的位置（也可以放在其他父组件位置上）

```javascript
import {RecoilRoot} from 'recoil';
...
<RecoilRoot>
   <App />
</RecoilRoot>
```

*   设置公共状态：`atom` store.js

```arduino
export const cloudClassSpuState = atom<number>({
    key: 'cloudClassSpuState', 
    default: 702 
});
```

一个公共状态可以用一个`atom`表示

*   使用和修改公共状态：`useRecoilState`

```javascript
import { cloudClassSpuState} from '../store.js';
import {useRecoilState} from 'recoil';
...
export default function StudyRoom() {
    const [cloudClassSpu, setSpu] = useRecoilState<number>(cloudClassSpuState);
    return <div>{cloudClassSpu}</div>
}
```

`useRecoilState`返回的是一个数组，第一个值是公共状态，第二个值就是对这个公共状态的修改方法，调用这个方法后其他有依赖到这个状态的组件自动会进行更新。

以上就是Recoil的级别用法了，核心的api就是`atom`和`useRecoilState`

其他常用api
-------

*   `selector`和`useRecoilValue` selector表示派生状态，类似于`vue`中的`computed`,

```ini
const todoListStatsState = selector({
  key: 'todoListStatsState',
  get: ({get}) => {
    const todoList = get(todoListState);
    const totalNum = todoList.length;
    const totalCompletedNum = todoList.filter((item) => item.isComplete).length;
    const totalUncompletedNum = totalNum - totalCompletedNum;
    const percentCompleted = totalNum === 0 ? 0 : totalCompletedNum / totalNum * 100;

    return {
      totalNum,
      totalCompletedNum,
      totalUncompletedNum,
      percentCompleted,
    };
  },
});
```

而要使用`selector`则需要`useRecoilValue`

```javascript
function TodoListStats() {
  const {
    totalNum,
    totalCompletedNum,
    totalUncompletedNum,
    percentCompleted,
  } = useRecoilValue(todoListStatsState);

  const formattedPercentCompleted = Math.round(percentCompleted);

  return (
    <ul>
      <li>Total items: {totalNum}</li>
      <li>Items completed: {totalCompletedNum}</li>
      <li>Items not completed: {totalUncompletedNum}</li>
      <li>Percent completed: {formattedPercentCompleted}</li>
    </ul>
  );
}
```

这里在定义`selector`的时候只设置了`get`而没有设置`set`，因此这个`selector`是只读的

另外`selector`的get可以返回一个`promise`，在获取`selector`中值的时候会等待这个`promise`吐出结果，这就是异步`selector`

*   `useSetRecoilState` 当使用`useRecoilState`获取状态时，组件也同时会订阅该状态，一旦发生变更就会进行更新，但是有些组件仅仅需要进行状态的修改而不需要渲染该状态，这时候就可以用这个api

```javascript
import { useSetRecoilState} from 'recoil';
import {infoState} from '../store';

function form() {
  const setInfoState = useSetRecoilState(infoState);\
  return (
      <button onClick={() => setInfoState({end:false})}>set info</button>
)

```

*   `useResetRecoilState` 这个api可以将某个state重设为默认值，比如用在表单的重置按钮上
    
*   `useRecoilStateLoadable` 这个api可以读取异步`selector`,并返回一个[`Loadable`](https://www.recoiljs.cn%2Fdocs%2Fapi-reference%2Fcore%2FLoadable "https://www.recoiljs.cn/docs/api-reference/core/Loadable") 对象，可以从这个对象中获知值是否在加载中、已完成、出错。
    

掌握以上这些常用api就可以在项目中启用`recoil`了，相比`redux`,这个库对状态的管理和组织更为灵活。