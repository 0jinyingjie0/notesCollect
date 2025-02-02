# Vuex 入门

## 介绍

### 组件化开发

在现代 Web 开发复杂多变的需求驱动之下，组件化开发已然成为了事实上的标准。

![Component Tree](./assets/components.png)

组件化开发为我们带来了：

- 更快的开发效率
- 更好的可维护性

### 组件内的状态管理流程

每个组件都有自己的状态、视图和行为等组成部分。

```js
new Vue({
  // state
  data() {
    return {
      count: 0
    };
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment() {
      this.count++;
    }
  }
});
```

- **state**，驱动应用的数据源；
- **view**，以声明方式将 **state** 映射到视图；
- **actions**，响应在 **view** 上的用户输入导致的状态变化。

<img src="./assets/flow-1567760103255.png" width="450">

### 组件通信

然而大多数场景下的组件都并不是独立存在的，而是相互协作共同构成了一个复杂的业务功能。在 Vue 中为不同的组件关系提供了不同的通信规则。

![891636a0-af23-11e7-b111-4d6e630f480d.png](./assets/891636a0-af23-11e7-b111-4d6e630f480d.png)

- 父子关系
- 非父子
- 。。。

#### 父传子：Props Down

```html
<blog-post title="My journey with Vue"></blog-post>
```

```js
Vue.component("blog-post", {
  props: ["title"],
  template: "<h3>{{ title }}</h3>"
});
```

#### 子传父：Events Up

在子组件中使用 `$emit` 发布一个自定义事件：

```html
<button v-on:click="$emit('enlarge-text')">
  Enlarge text
</button>
```

在使用这个组件的时候，使用 `v-on` 监听这个自定义事件

```html
<blog-post ... v-on:enlarge-text="postFontSize += 0.1"></blog-post>
```

#### 父亲直接访问孩子：通过 ref 获取子组件

ref 有两个作用：

- 如果你把它作用到普通 HTML 标签上，则获取到的是 DOM
- 如果你把它作用到组件标签上，则获取到的是组件实例

在使用组件的时候，添加 ref 属性：

```html
<blog-post title="My journey with Vue" ref="post"></blog-post>
```

然后使用 `$refs` 访问：

```js
this.$refs.post;
```

> 使用建议：不在玩不得以的情况下，不要通过这种方式修改数据，如果滥用这种方式会导致数据管理的混乱。

#### 非父子组件通信：Event Bus

我们可以使用一个非常简单的 Event Bus 来解决这个问题：

`event-bus.js`:

```js
export default new Vue();
```

然后在需要通信的两端：

使用 `$on` 订阅：

```js
// 没有参数
bus.$on("自定义事件名称", () => {
  // 执行操作
});

// 有参数
bus.$on("自定义事件名称", data => {
  // 执行操作
});
```

使用 `$emit` 发布：

```js
// 没有自定义传参
bus.$emit("自定义事件名称");

// 有自定义传参
bus.$emit("自定义事件名称", 数据);
```

### 多个组件状态共享

但是，当我们的应用遇到**多个组件共享状态**时：

- 多个视图依赖于同一状态
- 来自不同视图的行为需要变更同一状态

> 最典型的场景就是购物车

对于问题一，传参的方法对于多层嵌套的组件将会非常繁琐，并且对于兄弟组件间的状态传递无能为力。

对于问题二，我们经常会采用父子组件直接引用或者通过事件来变更和同步状态的多份拷贝。以上的这些模式非常脆弱，通常会导致无法维护的代码。

因此，我们为什么不把组件的共享状态抽取出来，以一个全局单例模式管理呢？在这种模式下，我们的组件树构成了一个巨大的“视图”，不管在树的哪个位置，任何组件都能获取状态或者触发行为！

![image-20191128171659282](assets/image-20191128171659282.png)

通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。

### 什么是 Vuex

