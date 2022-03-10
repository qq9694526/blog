---
title: 入门即巅峰，Vue3.0学习总结之响应式API
theme: github
index_img: /img/1624533928540.jpg
date: 2021-12-24
categories: 
- Vue
---


> 写作就像开源，即渡人也渡己。 --码农胖大海

### 前言

先赞后看，已成习惯。大家好，我是在编程世界里苟且偷生的大海。

本文旨在通俗易懂的描述“响应式API”的概念、作用和语法，作为自己学习的延续和阶段性成果展示。

### 一、什么是响应性API
1. 在描述概念之前，先来回顾一下，什么是响应性。

   > 响应性是一种允许我们以声明式的方式去适应变化的编程范例。

   适应变化？还是很懵……先来看一个非响应性的例子。

   ``` js
   let val1 = 2
   let val2 = 3
   let sum = val1 + val2
   console.log(sum) // 5
   val1 = 3
   console.log(sum) // 仍然是 5
   ```
   
   我们改变了第一个值，sum的值并没有改变。
   
   相反，如果改变一个值，sum被重新计算和赋值了，那么这就是响应性的。
   
   ```js
   export default{
     data(){
       return {
         val1:2,
         val2:3
       }
     },
     computed:{
       sum(){
         return this.val1 + this.val2
       }
     },
     created(){
       console.log(this.sum) // 5
       this.val1 = 3
       console.log(this.sum) // 6 sum会随着改变
     }
   }
   ```
   
2. Vue中的响应性
  
   上面例子中用到的computed计算属性，就是Vue对响应性的一种实现。
   
   除此之外，响应性在Vue中还通常表现为：改变绑定在模板上的数据，视图会自动更新。这就是 Vue引以为傲的“响应性系统”，也是Web前端由“操作DOM”到“数据驱动”巨大变革的基石。
   
   至于它是如何实现的？这是一个耳熟能详、老生常谈的面试题，“介绍一下Vue的响应式原理”。
   
2. 在回答完什么是响应性后，什么是响应式API也就不言自喻了
   响应式API是Vue3.0新增的，为 JavaScript 对象创建响应式状态/效果的一系列函数/方法的总称。核心语法包括reactive、ref、coumpetd、watch等

### 二、核心语法介绍

#### reactive()

1. 该 API 返回一个响应式的对象状态。使我们可以显式的定义一个响应式状态。
   ``` js
   import { reactive } from 'vue'
   // 响应式状态
   const state = reactive({
     count: 0
   })
   console.log(state) // Proxy {count: 0}
   ```
   把响应式的对象绑定到模板上，数据变化时，视图就会自动更新。
   
   ```
   <template>
     <div>{{ state }}</div>
   </template>
   ```
   
2. 该响应式转换是“深度转换”——它会影响传递对象的所有嵌套 property。

   ```js
   const state = reactive({
     obj: {
     	count1: 0, // 注意，比上面多了一层嵌套
     }
   })
   // state.obj.count1 也是响应式的
   ```
   
3. 我们定义在data选项中的数据，其响应性也是由 `reactive()` 实现的。

   > 当从组件中的 `data()` 返回一个对象时，它在内部交由 `reactive()` 使其成为响应式对象。

