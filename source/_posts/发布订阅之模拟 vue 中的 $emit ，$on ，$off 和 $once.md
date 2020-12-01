---
title: 发布订阅之模拟 vue 中的 $emit ，$on ，$off 和 $once
date: 2020-11-30 21:19:45
tags: [ nodejs, 设计模式]
categories: 前端
copyright: true
author: Jerry Liu
top:
description: 设计模式之订阅发布模式，手动实现模拟 vue 中的 $emit, $on, $off 和 $once, 发布事件，订阅，取消订阅，只订阅一次事件。
---

## 模拟发布订阅模式中 vue 的 $on 和 $emit 事件

&emsp;在 vue 中的 $on 对一个事件进行注册监听，当使用 $emit 对事件进行推送时，所有监听注册该事件的都会接收到 \$emit 推送的数据。

&emsp;现在模拟一个简单的 emit 和 on 实现订阅 发布功能。

&emsp;使用一个 emiEventList 对象来缓存注册的事件及回调函数，当我们使用 on 对事件注册时，on 注册的事件和回调函数会缓存在 emiEventList 对象里面， 当我们使用 emit 事件对我们注册的事件发布数据时，然后循环调用缓存在 emiEventList 对象里面注册的回调函数实现推送的功能。

```javascript
class Message {
  constructor() {
    this.emiEventList = {}; // 缓存调用 on 事件进行注册的回调函数
  }

  emit(eventName = "event", value) {
    // 遍历调用 emiEventList
    if (this.emiEventList[eventName]) {
      for (const callback of this.emiEventList[eventName]) {
        callback.call(this, value);
      }
    }
  }

  on(eventName = "event", callback) {
    if (eventName in this.emiEventList) {
      this.emiEventList[eventName].push(callback);
    } else {
      this.emiEventList[eventName] = [callback];
    }
  }
}

const msg = new Message(); // 创建一个 Message 对象实例

// 注册 dan 和 shuang 两个事件
msg.on("dan", (value) => {
  console.log(value, "dan"); // 每隔2s依次打印 1dan 3dan 5dan ...
});

msg.on("shuang", (value) => {
  console.log(value, "shuang"); // 每隔2s依次推送 0shuang 2shuang 4shuang ...
});

let count = 0;
setInterval(() => {
  if (count % 2 === 0) {
    // 当 count 为双数时推送给 shuang 事件
    msg.emit("shuang", count); // 每隔2s依次推送 0 2 4 ...
  } else {
    // 当 count 为双数时推送给 dan 事件
    msg.emit("dan", count); // 每隔2s依次推送 1 3 5 ...
  }
  count++;
}, 1000);

/* 
输出：
  0shuang
  1dan
  2shuang
  3dan
  4shaung
  .
  .
  .
*/
```

## 模拟发布订阅模式中 vue 的 \$off 事件

&emsp;在 vue 中可以使用 \$off 注销已经注册的事件。

&emsp;当实例调用 off 注销事件时，我们根据注销的事件名去删除或清空 emiEventList 中缓存的回调函数来取消对事件的监听。

```javascript
class Message {
  constructor() {
    this.emiEventList = {};
  }

  emit(eventName = "event", value) {
    // 遍历调用 emiEventList
    if (this.emiEventList[eventName]) {
      for (const callback of this.emiEventList[eventName]) {
        callback.call(this, value);
      }
    }
  }

  on(eventName = "event", callback) {
    if (eventName in this.emiEventList) {
      this.emiEventList[eventName].push(callback);
    } else {
      this.emiEventList[eventName] = [callback];
    }
  }

  off(eventName = "event", callback) {
    // 注销 emit 注册事件
    this.regEvent(this.emiEventList, eventName, callback);
  }

  // 注销注册事件
  regEvent(eventList, eventName, callback) {
    const name = eventList[eventName];
    // 回调函数不存在清空所有
    if (name && !callback) {
      name.length = 0;
    }
    if (name && name.includes(callback)) {
      name.splice(name.indexOf(callback), 1);
    }
  }
}

const msg = new Message(); // 创建一个 Message 对象实例

// 注册两个事件
const event = (value) => {
  console.log(value, "event");
};
// 注册具名事件
msg.on("event", event);

// 注册匿名事件
msg.on("event", (value) => {
  console.log(value);
});

let count = 0;
setInterval(() => {
  msg.emit("event", count);
  if (count === 2) {
    // 注销一个具名事件
    msg.off("event", event);
  }
  if (count === 4) {
    // 注销所有事件
    msg.off("event");
  }
  count++;
}, 1000);

/* 
输出：
  0 event
  0
  1 event
  1
  2 event
  2
  3
  4

*/
```

## 模拟发布订阅模式中 vue 的 \$once 事件

&emsp;在 vue 中可以使用 \$once 注册一个只触发一次的事件。

&emsp;当我们使用 once 对事件进行注册时，我们把 once 注册的事件放入 onceEventList 对象中（ onceEventList 对象专门储存 once 注册的回调函数），当使用 emit 第一次对事件发布数据时，依次循环调用 onceEventList 缓存的事件，然后在注销相应的事件。

```javascript
class Message {
  constructor() {
    this.emiEventList = {};
    this.onceEventList = {};
  }

  emit(eventName = "event", value) {
    // 遍历调用 emiEventList
    if (this.emiEventList[eventName]) {
      for (const callback of this.emiEventList[eventName]) {
        callback.call(this, value);
      }
    }
    // 遍历调用 onceEventList 并注销函数
    if (this.onceEventList[eventName]) {
      for (const callback of this.onceEventList[eventName]) {
        callback.call(this, value);
        this.regEvent(this.onceEventList, eventName, callback);
      }
    }
  }

  on(eventName = "event", callback) {
    if (eventName in this.emiEventList) {
      this.emiEventList[eventName].push(callback);
    } else {
      this.emiEventList[eventName] = [callback];
    }
  }

  // 注册只执行一次的事件
  once(eventName = "event", callback) {
    if (eventName in this.onceEventList) {
      this.onceEventList[eventName].push(callback);
    } else {
      this.onceEventList[eventName] = [callback];
    }
  }

  off(eventName = "event", callback) {
    // 注销 emit 注册事件
    this.regEvent(this.emiEventList, eventName, callback);
    // 注销 once 注册事件
    this.regEvent(this.onceEventList, eventName, callback);
  }

  // 注销注册事件
  regEvent(eventList, eventName, callback) {
    const name = eventList[eventName];
    // 回调函数不存在清空所有
    if (name && !callback) {
      name.length = 0;
    }
    if (name && name.includes(callback)) {
      name.splice(name.indexOf(callback), 1);
    }
  }
}

const msg = new Message(); // 创建一个 Message 对象实例

// 注册两个事件，一个是 once 事件，一个是普通事件
msg.once("onceEvent", (value) => {
  console.log(value, " once"); // 只执行一次
});

msg.on("onceEvent", (value) => {
  console.log(value, " any"); // 正常执行
});

let count = 0;
setInterval(() => {
  msg.emit("onceEvent", count);
  count++;
}, 1000);

/* 
输出：
  0  any
  0  once
  1  any
  2  any
  3  any
  .
  .
  .
*/
```
