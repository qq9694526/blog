---
title: 一文读懂Vue3.0的改进与优化
theme: github
date: 2021-03-17
categories: 
- 文章
---
### 前言

本文旨在通俗易懂的回答：相较于2x，Vue3.0都做了哪些改进与优化。

分为三个层面：性能、源码和新特性。

### 性能优化

#### 1.引入tree-shaking技术，减少打包体积

众所周知，ES6的Module，当我们引入某个模块，模块是整个加载的，构建时也会被全部打包。即便我们只用了其中1个方法。

```
import { a } from './xxx.js' 
```

我只想收获一缕春风，你却给了我整个春天。

tree-shaking会在编译阶段标记未被引用的函数或对象，在压缩阶段去删除那些标记过的无用代码，从而实现按需打包。

#### 2.数据劫持方案优化，由Object.defineProperty改为Proxy对象

众所周知，Object.defineProperty有局限性。它侦听的是对象上某个属性的变化。

```
Object.defineProperty(data, 'a',{
  get(){
    // track
  },
  set(){
    // trigger
  }
})
```

注意！它侦听的是某个**属性**。如果要侦听对象，是需要通过循环遍历劫持所有属性来实现的。这必然会带来性能负担，且不能侦听对象属性的新增和删除。这就是官方所说的[”由于 JavaScript 的限制，Vue **不能检测**数组和对象的变化。“]([https://cn.vuejs.org/v2/guide/reactivity.html#%E5%A6%82%E4%BD%95%E8%BF%BD%E8%B8%AA%E5%8F%98%E5%8C%96)。大家有没有感受到一股怨念……哈哈😁

造成的直接结果就是，响应式在以下场景是失效的。

* 当添加或移除对象属性时
* 当利用索引直接设置一个数组项时
* 当修改数组的长度时

为此，Vue不得不扩展Array对象并提供额外的$set和\$delete来解决这个问题，简单的背后可“不简单”。

[Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)的出现则像一束光照在了响应式的脑门上，使得Vue有机会弥补”JavaScript 限制“的遗憾。

因为，它劫持的是整个对象。

```
const target = {
  message1: "hello",
  message2: "everyone"
};

const handler = {
  get: function(target, prop, receiver) {
    return "world";
  }
};

const proxy = new Proxy(target, handler);
```

响应式的天命将星有没有？这里给尤大来点背景音乐——爱不释手的……红色高跟鞋^^破音……

#### 3.编译优化

众所周知，Vue2.0中数据更新触发重新渲染的粒度是组件级的。这相较于React会重新渲染整个组件子树来说（不使用`PureComponent` 和 `shouldComponentUpdate`的情况下 ），有着明显的性能提升。但尤大觉得这还不够，他还能更秀。

Vue3.0 在编译阶段设计了 Block 的概念。它会根据是否有响应式插值，把节点区分为动态和静态，然后在patch阶段 只比对并更新 Block 中的动态子节点。从而避免了不必要的静态节点的比对，实现了运行时组件更新的性能优化。

除此之外，编译过程中还增加了AST（抽象语法树）。流程大致为：解析template模板生成AST节点对象，遍历AST并进行词法分析，通过各种转换函数完善虚拟节点的语义和信息，最终生成用于渲染 vnode的render函数。

编译应该是Vue源码中最复杂的一块了，尢大也说过，懂编译原理就可以为所欲为，感兴趣的小朋友，墙裂推荐去看看黄轶老师的《Vue.js 3.0 核心源码解析》。突出一个字：硬核。

### 源码层面

#### 1.更好的源码组织方式

Vue3.0采用了 monorepo 的方式管理项目代码。我们先通过目录结构直观的感受一下……

```
// 2.x
├── src
│   ├── compiler //模板编译相关
│   ├── core //与平台无关的通用运行时代码
│   ├── platforms //平台专有代码
│   ├── server //服务端渲染
│   ├── sfc //.vue单文件解析
│   └── shared //共享工具代码
```

```
// 3.x
├── packages
│   ├── compiler-core
│   ├── compiler-dom
│   ├── compiler-sfc
│   ├── compiler-ssr
│   ├── global.d.ts
│   ├── reactivity
│   ├── runtime-core
│   ├── runtime-dom
│   ├── runtime-test
│   ├── server-renderer
│   ├── shared
│   ├── size-check
│   ├── template-explorer
│   └── vue
```

更细粒度、更明晰的模块划分有没有？其实，monorepo最主要的好处是统一的工作流和代码共享。Vue3.0源码中的一些 package是可以独立运行的，比如 reactivity 响应式库。

#### 2.使用TypeScript开发

没啥好说的，应该是三赢。Vue趁着TypeScript的大势所趋，顺势而上；TypeScript平添一员猛将，帝国版图一日千里；咱们平民老百姓，则省去了维护.ts的烦恼。真的好想一键三连啊……[狗头👏]

本人TypeScript用的少，说不出什么深刻的理解。只能草草的祝TypeScript长命百岁，然后默默记住它的好：

* 显式类型使我们的代码可读性更高、更加的健壮可靠；
* 静态类型检测可以帮助我们避免很多由于类型导致的错误；
* 有利于 IDE 对变量类型的推导，从而提供精准有效的代码提示。

### 新特性

#### 1.复合式API

如果追根溯源的话，还得从"Hello Vue!"说起。

```
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```

```
Hello Vue!
```

[我们已经成功创建了第一个 Vue 应用！所有东西都是**响应式的**！](https://cn.vuejs.org/v2/guide/index.html#%E5%A3%B0%E6%98%8E%E5%BC%8F%E6%B8%B2%E6%9F%93)”

时至今日，仿佛还是能感受到那种力透纸背的骄傲与欣喜。不经历JQ的时代，怕是很难体会到这点。但这并不妨碍，我们对它有着近乎一致的第一印象：简单、直观。

通过配置的方式开发页面，显然更符合人类的直觉 ，这为Vue赢得了“容易上手”的好名声。也是它后来能迅速声名鹤起，继而席卷整个前端世界的重要原因之一。

但……**命运中的一切馈赠早已在暗中标好了价格**，这是有代价的。

当组件包含功能较多，变得越来越复杂的时候，Options API的方式会导致逻辑关注点分散，继而使得理解和维护组件变得困难。官方文档中的这个大型组件的示例，很好的展示了这点。其中逻辑关注点按颜色进行了分组。

![Vue 选项式 API: 按选项类型分组的代码](/img/vue3-api.png)

为了解决这个问题，使我们能够将同一逻辑关注点相关的代码配置在一起，并实现逻辑复用，组合式 API应运而生。

直接上答案，下面是通过组合式 API重构后的代码。

```js
// src/components/UserRepositories.vue
import { toRefs } from 'vue'
import useUserRepositories from '@/composables/useUserRepositories'
import useRepositoryNameSearch from '@/composables/useRepositoryNameSearch'
import useRepositoryFilters from '@/composables/useRepositoryFilters'

export default {
  components: { RepositoriesFilters, RepositoriesSortBy, RepositoriesList },
  props: {
    user: { type: String }
  },
  setup(props) {
    const { user } = toRefs(props)

    const { repositories, getUserRepositories } = useUserRepositories(user)

    const {
      searchQuery,
      repositoriesMatchingSearchQuery
    } = useRepositoryNameSearch(repositories)

    const {
      filters,
      updateFilters,
      filteredRepositories
    } = useRepositoryFilters(repositoriesMatchingSearchQuery)

    return {
      // 因为我们并不关心未经过滤的仓库
      // 我们可以在 `repositories` 名称下暴露过滤后的结果
      repositories: filteredRepositories,
      getUserRepositories,
      searchQuery,
      filters,
      updateFilters
    }
  }
}
```

我们先忽略setup、toRefs这些有些陌生的语法，仅从结构上去看。可以明显看到，它按“逻辑相关”的原则，将部分组件逻辑抽象成可重用块，拆分出了三个composables，然后把其间的功能点在父组件中进行了统一的安装执行。代码结构、运行逻辑清晰明了，逻辑关注点分散的问题一去不复返。

同时，它还完成了组件级以下的逻辑复用。在此之前，当逻辑分散在data、computed、methods、filters…… 我们想要复用，一般是通过mixins来实现的。mixins的缺点很明显，首先我们不能向 mixin 传递任何参数来改变它的逻辑，这降低了它在抽象逻辑方面的灵活性。另外，当组件中存在多个mixins，就存在变量覆盖、数据来源不明的问题。

本节有些长，我们做个总结：**复合式API是一种通过逻辑关注点组织代码的新方法，它可以帮助我们更好的进行代码逻辑的聚合和复用。**

#### 2.Teleport

Teleport是3.0新增的内置组件，它允许我们指定组件元素挂载到哪个DOM节点下。以应对那些 组件中部分元素需要放置在组件外节点甚至Vue app 之外的场景。

下面是一个全屏模态窗的例子

```
// vue
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Open full screen modal! (With teleport!)
    </button>
    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          I'm a teleported modal! 
          (My parent is "body")
          <button @click="modalOpen = false">
            Close
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return { 
      modalOpen: false
    }
  }
})
// css
.modal {
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  background-color: rgba(0,0,0,.5);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.modal div {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: white;
  width: 300px;
  height: 300px;
  padding: 5px;
}
```

其中的.modal节点会被挂载到props to所指定的body中去。

语法简单、功能直接，能说的并不多。只是这组件名 起得可太秀了，Teleport：心灵传输。

#### 3.片段

众所周知，2.0的template必须有且仅有一个根节点。否则程序会报错：

> The template root requires exactly one element

```
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>
```

3.0做了改进，消除了这个限制。官方把 支持多根节点组件的这个特性，称之为片段。

```
<template>
  <header>...</header>
  <main v-bind="$attrs">...</main>
  <footer>...</footer>
</template>
```

这时候一定有同学会问，为什么2.0不行 3.0就行了呢？

还记得咱们前面提到的“Vue2.0中数据更新触发重新渲染的粒度是组件级的”吗？Vue就是通过这个根节点，把diff限定在某个组件内提升了新旧DOM比对效率，从而提升渲染性能的。

3.0的话这个优点还在，只是引入AST后的编译过程，在解析Template模板的最后，始终会创建 AST 根节点 。这个虚拟根节点能起到相同的作用，这样显式的根节点就不再必要了。

### 总结

以上，就是本学渣阅读和思考后给出的答案，充斥着大量主观的个人理解。如有偏颇，欢迎指正。

### 参考资料

1. [Vue3中文文档](https://www.vue3js.cn/docs/zh/guide/migration/introduction.html)

2. [深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)

3. [黄轶老师的《Vue.js 3.0 核心源码解析》](https://t10.lagounews.com/aRF6RyRScd850 )

4. [Monorepo——大型前端项目的代码管理方式](https://segmentfault.com/a/1190000019309820)

