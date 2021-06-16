---
title: JavaScript设计模式实战之策略模式
theme: github
date: 2020-12-29
categories: 
- JavaScript
---
### 概念

策略模式就是把用于实现同一目标的不同算法、规则封装起来，并使它们可以被相互替换的做法。

它提供了对开放封闭原则的完美支持，可以有效避免多重条件判断语句。

像下面这个“根据状态值获取对应文本”的函数，就有策略模式的影子。

``` js
function getStatusText(value){
  return {
    "Y":"已完成",
    "N":"已取消",
    "C":"待确认"
  }[value]:"状态未明"
}
```

### 实践

应用“策略模式”对表单验证的代码进行重构。

1. 新增一个Validate类

``` js
// 设计模式之策略模式--表单验证类
class Validate {
  constructor () {
    this.validateFuncs = []
  }
  add (value, rule, errorMsg) {
    this.validateFuncs.push(() => {
      // 获取对应策略的处理函数
      return this._strategies[rule](value, errorMsg)
    })
  }
  check () {
    const validateFuncs = this.validateFuncs
    let result = true
    for (let i = 0, len = validateFuncs.length; i < len; i++) {
      const errorMsg = (() => validateFuncs[i]())()
      if (errorMsg) {
        this.$toast(errorMsg)
        result = false
        break
      }
    }
    return result
  }
  // 策略
  _strategies = {
    isNotEmpty (value, errorMsg) {
      const trimedValue = typeof value === 'string' ? value.trim() : value
      if (trimedValue === '' || trimedValue === 0) {
        return errorMsg
      }
    },
    isNotLessThan (value, errorMsg) {
      const [num1, num2] = value
      if (num1 < num2) {
        return errorMsg
      }
    }
  }
}
export default Validate
```

2. 使用前代码

``` js
onSubmit() {
  if (!this.payAmount) {
    this.$toast("请输入购买金额")
    return
  }
  if (this.payAmount > this.detail.baseAmt) {
    this.$toast("购买金额不能小于起存金额")
    return
  }
  ...
}
```

3. 使用后

``` js
onSubmit() {
  const validate = new Validate()
  validate.add(this.payAmount, 'isNotEmpty', '请输入购买金额')
  validate.add([this.payAmount, this.detail.baseAmt], 'isNotLessThan', '购买金额不能小于起存金额')
  ...
  if (!validate.check()) {
    return
  }
}
```

它的优点是通过取代大段ifelse语句，提升了代码可读性；通过封装变化（校验规则），提升了代码重用性和扩展性。缺点也有，就是我们在使用前需要了解已有策略，这违反了最少知识原则，但可以接受。