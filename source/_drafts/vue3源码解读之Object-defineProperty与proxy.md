---
title: vue3源码解读之Object.defineProperty与proxy
tags: []
id: '174'
categories:
  - - 未分类
---

## 关于Object.defineProperty

定义：Object.defineProperty()方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象

### usage

```javascript
const object1 = {};
let getNum=0;
let setNum=0;
Object.defineProperty(object1, 'property1', {
  get: function () {
      getNum++;
      return 8
  },
  set: function () {
      setNum++
  }
});

object1.property1 = 77;

console.log(object1.property1);
// expected output: 8
// expected output: 1 1
```

## 关于Proxy

## Object.defineProperty实现双向绑定的缺点

## proxy实现双向绑定的优点