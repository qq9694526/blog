---
title: JavaScript设计模式实践之迭代器模式
theme: github
date: 2020-12-31
categories: 
- JavaScript
---
### 写在前面
最近在读《JavaScript设计模式与开发实践》，恰逢手头上的工作任务是项目重构。本系列文章就是对本次学习实战的记录和总结。
### 概念

迭代器模式是指提供一种方法，顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

现代语言一般都内置的有迭代器实现，我们通常称之为遍历函数。这个在咱们js里是比比皆是，forEach、map、filter、find……数不胜数。详情建议查阅我早前写的另一篇文章[JS遍历方法总结](https://juejin.cn/post/6844903736880414734)。

迭代器分为内部迭代器和外部迭代器。咱们用到的大多数像forEach、$.each都是内部迭代器，外部只需要一次调用。而外部迭代器会显式地请求迭代下一个元素。就像Generator函数那样，需要外部调用next方法。

``` js
// 出自《ECMAScript 6 入门》 
const g = function* (x, y) {
  let result = yield x + y;
  return result;
};

const gen = g(1, 2);
gen.next(); // Object {value: 3, done: false}

gen.next(1); // Object {value: 1, done: true}
```

### 实践

感觉JavaScript语言对迭代器的实现已经足够强大。很早以前就已经有用map、filter、find、some、every这些高阶函数消灭掉了for循环。

### 感悟

浅层次的重构靠整洁代码，深层次的重构靠设计模式。

