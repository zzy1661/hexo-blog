---
 title: "抛开State和Context谈前端组件通信(一)——实例劫持+中介者通信"
 date: 2021-11-16
 tags: [前端]
 categories: [前端笔记]
---

这是我参与11月更文挑战的第16天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095 "https://juejin.cn/post/7023643374569816095")

前言
--

说起前端组件通信，大家都能想到好几个办法：父子props相传、提升公共state、使用Context，然而这些常规方法，往往依赖于框架本身的api，或者受限于组件之间的关系。那么抛开框架，两个独立的组件，我们应该如何让他们相互通信呢？

我们不妨设想一个场景：页面上有两个标签页打开着：TabA和TabB, Tab A页面有一个按钮，点击后请求一个接口，然后关闭当前Tab，Tab B请求一个接口并弹出一个对话框组件,能够控制Tab 关闭的的组件是Tab Container C。

关系分析
----

这里有三个对象：

`class component TabA`,

`class component TabB`,

`class component TabContainerC`，

还有三个关键方法：

`TabA`的`function leave(){}` ,

`TabContainerC`的`closeTab` 和

`TabB`的`openDialog`。

param: 'A'

tabA.leave()

tabC.closeTab(param)

tabB.openDialog()

三个函数的管理和调用权应该在它所在的组件实例上，如果强行提升公共状态和方法，代码会既复杂又不容易维护。

如果三个组件能相互调用，那么代码就会非常简单。

劫持组件实例
------

利用TS的装饰器，我们可以很容易地劫持到组件实例

```TS
const ins = {};
function colleague(colleagueName: string) {
    return function<T extends {new (...args: any[]): {}}>(target: T) {
        // 劫持构造函数，获得this
        let colleague: Colleague<any> = null;
        return class extends target {
            constructor(...arg: any[]) {
                super(...arg);
                // 获取this并放入集合中
                ins[target.displayName] = this
            }
            componentWillUnmount() {
                // 伪代码，卸载时清除被劫持的实例
                delete ins[target.displayName]
            }
        };
    };
},

```

使用装饰器

```TSX
@colleague()
class TabContainerC extends React.Component<PropsTypes> {
    close(param:string) {
        ...
    }
}
```

TabA调用TabContainerC

```TSX
class TabA extends React.Component<PropsTypes> {
    leave(){
        ins['TabContainerC']?.close('A')
    }
}
```

中介者模式
-----

上面的写法看似能解决问题，但有好几个缺陷，最大的问题就是**无序调用**。

A

B

C

D

E

如果`TabContainerC`的某个方法因为某些额外参数的问题，并不想对外暴露，怎么做屏蔽？

如果项目组件有1000个，如何控制它们之间的调用，比如在调用时打个小小的`console.log`

这时候便可以考虑中介者设计模式

A

X

B

C

D

E

X为中介者，负责给其他成员进行沟通，那么X就可以对所有成员的交流进行管控：比如日志、容错、过滤等

概念设计和Api设计
----------

### 两种角色

*   Mediator ：中介者
    *   用于管理成员间的消息传递。Colleague 注册后，Mediator 能够获得这个成员的信息，当需要调用某个 Colleague 的方法时，可以通知 Mediator 去找到对应的 Colleague
*   Colleague ：成员
    *   组件实例的描述，包括组件的实例，组件公开的方法。以成员名作为唯一标志，成员名默认是这个组件的名称，也支持自定义

### 三个API

*   `@colleague(colleagueName)` : 注册为成员,参数 colleagueName 是**必选**的成员名
*   `@colleagueAction(actionName)`: 声明公开的方法，actionName 是方法标志
*   `@notifyMediator(colleagueName, actionName,params?)`： 通知中介，让名为 colleagueName 的成员执行 actionName 这一公开的方法,并可以传递params参数

使用方法：

```tsx
@colleague('TabContainerC')
class TabContainerC extends React.Component<PropsTypes> {
    ...
    @colleagueAction('closeTab')
    close(params:string) {
        ...
    }
    ...
}

@colleague('TabA')
class TabA extends React.Component<PropsTypes> {
    @notifyMediator('TabContainerC', 'closeTab','A')
    leave(newAccount: AccountManageModel.Account) {
        ...
    }
}
```

### 若干细节

*   `reflect-metadata`：方法装饰器在类装饰器前执行，导致处理公开方法时，还没有相应的成员生成，需要使用 reflect-metadata 对其标识，再在重写构造函数时遍历获取
*   打包后class名会变，因此注册成员是,参数 colleagueName 是**必选**的成员名
*   当返回promise时，支持异步调用其他实例