3. 基于Proxy跟踪数据变化，避免了 Vue 早期版本中存在的响应性失效问题。
   由于 JavaScript 的限制（Object.defineProperty），Vue **不能检测**数组和对象的变化。响应性在以下场景会失效：

   * 当添加或移除对象属性时
   * 当利用索引直接设置一个数组项时
   * 当修改数组的长度时

   ```js
   var vm = new Vue({
     data: {
       a: 1
     }
   })
   // `vm.a` 现在是响应式的
   
   vm.b = 2
   // `vm.b` 不是响应式的
   ```

   Vue3.0改用ES6的 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 跟踪数据变化，避免了上述问题。如果打印reactive函数的返回，可以明显看到它是一个Proxy对象。
   
   ```js
   const state = reactive({
     count: 0
   })
   console.log(state) // Proxy {count: 0}
   ```
   
   至于，为什么Object.defineProperty的方案有缺陷，以及为什么Proxy能避免这个问题并有着更好的性能，之前在[《一文读懂Vue3的改进与优化》](https://juejin.cn/post/6940559895330717709)中有详细描述，在此就不赘述了。

#### ref()

1. 接受一个内部值并返回一个响应式且可变的 ref 对象。

   ```js
   import { ref } from 'vue'
   let count = ref(1)
   console.log(count); // RefImpl {_shallow: false, dep: undefined, __v_isRef: true, _rawValue: 1, _value: 1}
   ```
   对，它和`reactive()`一样，都能创建响应式对象。区别在于ref可以接受基础类型的值，而reactive不可以。
   
2. 如果接受的值是一个对象，那么它将被 [reactive](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive) 函数处理为深层的响应式对象。
  
   ```js
   const refObj = ref({
     count:'1',
   })
   console.log(refObj.value) // Proxy {count: '1'}
   ```
   
   此时`ref(obj).value` 等效于 `reactive(obj)`
   
   ```js
   const obj = {
     count:'1'
   }
   console.log(ref(obj).value === reactive(obj)) // true
   ```
   
3. 在setup函数中，需要通过.value属性访问和改变它的值

   ```js
   export default {
     setup() {
       let count = ref(1);
   		console.log(count.value); // 1
   		count.value = 2
   		console.log(count.value); // 2
     },
   };
   ```

4. 在模板中访问则不需要，Vue会自动对Ref 进行解包

   > 当 ref 作为渲染上下文 (从 [setup()](https://v3.cn.vuejs.org/guide/composition-api-setup.html) 中返回的对象) 上的 property 返回并可以在模板中被访问时，它将自动浅层次解包内部值。只有访问嵌套的 ref 时需要在模板中添加 `.value`

   ```vue
   <template>
     <div>
       // 不需要.value
       <span>{{ count }}</span>
       <button @click="count ++">Increment count</button>
       // 需要.value
       <button @click="nested.count.value ++">Nested Increment count</button>
     </div>
   </template>
   
   <script>
     import { ref } from 'vue'
     export default {
       setup() {
         const count = ref(0)
         return {
           count,
           nested: {
             count
           }
         }
       }
     }
   </script>
   ```

#### toRefs()

说白了就是，响应式对象版的 解构赋值。它使我们可以像使用 [ES6 解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)那样，来获取我们想要的**响应式对象**的属性，且不失去响应性。

```js
  setup() {
    const state = reactive({
    	foo: 1,
    	bar: 2
  	})
    // 可以在不失去响应性的情况下解构
    const { foo, bar } = toRefs(state)
    return {
      foo,
      bar
    }
  }
```

像这样辅助类的响应式API还有很多，比如使用 `readonly` 防止更改响应式对象、使用`isRef`检查值是否为一个 ref 对象等等，就不一一介绍了。

原因有二，①仅使用reactive和ref就能满足大多数的应用场景，延后学习，并不妨碍我们“入门”和做项目；②这类API加起来有十来个，需要消耗不少的精力和注意力，放在入门阶段去啃，凭空增加入门难度。

凡事有个主次，我们大可以走马观花的过一遍API，大致有个印象，等将来用到了再回来补课。

#### computed()
同选项式API中的computed
```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)
console.log(plusOne.value) // 2
```
#### watch()

同选项式API中的watch

```
import { onMounted, ref, watch } from "vue";
// setup中的watch函数，等效选项式API中的watch

export default {
  setup(props, context) {
    let defaultName = ref("码农胖大海");
    onMounted(() => {
      console.log("mounted!");
      // 更新数据以触发watch
      defaultName.value = "mounted后的大海";
    });
    watch(defaultName, (newValue, oldValue) => {
      console.log("The defaultName is: " + defaultName.value); // The defaultName is: mounted后的大海
      console.log("The defaultName newValue is: " + newValue); // The defaultName newValue is: mounted后的大海
      console.log("The defaultName oldValue is: " + oldValue); // The defaultName oldValue is: 码农胖大海
    });
    return {
      defaultName,
    };
  },
};
```

### 三、总结

响应式API可以看做是复合式API的延续，他们是一体的。只是Vue从功能作用上把他们分为了两大类。

这是《入门即巅峰，Vue3.0学习总结》的第三篇。

1. [入门即巅峰，一文读懂Vue3.0的改进与优化](https://juejin.cn/post/6940559895330717709)  

2. [入门即巅峰，Vue3.0学习总结之复合式API](https://juejin.cn/post/7041843727958507528)