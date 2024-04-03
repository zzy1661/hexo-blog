---
 title: "React library 快速入门——React Query"
 date: 2021-11-08
 tags: [前端]
 categories: [前端笔记]
---

这是我参与11月更文挑战的第 8 天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095/ "https://juejin.cn/post/7023643374569816095/")

前言
--

React的生态非常庞大，各种lib层出不穷，本系列将介绍其中最受欢迎的一些lib，并给出快速入门的实践案例。

本期介绍的是React Query。

概览
--

React Query是一个管理请求数据的库。

> React Query is often described as the missing data-fetching library for React, but in more technical terms, it makes **fetching, caching, synchronizing and updating server state** in your React applications a breeze.

看官网解释可能有些费解，上代码：

```javascript
import React from "react";
import ReactDOM from "react-dom";
import { QueryClient, QueryClientProvider, useQuery } from "react-query";
import { ReactQueryDevtools } from "react-query/devtools";

const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}

function Example() {
  const { isLoading, error, data, isFetching } = useQuery("repoData", () =>
    fetch(
      "https://api.github.com/repos/tannerlinsley/react-query"
    ).then((res) => res.json())
  );

  if (isLoading) return "Loading...";

  if (error) return "An error has occurred: " + error.message;

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
      <strong>👀 {data.subscribers_count}</strong>{" "}
      <strong>✨ {data.stargazers_count}</strong>{" "}
      <strong>🍴 {data.forks_count}</strong>
      <div>{isFetching ? "Updating..." : ""}</div>
      <ReactQueryDevtools initialIsOpen />
    </div>
  );
}

const rootElement = document.getElementById("root");
ReactDOM.render(<App />, rootElement);
```

这个例子很好地体现出了我们对一个请求最关心的三个部分：返回值、错误和请求loading。react query并不替你发送请求，它做的是对请求的管理。

核心api
-----

### useQuery

当需要查询数据的时候使用`useQuery`

#### 语法

```ini
 const result = useQuery({
   queryKey,
   queryFn,
   enabled,
 })
```

#### 参数说明

```typescript
  const [page, setPage] = useState<{
    current: number;
    pageSize: number;
  }>({ current: 1, pageSize: 10 });
  const [params, setParams] = useState<{ name: string; status: number }>();
  
 const { data, isLoading, refetch } = useQuery(
    ['files', page, params],
    () => {
      return fetch({
        url: '/api/files',
        method: 'POST',
        body: { page, params },
      });
    },
    {
      onSuccess(data) {
        console.log(data);
      },
    },
  );
```

这段代码里，在page和params变化的时候会自动进行请求，因此不需要额外用户操作后再次发起请求更新数据。 但是默认情况下，第一次进来也会进行一次请求，另外如果请求失败，会进行三次重试。这是react-query默认设置的。

*   `queryKey`
    *   **必传**
    *   用于此查询的query key。
    *   当此键更改时，查询将自动更新（只要`enabled`未设置为`false`）。
*   `queryFn:(context: QueryFunctionContext) => Promise<TData>`
    *   在没有指定默认的query function时**必传**
    *   可以从context中获取`queryKey`
    *   必须返回一个promise
*   第三个参数可以传一个对象，其中几个常用的属性如下：
    *   enabled： 可以关闭自动更新
    *   retry: 请求失败重试，可以关闭或传入重试次数
    *   onSuccess: 成功回调
    *   onError：失败回调
    *   refetchOnWindowFocus: 如果数据过期了，当window focus时会进行请求（这也是默认设置），可以关闭该特性
    *   staleTime：多长时间数据会过期

### QueryClient

以上选型可以通过QueryClient设置默认值：

```php
const queryClient = new QueryClient();
// 默认不在挂载和focus时自动运行，不重试，但是默认enable为true，根据依赖变化自动运行
queryClient.setDefaultOptions({
  queries: {
    refetchOnWindowFocus: false,
    refetchOnMount: false,
    refetchOnReconnect: false,
    retry: false,
  },
});
```

### useMutation

当需要变更数据时使用useMutation

```ini
const Upload: FC<UploadProps> = function (props) {
  const component = props.dragger ? FileSelector.Dragger : FileSelector;

  const { mutate, isLoading } = useMutation(
    (file: File) => {
      const formData = new FormData();
      formData.append('file', file);
      formData.append('name', file.name);
      return fetch({ url: '/api/upload', method: 'POST', body: formData });
    },
    {
      onSuccess(data) {
        message.success('文件上传成功');
        props.onSuccess?.(data);
      },
      onError(err) {
        message.error('文件上传失败');
        props.onError?.(err);
      },
    },
  );
  const upload = (f: RcCustomRequestOptions) => {
    mutate(f.file);
  };
  return React.createElement(
    component,
    { customRequest: upload, ...props },
    props.children || props.dragger ? (
      <p className="p-6">点击或拖拽文件到此区域</p>
    ) : (
      <Button
        type="primary"
        htmlType="button"
        loading={isLoading}
        disabled={isLoading}
      >
        <Icon type={props.icon || 'upload'} className="align-middle" />
        <span className="align-middle"> {props.text || '上传文件'}</span>
      </Button>
    ),
  );
};
```

上面这段代码是一个简单的上传组件：

用户点击按钮

选择文件

发起请求

![image.png](../imgs/af4554dd19ce4a6aa911769fc36a839f.png) 这里的请求需要我们主动发起，这时候就需要调用`useMutation`返回的`mutate`

#### 参数

```scss
 const {
   data,
   error,
   isError,
   isIdle,
   isLoading,
   isPaused,
   isSuccess,
   mutate,
   mutateAsync,
   reset,
   status,
 } = useMutation(mutationFn, {
   mutationKey,
   onError,
   onMutate,
   onSettled,
   onSuccess,
   useErrorBoundary,
 })

mutate(variables, {
   onError,
   onSettled,
   onSuccess,
 })

```

*   `mutationFn: (variables: TVariables) => Promise<TData>`
    
    *   **必传**
    *   一个返回promise的函数。
    *   参数`variables`由`mutate`传过来
*   `onSuccess: (data: TData, variables: TVariables, context?: TContext) => Promise<unknown> | void`
    
    *   可选
    *   成功回调
*   `onError: (err: TError, variables: TVariables, context?: TContext) => Promise<unknown> | void`
    
    *   可选的
    *   失败回调
*   `retry: boolean | number | (failureCount: number, error: TError) => boolean`
    
    *   如果设置为`false`，失败后不会重试。