---
 title: "asnyc/await的并行"
 date: 2019-12-31
 tags: [ECMAScript 6]
 categories: [前端笔记]
---

一直以为es7里面的async和await可以用来简化串行异步代码，而没有想到还能并行。  
说到底，这俩货不过是promise的语法糖，await的作用只是串行解析promise。  
通常我们这样写：

```javascript
function asyncAwaitFn(str) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(str)
        }, 1000);
    })
}
const parallel = async () => { //并行执行
    console.time('parallel')
    const parallelOne = await asyncAwaitFn('1');
    const parallelTwo = await asyncAwaitFn('2')
    console.log(parallelOne) //1
    console.log(parallelTwo) //2
    console.timeEnd('parallel') //2003.509033203125ms
}
parallel()
```

这是串行，显然最后的执行时间应该大于2000ms。

但如果换一种写法：

```javascript
function asyncAwaitFn(str) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            resolve(str)
        }, 1000);
    })
}
const parallel = async () => { //并行执行
    console.time('parallel')
    const parallelOne = asyncAwaitFn('1');
    const parallelTwo = asyncAwaitFn('2')
    console.log(await parallelOne) //1
    console.log(await parallelTwo) //2
    console.timeEnd('parallel') //1001.87255859375ms
}
parallel()
```

最后执行时间只要1000ms，显然是并行了。

**不过严谨来说，这依然是promise本身的并行罢了。**