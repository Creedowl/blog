---
title: Vue.js 从入坑到入土
date: 2019-04-23T21:25:25+08:00
draft: false
toc: true
tags:
 - vue
 - 学习笔记
---
# Vue.js 从入坑到入土

近年来前端领域发展飞速，从最早的直接写html, js, css 到使用jQuery快速开发，再到近年来使用Vuejs，Reactjs 实现工程化开发，也就短短十几年。虽然前端项目结构越来越复杂，但也越来越规范，清晰，更加工程化。当前较为流行的前端框架有Vuejs、Reactjs、Angularjs 等，而Vuejs由于其灵(zhong)活(wen)轻(wen)便(dang)，较为流行。虽然Vuejs有完善的中文文档，但文档主要在介绍其用法，缺少如何实现一个完整项目的指导，本文就介绍如何用Vuejs创建一个完整的前端项目，并与后端进行交互。

**本文基于[Vuejs@2.x](mailto:Vuejs@2.x)及[VueCLI@3.x](mailto:VueCLI@3.x)**

1. 所需技术栈：html, javascript, css基础知识，其中js建议了解**ES2015**标准，特别要掌握**promise**的使用
2. 使用的工具：~~宇宙第一编辑器~~ VSCode —— 现在对vue项目的支持已经很好了，可以自动安装vue项目需要的插件

# 0x0 安装

