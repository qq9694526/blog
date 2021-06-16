---
title: JavaScript设计模式实践之命令模式
theme: github
date: 2021-01-06
categories: 
- JavaScript
---
### 概念

命令模式就是将过程式的请求调用封装在command对象的execute方法里的做法。

### 例子
比如说有个“用户点击按钮，刷新菜单目录”的业务逻辑。过程式的调用：

```js
const MenuBar = {
  refresh: function () {
    console.log('刷新菜单目录');
  }
};
const bindClick = () => {
  button.onclick = () => {
    MenuBar.refresh()
  }
}
bindClick()
```

封装成命令对象：

``` js
// 封装命令对象
var RefreshMenuBarCommand = function (receiver) {
  this.receiver = receiver;
};
RefreshMenuBarCommand.prototype.execute = function () {
  this.receiver.refresh();
};
// 安装命令
const bindClick = (command) => {
  button.onclick = () => {
    command.execute();
  }
}
bindClick(new RefreshMenuBarCommand(MenuBar))
```

可以看到它通过传递命令对象，实现了请求发送者和接收者的解耦。但也很明显：简单的事情变复杂了。

其实，在javascript中是不用这么复杂的。因为**命令模式其实是回调函数的一个面向对象的替代品**。

咱们直接用回调函数的方式：

```js
const bindClick = ( callback )=>{
  button.onclick = ()=> {
    callback();
  }
};
bindClick(MenuBar.refresh)
```

这样也是解耦的。

其实，由于callback的先入为主，咱们前端很难体会到命令模式的好处，事实上也是如此的。它封装、解耦、易扩展的作用和优点，通过callback + 闭包或发布-订阅模式 也是能获得的。

总之，命令模式就是**以面向对象之名干回调函数之实**。在JavaScript语言里，咱们可以取其意，但并不一定要用其形。

### 摘抄

设计模式的主题总是把不变的事物和变化的事物分离开来。   --《JavaScript设计模式与开发实践》