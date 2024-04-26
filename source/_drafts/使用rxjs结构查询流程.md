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

  changeQueryCondition = (condition) => {
    this.queryCondition = condition;
  };

  changeResultCondition = (condition) => {
    this.resultCondition = condition;
  };
}

const subjectCondition$ = new Rx.Subject();
const subjectTableData$ = new Rx.Subject();
const conditionSubscribe = subjectCondition$.subscribe(({ condition, fetchTable }) => {
  contitionDataStore.changeQueryCondition(condition);
  if (fetchTable) {
    tableDataStore.fetchTableData();
  }
});
const tableDataSubscribe = subjectTableData$.debounce(100).subscribe(() => {
  tableDataStore.fetchTableData();
});
const tableDataStore = new TableDataStore();
const contitionDataStore = new ContitionDataStore();
