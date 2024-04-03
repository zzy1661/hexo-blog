---
 title: "三分钟搞懂js属性描述符之基础知识"
 date: 
 tags: [ECMAScript 6]
 categories: 
---

前言：这年头所有用过vue的人都知道get，set，Object.defineProperty，可听到属性描述符却有点懵。另外实际使用的场景有哪些，又有哪些坑可能很多人并不清楚，因此打算分两篇文章，一篇讲基础知识，另一篇讲实际应用中的坑。

什么是属性描述符对象
----------

下面这么个对象就是完整的属性描述符。

```
{
    configurable: false,
    enumberable: false,
    value: 'value',
    writable: false,
    get: function() {
        console.log('get')
    },
    set: function(val) {
        console.log('set')
    }
}
```

当然正常情况下你不会看到完整的属性描述符。它的每个属性都是**可选的**。当一个属性描述符没有没有get和set时，它是个**数据描述符**。当它没有value和writable时，它是个**存取描述符**。如果一个描述符同时有(value或writable)和(get或set)关键字，将会产生一个异常。

描述符可同时具有的键值

PropertyDescriptor

configurable

enumerable

value

writable

get

set

数据描述符

Yes

Yes

Yes

Yes

No

No

存取描述符

Yes

Yes

No

No

Yes

Yes

每种属性的含义
-------

`configurable`  
当且仅当该属性的 configurable 为 true 时，该属性`描述符`才能够被改变，同时该属性也能从对应的对象上被删除。

`enumerable`  
当且仅当该属性的`enumerable`为`true`时，该属性才能够出现在对象的枚举属性中。

`value`  
该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。

`writable`  
当且仅当该属性的`writable`为`true`时，`value`才能被[赋值运算符](https://developer.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FOperators%2FAssignment_Operators "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators")改变。

`get`  
一个给属性提供 getter 的方法，如果没有 getter 则为 `undefined`。当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入`this`对象（由于继承关系，这里的`this`并不一定是定义该属性的对象）。

`set`  
一个给属性提供 setter 的方法，如果没有 setter 则为 `undefined`。当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。

### 关于默认值

以上的定义和解释来自于mdn，但是删除了默认值的说明。默认值分两种语境，一个是我没有设置属性描述符时，就去取它；另一种是我设置属性描述符，但是我省略了部分属性。**mdn上的默认值是后者**。

获取属性描述符
-------

`Object.getOwnPropertyDescriptor`和`Object.getOwnPropertyDescriptors`,两者模式差不多，懂一个就懂另一个了。  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d5d37237e~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
可见其中writable、enumerable、configurable默认是true

设置属性描述符
-------

`Object.defineProperty`和`Object.defineProperties`，同样一个就懂另一个了。  
我们分别设置get和value，然后再获取属性描述符看看区别  
get:  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d5d34e349~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
value:  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d5d4ce573~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
可以看到，当我设置了属性描述符后writable、enumerable、configurable默认是false。  
**enumerable为false时**  
并且细心的同学发现了，对象的这个属性名的颜色有点不一样。不过此时我们先忽略这个颜色的区别。  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d5d506699~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
我们将它转为json字符串，发现没有age和address。嗯，这就是因为enumerable设置成了false，使得它被转为字符串的过程中无法遍历到这两个属性。我们可以将enumerable设置成true再试一下。  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d864d0ad5~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
**configurable为false时**  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d8b63e926~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")  
因为缺省的configurable默认为false，当我们再次设置时就报错了。此时就陷入了无解的状态，因此我们平时开发中需要设置属性描述符时，**最好不要缺省enumerable和configurable**。  
我们从头来一次：  
![image.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f5ba4d8c24e694~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp "image.png")

关于属性描述符，我想大家应该都有一个大致的印象了，对于get和set我在后面实际应用中再解释吧~