代码参考
----

### 目录结构如下：

```
├───Colleague.ts
├───index.ts    
├───Mediator.ts 
└───readme.md   
```

### Colleague.ts:

```TS
/**
 * 成员类
 * 含有成员名称和源实例
 *
 */
export default class Colleague<T> {
    actions = new Map<string, (payload?: any) => void>();

    constructor(public name: string, private instance: T) {}
    setInstance(instance: T) {
        this.instance = instance;
    }
    getInstance() {
        return this.instance;
    }

    setAction(actionName: string, fn: (payload?: any[]) => void) {
        this.actions.set(actionName, fn);
    }

    // 执行特定的方法
    performAction(actionName: string, payload?: any) {
        const action = this.actions.get(actionName);
        if (!action) {
            console.warn('there is no action names ' + actionName + ' in colleague' + this.name);
        } else {
            return action.apply(this.instance, payload);
        }
    }
}
```

### Mediator.ts：

```TS
import Colleague from './Colleague';
/**
 * 中介者类
 * 用于收集成员
 * 转发成员间消息
 */
export default class Mediator {
    // 中介处登记的成员
    colleagues = new Map<string, Colleague<any>>();

    // 登记成员
    registColleague(colleague: Colleague<any>): void {
        this.colleagues.set(colleague.name, colleague);
    }
    // 清退成员
    unRegistColleague(colleague: Colleague<any>): void {
        this.colleagues.delete(colleague.name);
    }
    // 通知
    notify(colleagueName: string, actionName: string, payload?: any[]): void {
        const colleague = this.colleagues.get(colleagueName);
        if (!colleague) {
            console.warn('there is no colleague names ' + colleagueName);
        } else {
            colleague.performAction(actionName, payload);
        }
    }
}

```

### index.ts:

```TS
import Mediator from './Mediator';
import Colleague from './Colleague';
import 'reflect-metadata';

function mediatorFactory() {
    const mediator = new Mediator();
    console.log(mediator);
    // 创建闭包
    return {
        // 类装饰器
        colleague: function (colleagueName: string) {
            return function <T extends {new (...args: any[]): {}}>(target: T) {
                    // 劫持构造函数只为获得this
                    let colleague: Colleague<any> = null;
                    return class extends target {
                            static displayName = (target as any).displayName + '$Colleague';
                            constructor(...arg: any[]) {
                                    super(...arg);
                                    colleague = new Colleague(colleagueName, this);
                                    // 从元数据中获取保存的公开的方法
                                    Reflect.ownKeys(target.prototype).forEach(key => {
                                            if (typeof key === 'number') {
                                                    key = key.toString();
                                            }
                                            const actionMethod = Reflect.getMetadata('action', target.prototype, key);
                                            if (actionMethod) {
                                                    colleague.setAction(actionMethod.name, actionMethod.value);
                                            }
                                    });
                                    mediator.registColleague(colleague);
                            }
                            componentWillUnmount() {
                                    mediator.unRegistColleague(colleague);
                                    let willUnMount = target.prototype.componentWillUnmount;
                                    willUnMount && willUnMount.call(this);
                            }
                    };
            };
        },
        // 方法装饰器，开放的方法
        colleagueAction: function (actionName: string) {
            return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
                // 将开放的方法放入中介,此时还未执行类装饰器，需要先保存起来
                Reflect.defineMetadata('action', {name: actionName, value: descriptor.value}, target, propertyKey);
            };
        },
        // 方法装饰器
        notifyMediator: function (colleagueName: string, actionName: string, ...payload: any[]) {
            return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
                // 劫持方法，通知中介
                return {
                    writable: true,
                    enumerable: true,
                    configurable: true,
                    value: function (...args: any[]) {
                        // tslint:disable-next-line:no-invalid-this
                        let result = descriptor.value.apply(this, args);
                        if (result instanceof Promise) {
                            result.then(function () {
                                mediator.notify(colleagueName, actionName, payload);
                            });
                        } else {
                            mediator.notify(colleagueName, actionName, payload);
                        }
                        return result;
                    }
                };
            };
        }
    };
}

export const {colleague, colleagueAction, notifyMediator} = mediatorFactory();

```

结尾
--

实例劫持的方式虽然api简单，思路清晰，但是在无ui组件实例的情况下（比如hooks）显然是无法使用的，下一遍便介绍另一个方法: **HOT EVENT**。