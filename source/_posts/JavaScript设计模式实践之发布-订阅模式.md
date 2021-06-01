---
title: JavaScript设计模式实践之发布-订阅模式
theme: github
date: 2021-01-03
categories: 
- 文章
---
### 写在前面
最近在读《JavaScript设计模式与开发实践》，恰逢手头上的工作任务是项目重构。本系列文章就是对本次学习实战的记录和总结。
### 概念
发布-订阅模式又叫观察者模式。它定义对象间的一种一对多关系，当一个对象的状态发生变化时，所有依赖于它的对象都将得到通知。

举个现实生活里的例子，比如说你非常喜欢一个博客，但不知道它什么时候会更新，所以你就每天甚至每隔几小时去看一下是否有更新。如果用上发布-订阅模式，就相当于你关注了该博客的公众号，它有更新的话，你会立马收到一个推送消息。

基于此，我们应该很容易联想到dom事件、双向绑定、vuex。确实，在JavaScript语言中，DOM事件、Event对象、Proxy对象等都是发布-订阅模式的典型应用。它们都提供有订阅、发布的能力。

### 应用场景

除了对原生已有实现的使用，书中讲到的“网站登录”和“模块间通信”，都是该模式极佳的应用场景。这些例子很能说明该模式的特点和优势，这里贴一下代码。

当ajax登录成功，需要更新页面数据，我们通常的做法是这样。

```js
login.succ(function(data){
  header.setAvatar( data.avatar); // 设置header 模块的头像
  nav.setAvatar( data.avatar ); // 设置导航模块的头像
  message.refresh(); // 刷新消息列表
  cart.refresh(); // 刷新购物车列表
});
```

很明显，它产生了一个耦合。比如我们现在新增了一个收货地址的模块，也依赖用户信息。不可避免的登录模块就得被迫营业，增加一个调用。当模块较多，且是由不同人编写的时候，登录模块被迫营业的次数会越来越多以至于疲于应付。

我们完全可以用发布-订阅模式彻底解决这个问题。

```js
login.succ(function (data) {
  login.trigger('loginSucc', data) // 发布登录成功的消息
})

var header = (function () { // header 模块
  login.listen('loginSucc', function (data) {
    header.setAvatar(data.avatar);
  });
  return {
    setAvatar: function (data) {
      console.log('设置header 模块的头像');
    }
  }
})();

var nav = (function () { // nav 模块
  login.listen('loginSucc', function (data) {
    nav.setAvatar(data.avatar);
  });
  return {
    setAvatar: function (avatar) {
      console.log('设置nav 模块的头像');
    }
  }
})();

var address = (function () { // nav 模块
  login.listen('loginSucc', function (obj) {
    address.refresh(obj);
  });
  return {
    refresh: function (avatar) {
      console.log('刷新收货地址列表');
    }
  }
})();
```

### 实践

DOM事件相信大家都已经很熟悉了，这里就练一下Vue3.0响应式原理的基石：ES6的Proxy对象。

VUE官方在介绍[什么是响应性](https://www.vue3js.cn/docs/zh/guide/reactivity.html#%E4%BB%80%E4%B9%88%E6%98%AF%E5%93%8D%E5%BA%94%E6%80%A7)的时候举了一个例子：

``` js
let obj = {
  val1: 2,
  val2: 3
}
let sum = obj.val1 + obj.val2
console.log(sum) // 5
// 改变值
obj.val1 = 3
// sum是不变的
console.log(sum) // 5
```

当我们更新第一个值，sum并不会被修改。这……天经地义，理所当然，因为JavaScript通常是”非响应的“。

下面咱们就用Proxy+发布-订阅模式，让它变成响应式的。

1. 首先基于Proxy，封装一个监听值变化的方法

```js
// 通过Proxy监听对象变化
function addObjectListener(data, callback) {
  return new Proxy(data, {
    set(target, prop, value) {
      data[prop] = value
      callback()
      return value
    }
  })
}
```

2. 给obj添加监听

``` js
let obj = {
  val1: 2,
  val2: 3
}
let sum = obj.val1 + obj.val2
console.log(sum) // 5
// 改变值
obj.val1 = 3
// sum结果是不变的
console.log(sum) // 5
// 添加监听（订阅）
obj = addObjectListener(obj, () => {
  sum = obj.val1 + obj.val2
})
// 改变值
obj.val1 = 3
// 可以看到结果变化了
console.log(sum) // 6

obj.val2 = 8
console.log(sum) //11

```

可以看到在添加监听后，我们再次改变值，sum是有跟着改变的。

说回设计模式，这中间addObjectListener就是实例化发布者并添加订阅的过程，set函数触发callback则对应了发布/通知的这么一个动作。