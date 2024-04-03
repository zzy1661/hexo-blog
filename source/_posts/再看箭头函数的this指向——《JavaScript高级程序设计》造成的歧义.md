---
 title: "再看箭头函数的this指向——《JavaScript高级程序设计》造成的歧义"
 date: 2022-03-16
 tags: [前端,JavaScript]
 categories: [前端笔记]
---

箭头函数中的this指向什么？

这是一道经典JS问题。  
很多人都认为，**箭头函数中的this，指向的是它定义时的环境中的this**。这个观点来自于《JavaScript高级程序设计》，它关于箭头函数的this有如下解释：

> 在箭头函数中，this引用的是定义箭头函数的上下文。 ...

> 事件回调或定时回调中调用某个函数时，this值指向的并非想要的对象。此时将回调函数写成箭头函数就可以解决问题。这是因为箭头函数中的this会保留定义该函数时的上下文

并且还给出了一个经典例子：

```js
window.color = 'red';
let o = {
    color:'blue'
}
let sayColor = ()=>console.log(this.color)
sayColor(); //red
o.sayColor = sayColor;
o.sayColor(); //red
```

根据它的解释，我们可以这么理解：定义sayColor时的上下文中的this是window，因此这个函数中的this始终绑定了window。

**但是这个解释是有问题的!**

**上下文和this都是运行时概念**，当定义sayColor的上下文发生变化时，其this指向也会发生变化。

我们来看下面一个例子：

```js
const obj = {
  do() {
    return () => {
      console.log(this);
    };
  },
};

```

这里匿名箭头函数中的this指向的是什么？**这个问题其实无法回答，要看它是怎么执行的。**

```js
// 第一次调用
obj.do()(); // obj

// 第二次调用
const do1 = obj.do;
do1()(); // window

// 第三次调用
const do2 = obj.do();
do2(); // obj
```

这里可以看到this的指向会根据不同的上下文而发生变化。第一次和第三次调用中，匿名箭头函数定义时的上下文环境中的this是obj，而第二次调用时是window。

如果觉得有点费解，**将“定义”改为“声明”**——“在箭头函数中，this引用的是箭头函数声明时所在上下文”，这样就消除了歧义。

最后我们再来看一下《JavaScript权威指南第七版》中的解释：

“箭头函数不同于用关键字定义的函数：箭头函**数从定义它们的环境继承 this 关键字**，而不是像其他定义方式那样定义自己的调用上下文。”

> Arrow functions differ from functions defined in other ways in one critical way: they inherit the value of the this keyword from the environment in which they are defined rather than defining their own invocation context as functions defined in other ways do.

这段话可以浓缩为两个字“**继承**”（inherit），虽然看起来并不容易理解，但不会造成歧义。。。（怪不得各种官方定义都那么的费解）