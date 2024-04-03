---
 title: "Mobx6核心源码解析（四）: ObservableValue"
 date: 2021-12-06
 tags: [前端,MobX]
 categories: [前端笔记]
---

前言
--

```ini
let target = observable(obj)
```

在Mobx中`Atom`用来监测数据被观察和修改，在设置target的属性描述符的时候，对应每一个key都会创建一个一个`Atom`的子类`ObservableValue`实例。

ObservableValue
---------------

如果`obserable(obj)`的返回值target是"_明_",而obj是"_暗_"，那么`ObservableValue`实例则是"_明_"通往"_暗_"的单向桥梁。对target相应key的getter和setter都调用了这个实例的相应方法。

*   value\_ 这个属性保存了obj对象的对应的key的`descriptor.value`
*   get： 当访问target的属性的时候，会调用这个方法。返回的就是原obj对象对应key的相应value`reportObserved`，

```kotlin
 public get(): T {
    this.reportObserved()
    return this.dehanceValue(this.value_)
}
```

get的调用说明该key被依赖，其会调用`reportObserved`处理依赖。

*   reportObserved 这个方法继承自父类`Atom`,该函数中最有价值的事情是确定依赖关系。

```java
export function reportObserved(observable: IObservable): boolean {
    ...
    const derivation = globalState.trackingDerivation
    if (derivation !== null) {
       
        if (derivation.runId_ !== observable.lastAccessedBy_) {
            observable.lastAccessedBy_ = derivation.runId_
          
            derivation.newObserving_![derivation.unboundDepsCount_++] = observable
            if (!observable.isBeingObserved_ && globalState.trackingContext) {
                observable.isBeingObserved_ = true
                observable.onBO()
            }
        }
        return true
    } else if (observable.observers_.size === 0 && globalState.inBatch > 0) {
        queueForUnobservation(observable)
    }

    return false
}
```

`globalState.trackingDerivation`是一个`IDerivation`类型的数据，或者称之为"_派生_"，它有一个大名鼎鼎的实现类，叫`Reaction`。可以简单理解：**derivation是observable数据的观察者。** 当这段代码运行的时候就会创建出一个`reaction`，并挂到`globalState.trackingDerivation`上，这个过程在解析`autorun`源码的时候再具体分析。

```scss
auturun(()=>{...})
```

总之，这里通过`derivation.newObserving_![derivation.unboundDepsCount_++] = observable`,确定了`derivation`对这个`ObservableValue`实例有依赖。

*   set和setNewValue 当对target设置/修改值的时候会调用`set`或`setNewValue`，set内部也调用了`setNewValue`.

```kotlin
public set(newValue: T) {
    const oldValue = this.value_
    newValue = this.prepareNewValue_(newValue) as any
    ...
    if (newValue !== globalState.UNCHANGED) {
        this.setNewValue_(newValue)
    }
}
setNewValue_(newValue: T) {
    const oldValue = this.value_
    this.value_ = newValue
    this.reportChanged()
   ...
}
```

`setNewValue`触发时说明value变更，需要通知依赖进行相应的处理，这里调用的是`reportChanged`。

*   reportChanged 这个方法也继承自父类`Atom`

```scss
 public reportChanged() {
        startBatch()
        propagateChanged(this)
        endBatch()
    }
```

`propagateChanged`从`ObservableValue`实例`observable`中找到对它有依赖的所有`derivation`，并调用`onBecomeStale_`,这个方法在`Reaction`中定义，它在经过了一系列复杂的调用后，能够使得依赖的函数再次运行。

```ini
export function propagateChanged(observable: IObservable) {
    // invariantLOS(observable, "changed start");
    if (observable.lowestObserverState_ === IDerivationState_.STALE_) return
    observable.lowestObserverState_ = IDerivationState_.STALE_

    // Ideally we use for..of here, but the downcompiled version is really slow...
    observable.observers_.forEach(d => {
        if (d.dependenciesState_ === IDerivationState_.UP_TO_DATE_) {
            if (__DEV__ && d.isTracing_ !== TraceMode.NONE) {
                logTraceInfo(d, observable)
            }
            d.onBecomeStale_()
        }
        d.dependenciesState_ = IDerivationState_.STALE_
    })
    // invariantLOS(observable, "changed end");
}
```

未完待续
----

Mobx中“观察”部分的代码到这里就基本结束了，但是遗留了两个问题：`globalState.trackingDerivation`是何时确定的以及`observable.observers_`又是如何收集的。

下一节将开始Mobx“反应”部分的源码解析，回答这两个问题