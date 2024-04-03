---
 title: "在React中优雅地使用弹窗——useModal"
 date: 2021-11-18
 tags: [前端]
 categories: [前端笔记]
---

这是我参与11月更文挑战的第18天，活动详情查看：[2021最后一次更文挑战](https://juejin.cn/post/7023643374569816095 "https://juejin.cn/post/7023643374569816095")

前言
--

上一篇文章中介绍了以HOC的方法封装一个`withModal`api，将代码量压缩了一半。但是我们再思考一下这段代码：

```javascript
export default function TestPage() {
  const [visiable, setVisiable] = useState(false);
  // 打开弹窗
  const open = () => {
    setVisiable(true);
  };
  //关闭弹窗
  const close = () => {
    setVisiable(false);
  };
  //点击确定提交表单
  const submit = (ref: MutableRefObject<FormInstance>) => {
    ref.current.submit();
  };
  const afterSubmit = () => {
    close();
  };
  const UserFormModal = withModal({ title: '新建用户' }, { afterSubmit })(React.forwardRef(UserForm));

  return (
    <div>
      <div className="text-center">
        <Button type="primary" onClick={open}>
          新建
        </Button>
      </div>
      <UserFormModal visible={visiable} onCancel={close} onOk={submit} />
    </div>
  );
}
```

`TestPage`作为一个`Page`组件，需要关心`Modal`的关闭吗？

关注点分离
-----

对于这个表单弹窗，`TestPage`关心的是`open`，至于取消、提交、请求，这些操作的发生点都在组件内，因此也不应该由`TestPage`关心。 我们希望的调用方式是：

```tsx
const {open,Modal} = useModal(...)
...
<Button type="primary" onClick={open}>新建</Button>
```

useModal
--------

### 思路

控制弹窗显示的是Modal的visiable属性，如果利用hooks的`useState`api控制`visiable`，那么我们只要将`setVisiable`暴露出去便是外层想要的`open`方法了。

另外这里的弹窗，需要和表单进行交互，这并不是通用弹窗的必须逻辑，因此不如直接将这个Modal定位为FormModal，不再背负通用Modal的设计负担，这样无论是语义还是使用方式，都会简单很多。

### useFormModal

```tsx
import type { ModalProps } from 'antd';
import { Modal } from 'antd';
import type { FormInstance } from 'antd/es/form';
import React, { useState } from 'react';

const useFormModal = (modalProps: ModalProps, Slot: React.FC<any>) => {
  const [visiable, setVisiable] = useState(false);
  const open = () => {
    setVisiable(true);
  };
  const close = () => {
    setVisiable(false);
  };
  const FormModal = (slotProps: any) => {
    const onCancel = () => {
      close();
    };

    const ref = React.useRef<FormInstance>();
    const ok = () => {
      ref.current?.submit()
    };
    return (
      <Modal
        onCancel={onCancel}
        onOk={ok}
        visible={visiable}
        wrapClassName="modal-wrap"
        okText="提交"
        cancelButtonProps={{ shape: 'round' }}
        okButtonProps={{ shape: 'round' }}
        width={600}
        {...modalProps}
      >
        <Slot ref={ref} {...slotProps} afterSubmit={close} />
      </Modal>
    );
  };

  return {
    FormModal,
    open,
  };
};
export default useFormModal;

```

可以看到FormModal内部处理了表单提交和弹窗关闭的逻辑，同时还支持使用者传入自定义的Modal和Form组件的props，做到了高内聚和可扩展。

使用效果
----

```tsx
export default function TestPage() {


  const {open,FormModal: UserModal} = useFormModal({title:'新建用户'},React.forwardRef(UserForm))

  return (
    <div>
      <div className="text-center">
        <Button type="primary" onClick={open}>
          新建
        </Button>
      </div>
      <UserModal />
    </div>
  );
}
```

至此，我们成功地将一个70行左右的代码，压缩到15行以内。而干净的调用方式，给我们带来的，不仅是本次迭代工作量的减负，代码越少，可读性和可维护性越高，即便后续有多次迭代和需求变更，也可以尽可能地通过小范围改动安全地完成。