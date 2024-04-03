---
 title: "React18中的新特性——Automatic batching"
 date: 
 tags: [前端,React.js]
 categories: 
---

「这是我参与2022首次更文挑战的第5天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

从一道经典面试题开始——setState是同步的还是异步的
-----------------------------

曾有一段时间，这道题目几乎成为了react面试的必问题。

很多人都知道答案：在React的生命周期、合成事件中它是异步的；但是在异步事件、原生事件中，又是同步的

但这个答案仅仅是表象。

这个问题的根源是React或者Vue这种，由数据驱动视图的框架必须回答的一道命题——当数据改变时，如何减少渲染次数。

而这两个框架选择了截然不同的答案。Vue选择了异步更新。而React则是同步更新，合并state：也即setState触发生命周期的流转，但直到调用render前，才对state进行合并，从而表现出“异步”。然而在异步事件或原生事件脱离了react的掌控，没有开启批量更新，从而表现出同步，因此说setState“异步”，是一定要打上引号的。

但是2022年，有经验的面试官不会再问这道题了，因为React18中不会再出现这种既同步又“异步”的怪异现象了。

React18中的Automatic batching
---------------------------

在React18中，如果是调用`ReactDOM.createRoot(rootElement).render(<App />);`渲染根组件的话，将会开启Automatic batching。

### 什么是batching

batching就是合并re-render。

在React17或更早的版本中，下面的代码只会触发一次re-render。

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    setCount(c => c + 1); // Does not re-render yet
    setFlag(f => !f); // Does not re-render yet
    // React will only re-render once at the end (that's batching!)
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

但是在下面这段代码中，会触发两次： 虽然合成事件onClick的callback会开启batch,但是setCount和setFlag是在这个callback之后执行的(而不是在batching期间),因此会触发两次re-render。

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      setCount(c => c + 1); // Causes a re-render
      setFlag(f => !f); // Causes a re-render
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

### 什么是automatic batching

如果你的根组件渲染是通过`ReactDOM.createRoot(...).render(...)`，那么在下面的代码中，两次改变state只会触发一次re-render，即使它们在promise、setTimeout或原生事件中。

```javascript
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  function handleClick() {
    fetchSomething().then(() => {
      // React 18 and later DOES batch these:
      setCount(c => c + 1);
      setFlag(f => !f);
      // React will only re-render once at the end (that's batching!)
    });
  }

  return (
    <div>
      <button onClick={handleClick}>Next</button>
      <h1 style={{ color: flag ? "blue" : "black" }}>{count}</h1>
    </div>
  );
}
```

### React18中如何关闭Automatic batching

*   方案一： `ReactDOM.render(<App />, rootElement);`：只有`createRoot`才能开启批量更新
*   方案二：使用react-dom的`flushSync`

```scss
import { flushSync } from 'react-dom'; // Note: react-dom, not react

function handleClick() {
  flushSync(() => {
    setCounter(c => c + 1);
  });
  // React has updated the DOM by now
  flushSync(() => {
    setFlag(f => !f);
  });
  // React has updated the DOM by now
}
```