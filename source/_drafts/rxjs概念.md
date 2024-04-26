Observable：可观察对象
Observer：观察者
Subscription:订阅行为

```
var observable = Rx.Observable.create(function (observer) {

});
var subscription  = observable.subscribe(observer);
subscription.unsubscribe();

```
Subject： 主体
既是可观察对象，也是观察者
```
var subject = new Rx.Subject();
subject.subscribe(observer)

```
作为观察者
```
var subject = new Rx.Subject();
subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

var observable = Rx.Observable.from([1, 2, 3]);

observable.subscribe(subject); 
```
BehaviorSubject：保存着最近发送的值，并且当有新的观察者订阅时，会立即从 BehaviorSubject 那接收到“当前值”
ReplaySubject：保存n个旧值，当有新的观察者订阅时，会立即从 ReplaySubject 那接收到“旧值”
AsyncSubject： 只有当 Observable 发送了 complete 消息时，AsyncSubject 才会发送最后一个值


订阅当地新闻
首先有一个新闻中心，负责发布新闻
```
var subject = new Rx.Subject();
subject.subscribe({
  next: (v) => console.log('接受到新闻推送' + v)
});
subject.next('新闻1');
subject.next('新闻2');
```

如果订阅的是热点新闻，订阅时能获得当前或前一天的热点，可以使用 BehaviorSubject
```
var subject = new Rx.ReplaySubject('你已成功订阅');
const zhangsanSubscribe = subject.subscribe({
    next: (v) => console.log('接受到新闻推送' + v)
});
subject.next('新闻1');
```
如果要订阅最近的3个新闻，可以使用 ReplaySubject
```
var subject = new Rx.ReplaySubject(3);
const zhangsanSubscribe = subject.subscribe({
    next: (v) => console.log('接受到新闻推送' + v)
});

subject.next('新闻1');
subject.next('新闻2');
subject.next('新闻3');
subject.next('新闻4');
const lisiSubscribe = subject.subscribe({
    next: (v) => console.log('接受到新闻推送' + v)
});

const wangwuSubscribe = subject.take(2).subscribe({
    next: (v) => console.log('接受到新闻推送' + v)
});
```