> 官方文档：Vuex 是一个专为 Vue.js 应用程序开发的**状态管理模式**。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 [devtools extension](https://github.com/vuejs/vue-devtools)，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

- Vuex 是专门为 Vue.js 设计的状态管理库
- 它采用集中式的方式存储需要共享的数据
- 从使用角度，它就是一个 JavaScript 库
- 它的作用是进行状态管理，解决复杂组件通信，数据共享

### 什么情况下使用 Vuex

> 官方文档：
>
> Vuex 可以帮助我们管理共享状态，并附带了更多的概念和框架。这需要对短期和长期效益进行权衡。
>
> 如果您不打算开发大型单页应用，使用 Vuex 可能是繁琐冗余的。确实是如此——如果您的应用够简单，您最好不要使用 Vuex。一个简单的 [store 模式](https://cn.vuejs.org/v2/guide/state-management.html#简单状态管理起步使用)就足够您所需了。但是，如果您需要构建一个中大型单页应用，您很可能会考虑如何更好地在组件外部管理状态，Vuex 将会成为自然而然的选择。引用 Redux 的作者 Dan Abramov 的话说就是：Flux 架构就像眼镜：您自会知道什么时候需要它。

当你的应用中具有以下需求场景的时候：

- 多个视图依赖于同一状态
- 来自不同视图的行为需要变更同一状态

建议符合这种场景的业务使用 Vuex 来进行数据管理，例如非常典型的场景：购物车。

**注意：Vuex 不要滥用，不符合以上需求的业务不要使用，反而会让你的应用变得更麻烦。**

## 示例准备

![1566178446873](./assets/1566178446873.png)

以前的思路：

- 在每个组件中都生成一份儿数据
  - num: 0
- 点击 Foo 组件 + 的时候
  - 让自己内部的 num++
  - 利用 eventBus 发布一个自定义事件：数字+
- 在 Bar 组价中
  - 订阅 数字+ 事件，当事件发生，让自己内部的数字 +1

太麻烦了！

1、使用 VueCLI 创建一个练习项目：

```bash
vue create vuex-demo

cd vuex-demo

npm run serve
```

2、准备两个组件

Foo.vue

```html
<template>
  <div class="com">
    <h2>Foo 组件</h2>
    <p>5</p>
    <button>+</button>
  </div>
</template>

<script>
  export default {};
</script>

<style></style>
```

Bar.vue

```html
<template>
  <div class="com">
    <h2>Bar 组件</h2>
    <p>5</p>
    <button>+</button>
  </div>
</template>

<script>
  export default {};
</script>

<style></style>
```

3、最后在 App.vue 中加载使用 Foo 和 Bar

```html
<template>
  <div id="app">
    <foo />
    <bar />
  </div>
</template>

<script>
  import Foo from "./components/Foo";
  import Bar from "./components/Bar";

  export default {
    name: "App",
    components: {
      Foo,
      Bar
    }
  };
</script>

<style>
  .com {
    border: 1px solid #000;
    padding: 10px;
    margin-bottom: 20px;
  }
</style>
```

最终预览结果如下：

![1570759389550](assets/1570759389550.png)

## 安装 Vuex

官方文档：https://vuex.vuejs.org/zh/installation.html

最方便的方式就是使用 npm：

```bash
# 或者 yarn add vuex
npm install vuex
```

## 配置 Vuex

1、在项目中新建 `store/index.js`：

```js
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

/**
 * 创建一个 Vuex 容器实例，用来在组件的外部管理共享的数据状态
 */
const store = new Vuex.Store({
  /**
   * 类似于组件中的 data
   */
  state: {
    count: 0
  }
});

export default store;
```

2、在 `main.js` 将 store 注册到 Vue 根实例

```js
import Vue from "vue";
import App from "./App.vue";
import store from "./store/";

Vue.config.productionTip = false;

new Vue({
  render: h => h(App),
  store
}).$mount("#app");
```



## State

容器中的 state 就好比组件的 data，用来存储共享数据：

- 容器中的数据是共享的，任何组件都可以访问
- 容器中的数据也是响应式的，数据改变也会驱动视图更新

在组件中访问容器中的数据，可以通过以下几种方式。

在组件模板中直接通过 `$store` 访问容器数据：

```html
<div>
  <p>{{ $store.state.count }}</p>
</div>
```

在组件 JavaScript 中访问容器数据需要加 `this`：

```js
methods: {
  onClick () {
    console.log(this.$store.state.count)
  }
}
```

如果在组件中多次使用到容器数据，可以将其封装到一个计算属性中：

```js
computed: {
  count () {
    return this.$store.state.count
  }
}
```

然后就像使用访问自己的数据一样来访问容器中的数据

```
<div>
  <p>{{ count }}</p>
</div>
```

## 在调试工具中调试 Vuex 容器数据

也可以通过调试工具查看 Vuex 容器中的数据：

![1566179002700](./assets/1566179002700.png)

## Mutation

更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment(state) {
      // 变更状态
      state.count++;
    }
  }
});
```

然后在组件中调用 mutation：

```html
<template>
  <div class="com">
    <h2>Bar 组件</h2>
    <p>{{ $store.state.count }}</p>
    <!-- 如果是在 JS 中，则 this.$store.commit('increment') -->
    <button @click="$store.commit('increment')">-</button>
  </div>
