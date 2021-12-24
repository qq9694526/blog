---
title: 入门即巅峰，Vue3.0学习总结之复合式API
theme: github
index_img: /img/1624533928540.jpg
categories: 
- Vue
---
> 写作是对自己思想的开发和研究。

### 前言
先赞后看，已成习惯。大叫好我是奉旨撸码的胖大海。  
本文旨在通俗易懂的描述“复合式API”的概念、语法和作用，作为自己学习的延续和阶段性成果展示。

### 一、什么是复合式API？

复合式API是Vue3.0新增的、相对于”选项式API“而言的，一种新的组件编写形式。语法层面，主要由setup函数和在其内部调用的生命周期钩子构成，使用时一般还会搭配一些响应式API（下期内容）。
* setup函数
  组合式 API 的入口，一般做为组件选项使用。

  ```js
  export default {
    props: {
      name: {
        type: String,
      },
    },
    setup(props,context) {
      // setup 选项是一个接收 props 和 context 的函数
      console.log(props); // { name: '' }
      const defaultName = props.name||'码农胖大海'
      // 这里返回的任何内容可以用于组件的其余部分，模板以及refs获取组件实例后的访问
      return {
        defaultName
      };
    },
  };
  ```

  需要注意的是在 setup 中你应该避免使用 this，因为setup 选项在组件创建之前执行，此时组件实例还没有生产。