> 推荐使用[yarn](https://yarnpkg.com/lang/zh-hans)代替npm作为包管理器

## 安装VueCLI

> VueCLI是Vuejs官方推出的项目脚手架工具，可以方便的创建、管理Vue项目，还提供优雅的web管理界面

全局安装[VueCLI](https://cli.vuejs.org/zh/)

```bash
yarn global add @vue/cli
```

## 安装Vue-devtools

> [Vue-devtools](https://github.com/vuejs/vue-devtools) 是vue官方的浏览器开发插件，可以方便的在浏览器中调试vue应用

Vue-devtools 提供多平台的浏览器插件可以直接安装使用

[installation](https://github.com/vuejs/vue-devtools#installation)

# 0x1 新建项目

使用 `vue create vue_demo` 创建新项目，接下来会出现一个交互式命令行帮你配置项目。

**注意此处不要使用默认的配置！具体配置参考下图**

> 坑x1 默认的配置十分简单，新手经常不知道怎么写多页面，弄不明白项目怎么组织的，所以需要自定义配置，添加vue-router插件实现多页面跳转

[![asciicast](https://asciinema.org/a/240155.svg)](https://asciinema.org/a/240155)

# 0x2 项目框架

此时项目已经创建成功，如果你按照提示启动项目就能看到经典的vue启动页

![image](http://ww1.sinaimg.cn/large/0071ouepgy1g1yg2r5ysxj30up0mx3zv.jpg)

当前的项目结构如下

```text
.                     项目根目录
├── README.md 
├── babel.config.js
├── package.json      项目主配置文件
├── public            公共资源文件夹
│   ├── favicon.ico
│   └── index.html    入口html
├── src               源码根目录
│   ├── App.vue       入口文件
│   ├── assets        资源文件夹
│   │   └── logo.png
│   ├── components    组件文件夹
│   │   └── HelloWorld.vue 
│   ├── main.js       入口配置文件
│   ├── router.js     路由文件
│   ├── store.js      状态管理
│   └── views         页面文件夹
│       ├── About.vue
│       └── Home.vue
└── yarn.lock         依赖文件
```

在下面的章节具体介绍各个文件、文件夹的用处和如何组织项目。

# 0x3 项目结构

用vue开发的项目一般是[SPA(单页应用)](https://zh.wikipedia.org/wiki/单页应用)，通过动态重写当前页面来与用户交互，而不会重新加载整个新页面。所以要组织多页面就要通过vue-router来组织页面，而vuex用来保存状态。

作为单页应用，当然需要一个入口html文件，也就是 `public/index.html` :

```html
public/index.html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>vue_demo</title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but vue_demo doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```

这是默认生成的 `index.html` ，而编译好的文件（网页内容）会被自动注入到 `` 中，也就意味着要使用vue开发的应用，浏览器必须支持js。

而#app所绑定的vue实例，则位于 `src/main.js`

------

```js
src/main.js

import Vue from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'
Vue.config.productionTip = false
new Vue({
  router, // 使用vue-router插件
  store, // 使用vuex插件
  render: h => h(App) // 渲染页面
}).$mount('#app') // 绑定到 index.html -> div#app
```

其中渲染的页面位于 `src/App.vue`

------

> xxx.vue 是vue中的[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)，将 `html css javascript` 集合在同一个文件，便于组织管理

```html
src/App.vue

<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>
      <router-link to="/about">About</router-link>
    </div>
    <router-view/>
  </div>
</template>
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}
#nav a {
  font-weight: bold;
  color: #2c3e50;
}
#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
```

其中 `` 是页面的 `html` 元素，`style` 是样式元素，还可以添加一个 `javascript` 为js元素。其中 `` 是路由组件，会被自动渲染成当前路由对应的页面

------

```js
src/router.js

import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'
Vue.use(Router)
export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes: [
    {
      path: '/',
      name: 'home',
      component: Home
    },
    {
      path: '/about',
      name: 'about',
      // route level code-splitting
      // this generates a separate chunk (about.[hash].js) for this route
      // which is lazy-loaded when the route is visited.
      component: () => import(/* webpackChunkName: "about" */ './views/About.vue')
    }
  ]
})
```

如 `http://localhost:8080/` 下 `` 被渲染成 `home` 页面，`http://localhost:8080/about` 下 `` 被渲染成 `about` 页面。

> 技巧 这里加载组件使用函数加载，就可以实现懒加载以优化性能。

------

`src/views/` 则是保存页面文件，例如

```html
src/views/Home.vue

<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>
<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
export default {
  name: 'home',
  components: {
    HelloWorld
  }
}
</script>
```

这就是路由 `home` 对应的页面，其中用到了 `HelloWorld` 组件，需要在js中引入。

------

`src/components/` 里面保存可复用的组件，例如上面的 `` 就是一个独立的组件（component）

```html
src/components/HelloWorld.vue

<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <p>
      For a guide and recipes on how to configure / customize this project,<br>
      check out the
      <a href="https://cli.vuejs.org" target="_blank" rel="noopener">vue-cli documentation</a>.
    </p>
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1 {
  margin: 40px 0 0;
}
</style>
```

这就是一个典型的组件，结构就是一个正常的单文本组件，说明组件就是把一部分页面逻辑抽出来组成一个文件。这样的好处是分离与页面无关的组件，使页面逻辑更加清晰，同时也便于复用组件。如果有以下情况，我建议把部分代码分离成独立的组件：

1. 需要多次复用的组件，如基础按钮、表单等
2. 较为复杂但与页面逻辑无关的，如页面的Header、Footer等

> 技巧 在 `style` 表情上添加 `scoped` 属性可以限制CSS生效范围为该组件，不会影响其他元素，在编译时vue解释器会给这些css属性加上随机的hash保证唯一

------

vue还提供了一个状态管理模式 [vuex](https://vuex.vuejs.org/zh/)，可以很方便的管理所有组件的状态。对应文件 `src/store.js`

```js
src/store.js

import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
export default new Vuex.Store({
  state: {
  },
  mutations: {
  },
  actions: {
  }
})
```

~~由于我还没用过 `vuex` 所以自己看文档吧~~

对于简单的应用也可以不用 `vuex` ，一个简单的 [`store` 模式](https://cn.vuejs.org/v2/guide/state-management.html#简单状态管理起步使用) 就足够了

# 0x4 开发指引

有了上面的基础，我们就可以开始写代码了，这里以添加一个 `Foo` 页面展示从远端获得的数据为例

## 1. 给页面添加一个 `Tab`

`Tab` 属于与页面逻辑无关，但又处处要使用的元素，所以我们将其独立成为一个组件

```html
src/components/Tab.vue

<template>
  <div id="tab">
    <div class="item" @click="router('/')">
      <span>home</span>
    </div>
    <div class="item" @click="router('/about')">
      <span>about</span>
    </div>
    <div class="item" @click="router('/foo')">
      <span>foo</span>
    </div>
  </div>
</template>
<script>
export default {
  name: 'Tab',
  methods: {
    router (url){
      this.$router.push(url)
    }
  }
}
</script>
<style scoped>
#tab {
  display: flex;
  justify-content: space-around;
  align-items: center;
  bottom: 0px;
  position: fixed;
  height: 65px;
  left: 0px;
  width: 100%;
  background-color: aquamarine;
}
.item {
  padding-top: 5px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 50px;
}
</style>
```

在这里我们创建了一个底部导航栏，有三个 `Tab` ，并且对应跳转到不同的页面

## 2. 创建 `Foo.vue` 相关逻辑并与后端交互

因为我们需要从远端API获取数据并进行展示，就涉及网络请求。在vue中，官方推荐使用 [axios](https://github.com/axios/axios) 发起请求。`axios` 是一个基于 `promise` 的异步网络请求库，相关用法可参考文档或网上教程

安装 `axios`

```bash
yarn add axios
```

接着在 `src/main.js` 中加入以下语句启用 `axios`

```js
import Axios from 'axios'
// 把axios添加到原型链
Vue.prototype.$axios = Axios
```

现在就可以在vue实例中用 `this.$router` 来访问路由了

------

接着来创建 `Foo.vue`

```vue
src/views/Foo.vue

<template>
  <div id="foo">
    <p>{{ result }}</p>
    <button @click="bar()">bar</button>
  </div>
</template>
<script>
export default {
  name: 'Foo',
  data(){
    return {
      result : 'naive'
    }
  },
  methods: {
    bar(){
      /* eslint-disable */
      this.result = 'loading...'
      this.$axios.post('http://httpbin.org/post')
        .then(res => {this.result = res})
        .catch(res => {console.error(res)})
    }
  }
}
</script>
<style>
</style>
```

~~具体实现代码里面很清楚了~~ 这个文件的作用就是请求 `http://httpbin.org/get` 并把结果显示在界面上

**注意前后端分离的项目会有[跨域(CORS)问题](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)，这个问题比较复杂，巨坑，网上也有较多解决的文章~~咕咕咕~~**

# 0x5 技巧及坑们

这里介绍一些我写vue项目遇到的坑和使用的技巧

## data methods computed props 的区别

~~TODO~~

vue中存储数据有几种方式——data、

# 0x6 结语

到此，一个简单的vue项目就完成了，对于如何组织项目，前后端如何交互应该已经有了初步了解。本文也会持续更新，欢迎关注我的[blog](https://blog.creedowl.com/)

本文的实例请访问[github](https://github.com/creedowl/vue_demo/)