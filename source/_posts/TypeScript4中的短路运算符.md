---
 title: "TypeScript4中的短路运算符"
 date: 2021-01-10
 tags: [TypeScript]
 categories: 
---

```less
a ||= b
// 相当于: a = a || b

a &&= b
// 相当于: a = a && b

a ??= b
// 相当于: a = a ?? b	
// 相当于: a = a !== null && a !== undefined ? a : b


```