* 生命周期钩子
  可以通过直接导入 `onX` 函数来注册生命周期钩子

  ```js
  import { onMounted, onUpdated, onUnmounted } from 'vue'
  
  const MyComponent = {
    setup() {
      onMounted(() => {
        console.log('mounted!')
      })
      onUpdated(() => {
        console.log('updated!')
      })
      onUnmounted(() => {
        console.log('unmounted!')
      })
    }
  }
  ```

  和Vue的生命周期是一致的，除了create，因为setup本身就是这个阶段。

  详情可以参看[**选项式 API 的生命周期选项和组合式 API 之间的映射**](https://v3.cn.vuejs.org/api/composition-api.html#%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E9%92%A9%E5%AD%90)

* `<script setup>`

  
  Vue3.2新增的复合式API新写法，是在单文件组件中使用组合式 API 的编译时语法糖。

  ```vue
  <script setup>
  console.log('hello script setup')
  </script>
  ```

  里面的代码会被编译成组件 `setup()` 函数的内容，在每次组件实例被创建的时候执行。内置有`defineProps` 、 `defineEmits`等方法，以提供和setup()函数中props和context相似的能力。

  其间定义的所有变量都可以直接在模板使用，包括improt导入的组件或方法（官方：顶层的绑定会被暴露给模板）。但是，当需要通过模板 ref 或者 $parent访问该组件实例时，需要使用defineExpose明确要暴露出去的属性。

  ```vue
  <template>
    <div class="c-count-wrap">
      <span class="c-count-num">{{ currentTime }}</span>
    </div>
  </template>
  
  <script setup>
  let currentTime = ref(props.initialValue);
  let timer = null; // 定时器
  
  // ...省略很多代码，具体的参看最后一个例子
  const stop = () => {
    clearInterval(timer);
  };
  
  // 使用defineExpose明确要暴露出去的属性和方法
  defineExpose({
    stop,
  });
  </script>
  ```
  
  相比于普通的 script 语法，它具有更多优势：
  
  > - 更少的样板内容，更简洁的代码。
  > - 能够使用纯 Typescript 声明 props 和抛出事件。
  > - 更好的运行时性能 (其模板会被编译成与其同一作用域的渲染函数，没有任何的中间代理)。
  > - 更好的 IDE 类型推断性能 (减少语言服务器从代码中抽离类型的工作)。

### 二、它的出现是为了解决什么问题？

可以帮助我们**更好的**进行代码逻辑的**聚合**和**复用**。

当一个组件包含功能较多，变得越来越复杂的时候，选项式API的方式有一个弊端。它会导致逻辑关注点分散，继而使得理解和维护组件变得困难。官方文档中的这个大型组件的示例，很好的展示了这点（图中逻辑关注点按颜色进行了分组）。

![Vue 选项式 API: 按选项类型分组的代码](http://7niu.zhaohaipeng.com/62783021-7ce24400-ba89-11e9-9dd3-36f4f6b1fae2.png)

复合式API允许我们将这些分散在data、computed、methods、filters……中的相关逻辑拎出来写在一起，以实现代码逻辑的聚合和复用。类似mixins，但比mixins要灵活，且没有变量覆盖、数据来源不明的问题。


### 三、举个例子

通过一个例子感受下“选项式”和“复合式”的区别。

这里拿很久之前实现过的倒计时组件举例。代码做了简化，但麻雀虽小，五脏俱全，非常适合练手。[Github上有完整代码](https://github.com/qq9694526/vue3-study)

![image-20211211172627033](http://7niu.zhaohaipeng.com/image-20211211172627033.png)

模板部分很简单，由两个颜色的svg图片和中间的数字构成。

```html
<template>
  <div class="c-count-wrap">
    <svg xmlns="http://www.w3.org/200/svg" height="110" width="110">
      <circle cx="55" cy="55" r="50" fill="none" stroke="#ccc" stroke-width="5" stroke-linecap="round"/>
      <circle class="c-count-process" cx="55" cy="55" r="50" fill="none" stroke="#ff9800" stroke-width="5" :stroke-dasharray="`${process},10000`"/>
    </svg>
    <span class="c-count-num">{{ currentTime }}</span>
  </div>
</template>
<style>
.c-count-wrap {
  display: inline-block;
  position: relative;
  font-size: 0;
}
.c-count-wrap .c-count-num {
  position: absolute;
  display: inline-block;
  top: 50%;
  left: 0;
  width: 100%;
  text-align: center;
  transform: translateY(-50%);
  font-size: 14px;
  white-space: nowrap;
}
.c-count-wrap .c-count-process {
  transform-origin: 55px 55px;
  transform: rotate(-90deg);
}
</style>
```

Js部分则是有2个入参、1个计算属性、3个方法和1个事件。

#### 选项式API的实现

```
<script>
export default {
  props: {
    initialValue: {
      type: Number,
      default: 10,
    },
    autoPlay: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      currentTime: 0,
      timer: null,
    };
  },
  computed: {
  	// 环形进度条
    process() {
      const totalTime = this.initialValue;
      const currentPercent = parseFloat(this.currentTime / totalTime).toFixed(
        2
      );
      const circleLength = Math.floor(2 * Math.PI * 50);
      return currentPercent * circleLength;
    },
  },
  created() {
    this.currentTime = this.initialValue;
    if (this.autoPlay) {
      this.start();
    }
  },
  methods: {
    start() {
      clearInterval(this.timer);
      this.timer = setInterval(() => {
        if (this.currentTime <= 0) {
          clearInterval(this.timer);
          // 派发事件-倒计时结束
          this.$emit("turnOver");
          return;
        }
        this.currentTime -= 1;
      }, 1000);
    },
    stop() {
      clearInterval(this.timer);
    },
    reset() {
      this.stop();
      this.currentTime = this.initialValue;
    },
  },
};
</script>
```

#### 复合式API的实现

```js
<script>
import { ref, computed } from 'vue'
export default {
  props: {
    initialValue: {
      type: Number,
      default: 10,
    },
    autoPlay: {
      type: Boolean,
      default: true,
    },
  },
  setup(props,context){
    let currentTime = ref(props.initialValue)
    let timer = null // 定时器

    const start = ()=> {
      clearInterval(timer);
      timer = setInterval(() => {
        if (currentTime.value <= 0) {
          clearInterval(timer);
          // 派发事件
          context.emit("turnOver");
          return;
        }
        currentTime.value -= 1;
      }, 1000);
    }
    const stop = () => {
      clearInterval(timer);
    }
    const reset = ()=> {
      stop()
      currentTime.value = props.initialValue;
    }
    // 环形进度条
    const process = computed(()=>{
      const totalTime = props.initialValue;
      const currentPercent = parseFloat(currentTime.value / totalTime).toFixed(
        2
      );
      const circleLength = Math.floor(2 * Math.PI * 50);
      return currentPercent * circleLength;
    })

    if (props.autoPlay) {
      start();
    }

    return {
      currentTime,
      process,
      start,
      stop,
      reset
    }
  }
};
</script>
```

#### script setup版实现

```js
<script setup>
// 这里定义的所有变量都可以直接在模板使用，包括improt导入的组件或方法官方：顶层的绑定会被暴露给模板）。但是,
// 当需要通过模板 ref 或者 $parent访问该组件实例时，需要使用defineExpose明确要暴露出去的属性
import { ref, computed } from 'vue'

// 使用defineProps声明props
const props = defineProps({
  initialValue: {
    type: Number,
    default: 10,
  },
  autoPlay: {
    type: Boolean,
    default: true,
  },
});

// 使用defineEmits声明emits
const emit = defineEmits(["turnOver"]);

let currentTime = ref(props.initialValue);
let timer = null; // 定时器

const start = () => {
  clearInterval(timer);
  timer = setInterval(() => {
    if (currentTime.value <= 0) {
      clearInterval(timer);
      emit("turnOver");
      return;
    }
    currentTime.value -= 1;
  }, 1000);
};
const stop = () => {
  clearInterval(timer);
};
const reset = () => {
  stop()
  currentTime.value = props.initialValue;
};
// 环形进度条
const process = computed(() => {
  const totalTime = props.initialValue;
  const currentPercent = parseFloat(currentTime.value / totalTime).toFixed(2);
  const circleLength = Math.floor(2 * Math.PI * 50);
  return currentPercent * circleLength;
});

if (props.autoPlay) {
  start();
}

// 使用defineExpose明确要暴露出去的属性
defineExpose({
  start,
  stop,
  reset,
});
</script>
```

### 总结

复合式API是Vue3新增的、相较于“选项式API”而言的，一种新的组件编写形式。用以解决选项式API，在大型复杂组件中存在的逻辑关注点分散问题。它可以帮助我们更好的进行代码聚合和复用。

### 参考资料

1. https://v3.cn.vuejs.org/guide/composition-api-introduction.html