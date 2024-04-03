---
 title: "tailwind使用指南——老项目迁移"
 date: 2021-08-19
 tags: [CSS,前端]
 categories: [前端笔记]
---

tailwind的特点在于灵活,改造老项目也很方便，改造老项目主要有以下几种场景：

inline style
------------

使用class 替换

```diff
- <span style="color: red">*</span>
+ <span class="text-danger">*</span>
```

单样式的class
---------

使用class替换

```diff
- <span class="s-amount">￥{{ totalAmount }} </span>
+ <span class="text-danger">￥{{ totalAmount }} </span>
- .s-amount {
-   color: #fb9716;
- }
```

复杂样式的class
----------

使用@apply

```diff
.tag-status-error {
    @extend .tag-status;
    background: rgba(250, 58, 26, 0.12);
-   color: #fa3a1a;
+   @apply text-danger
}
```

css或scss函数，需要使用变量
-----------------

css的内置函数，或者搭配scss的mixin时

```diff
.btn-filter-picker-confirm {
    width: 375px;
    height: 98px;
-   background: linear-gradient(270deg, #8898ff 0%, #2e42c7 100%);
+   background: linear-gradient(270deg, theme(colors.blue-100) 0%, theme(colors.blue-200) 100%);
    font-size: 32px;
    font-weight: 400;
    line-height: 45px;
  }
.btn {
    width: 126px;
    height: 56px;
    line-height: 56px;
    border: 0;
    right: -33px;
-   @include fix-border-1(#ccc, 12px)
+   @include fix-border-1(theme('textColor[tip]'), 12px);
}
```

第三方组件需要接受tailwind定义的变量时
-----------------------

抽离主题变量为variables.js

```ini
const textColor = { basic: '#333333'}
module.exports = {
  textColor,
}

```

tailwind.config.js导入variables中的数据进行配置

```ini
const styleConfig = require('./src/styles/variables.js')
module.exports = {
    ...
    theme: {
        ...styleConfig
    }
    ...
}
```

项目中可以引入,放入全局状态中，比如vue可以通过provide/inject

```javascript
const styleConfig = require('@/view/styles/variables');
export default {
    name: 'App',
    provide() {
        return {
            colors:styleConfig.colors
            textColor:styleConfig.textColor
        }
    }
}
```

```diff
<van-icon 
    name="warning"
-    color="#2E42C7"
+    :color="textColor.primary" 
    class="little-icon box-36 mr-16" 
 />
 ...
 export default {
+     inject: ['textColor'],
 }
```