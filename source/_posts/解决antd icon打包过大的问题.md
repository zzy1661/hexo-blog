---
 title: "解决antd icon打包过大的问题"
 date: 
 tags: 
 categories: 
---

1.webpack配置alias

```
resolve: {
    alias: {
        ...
        '@ant-design/icons/lib/dist$': path.resolve(__dirname, '../src/icons.ts'),
        '@': path.resolve(__dirname, '../src')
    }
}
```

2.在src目录下编写icons.ts,内容是使用到的icon

```
export {default as UserOutline} from '@ant-design/icons/lib/outline/UserOutline';
export {default as CloseCircleFill} from '@ant-design/icons/lib/fill/CloseCircleFill';
export {default as InfoCircleFill} from '@ant-design/icons/lib/fill/InfoCircleFill';
export {default as CheckCircleFill} from '@ant-design/icons/lib/fill/CheckCircleFill';
```

注：1.如果打包错误，请检查alias的配置和icons.ts的路径2.icon.ts的内容，包括自己用到的icon和组件用到的icon