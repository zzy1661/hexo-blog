---
 title: "【翻译】`at`将加入ECMAScript"
 date: 2021-10-14
 tags: [前端]
 categories: [前端笔记]
---

如果你是个js开发者，你看你经常使用数组。它是语言中最基本的数据结构

事实上，正因为它太过基础，过去的几年里数组的 prototype 迅速扩充，添加了诸如`flat`和`filter`等方法，并且还在继续增加。

访问器
---

为了方位一个数组中的元素，你需要知道它的下标，JavaScript中的下标是从0开始的，因此第一个元素的下标是0.

```less
const arr = ['a', 'b', 'c', 'd']
arr[0] // this is "a"
arr[2] // this is "c"
```

正如你在上面的例子中看到的，你可以访问到第一个或者第三个元素。但是最后一个呢？在其他语言中，你可以通过类似这样的代码：

```less
const arr = ['a', 'b', 'c', 'd']
arr[-1] // This is NOT "d"
```

但是在JavaScript中不行！真的不行吗？... 现在`-1`已经是一个合法的key了。数组实际上是将index作为key的对象，因此`arr[-1]`是在查询`arr`对象的 `"-1"`key上的值，该值为`undefined`

最后一个元素
------

那么我们该如何不适用最后一个元素的下标就可以访问到它呢？的确有办法做到，但是它显然有些冗长：你可以使用length查找

```less
arr[arr.length - 1] // this is "d"
```

你也可以使用slice

```python
arr.slice(-1)[0] // this is "d"
```

AT
--

这就是`at`加入到JavaScript的原因。你能够用这样的写法代替上面的做法

```python
arr.at(-1) // this is "d"
```

`at`最棒的一点是，你再也不需要使用方括号了。

```scss
arr.at(0) // this is still "a"
```

如果传入了不合法的下标会怎样？

```scss
arr.at(5) // this is undefined
```

看上去各种写法的容忍度很高。

历史插曲
----

其实以前提出过类似的`item`api，但是它因为与主流的库冲突而不被web兼容。因此`at`成为当前的提案。

你愿意用它吗?
-------

这个提案已经被官方采纳了，并且将会成为ES2022的一部分。能预见到它将会成为对访问数组元素非常友好的语法糖。

原文：[laurieontech.com/posts/at-pr…](https://laurieontech.com%2Fposts%2Fat-proposal%2F "https://laurieontech.com/posts/at-proposal/")