</template>
```

mutation 函数也可以自定义传参：

```js
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

```js
store.commit("increment", 10);
```

mutation 只能传递 1 个自定义参数：

```js
// ...
mutations: {
  increment (state, n, m) {
    console.log(n, m) // 10 undefined
    state.count += n
  }
}
```

```js
store.commit("increment", 10, 20);
```

如果需要传递多个参数，就放到一个对象中：

```js
// ...
mutations: {
  increment (state, payload) {
    console.log(payload) // { n: 10, m: 20 }
    state.count += payload.n
  }
}
```

```js
store.commit("increment", {
  n: 10,
  m: 20
});
```

提交 mutation 的另一种方式是直接使用包含 `type` 属性的对象：

```js
store.commit({
  type: "increment",
  amount: 10
});
```

当使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，因此处理函数保持不变：

```js
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

总结：

- **修改容器的 state 务必使用 mutation 函数**，因为只有 mutation 函数中修改 state 数据才会和调试工具正常工作。
- mutation 函数的第 1 个参数是 state 对象，和你起啥名字没关系
- mutation 函数也可以自定义传参
  - 只能传递1个参数
  - 如果需要传递多个参数就放到一个对象中
- **不要在 mutation 中执行异步操作修改 state**

## 不要在 Mutation 中执行异步操作修改 state

调试工具会出现问题。

## Action

> 目标：
>
> - 了解 Action 的作用是什么
> - 掌握如何定义 Action
> - 掌握如何调用 Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态。
- Action 可以包含任意异步操作。

### 定义 Action

让我们来注册一个简单的 action：

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++;
    }
  },
  actions: {
    increment(context) {
      // 执行异步操作
      setTimeout(() => {
        // 提交 mutation 更新 state
        context.commit("increment");
      }, 1000);
    }
  }
});
```

Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 `context.commit` 提交一个 mutation。

Action 通过 `store.dispatch` 方法触发：

```js
store.dispatch("increment");
```

例如我在组件中：

```html
<button @click="$store.dispatch('increment')">分发 Action</button>
```

Actions 和 Mutation 一样，也支持自定义传参：

```html
<!-- 只有一个参数 -->
<button @click="$store.dispatch('increment', 10)">分发 Action</button>

<!-- 多个参数放到一个对象中 -->
<button @click="$store.dispatch('increment', { n: 10, m: 20 })">
  分发 Action
</button>
```

总结：

- action 中异步操作结束以后，提交 mutation 来修改 state
- 注意：也不要在 action 中直接修改 state，调试工具不工作，永远通过 mutation 修改
- action 函数的第 1 个参数是容器对象
- action 也可以像 mutation 函数一样自定义传参

## Vuex 状态管理流程

> 目标：
>
> - 了解 Vuex 的状态管理流程

### 同步操作

![1566202914280](./assets/1566202914280.png)

- 在组件中 commit 调用 mutation
- 在 mutation 中修改 state 状态
  - 只有 mutation 中修改 state 才会反应到调试工具中
  - **注意：不要在 mutation 中执行异步操作修改 state**
- state 状态发生改变，视图更新

### 异步操作

![1566203230907](./assets/1566203230907.png)

- 在组件中使用 dispatch 调用 action 函数
- 在 action 函数中执行异步操作
- action 函数中异步操作执行结束，提交 mutation
  - **注意：也不要在 action 中直接修改 state，调试工具工作有问题**
- 在 mutation 中修改 state
  - **注意：也不要在 mutation 中执行异步操作修改 state，调试工具工作有问题**
  - 只有 mutation 中修改 state 才能反应到调试工具中
- state 数据发生改变，视图更新
