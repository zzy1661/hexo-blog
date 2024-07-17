
在react的设计理念中，UI = f(state)。react解决了f，但是state的管理是一个很复杂的问题。
我将一个前端视图的数据分为两部分：过程数据和结果数据。
结果数据，就是所有state的最终值。过程数据，就是state变更过程中涉及到的所有非state数据。
前端开发的多数复杂度在于，state的计算涉及到各种数据和逻辑，有接口数据，用户操作数据，页面生成数据，全局/临时变量等等。
state=fn({serverData, userData, pageData, globalData, tempData...})
这个计算过程可能很复杂，而react本身的render也很消耗性能，因此需要区分普通的data和state。
state总是一个结果数据，即render中不进行复杂的计算以得到需要的state，这部分逻辑需要独立出来。


按照这种思路，一切数据的变更都可以看做事件，整个流程应该是触发事件——>数据处理——>渲染。

这里的第一个问题是，数据处理完得到新的state后，如何触发渲染。
react18可以借助use-sync-external-store，老版本的则需要借助setState或则forceUpdate，目前有很多状态库可以帮助我们解决这个问题，我的选型是mobx。

不能要求每个开发控制state的粒度以减少重复更新，这部分心智负担完全是因react的缺陷造成的，而mobx的依赖更新机制可以弥补这个缺陷。

第二个问题是，事件机制的设计。
事件机制可以自己实现一个发布订阅中心，也可以简单使用原生的CustomEvent。也有不少优秀的开源库，比如redux，我demo中用的是rxjs，仅仅为了方便。
在项目实践中，如果没有持续的数据流和多流合并处理，rxjs可能并不是一个好的选择，上手难度高，应用场景少，大部分api都用不上。
```
const getData = () => new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve({
      rows: [{ id: 1, name: 'A' }],
      total: 1,
    });
  }, 2000);
});
class TableDataStore {
  dataSources = [];

  total = 0;

  isFetching = false;

  async fetchTableData() {
    if (this.isFetching) return;
    this.isFetching = true;
    const data = await getData();
    this.dataSources = data.rows;
    this.total = data.total;
    this.isFetching = false;
  }
}
class ContitionDataStore {
  queryCondition = {};

  resultCondition = {};

  setQueryCondition = (condition) => {
    this.queryCondition = condition;
  };

  setResultCondition = (condition) => {
    this.resultCondition = condition;
  };
}

class PageContext {
  constructor(setting) {
    this.setting = this.settingParser(setting);
  }
  settingParser(setting){
    return setting
  }
  mergeSetting(setting){
    this.setting = {...this.setting,...setting};
    return this.setting;
  }
  isFetchable(condition){
    return true;
  }


}
const pageContext = new PageContext({})

const conditionSubject = new Rx.Subject();
const tableFetchSubject = new Rx.Subject();

const conditionSubscribe = conditionSubject.subscribe(({resultCondition, queryCondition, fetchTable }) => {
  if(queryCondition){
    contitionDataStore.setQueryCondition(conqueryConditiondition);
  }
  if (fetchTable) {
    const canFetch = pageContext.isFetchable(contitionDataStore.queryCondition);
    if(canFetch){
      tableFetchSubject.next()
    }
  }
});
const tableDataSubscribe = subjectTableData$.debounce(100).subscribe(() => {
  tableDataStore.fetchTableData(contitionDataStore.queryCondition).then(()=>{
    conditionSubscribe.next({resultCondition:contitionDataStore.queryCondition})
  });
});
const tableDataStore = new TableDataStore();
const contitionDataStore = new ContitionDataStore();
```

