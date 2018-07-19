## 响应式原理

@(VUE)

[TOC]

### 1. 追踪流程 

![data](https://cn.vuejs.org/images/data.png) 

按照官方文档，Vue实例的data选项，会通过`Object.defineProperty`将所有定义的属性转为setter/getter，并且添加watcher实例，使其在改变值时，达到监听并通知组件刷新的目的。其通知是采用异步方式来完成的`Promise`。

### 2. 使用注意

`VUE无法检测对象属性的添加或者删除`，所以要想使用VUE的响应式编程，必须要在data对象上进行该对象的初始化。