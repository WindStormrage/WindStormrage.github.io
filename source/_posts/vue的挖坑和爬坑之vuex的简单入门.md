---
title: vue的挖坑和爬坑之vuex的简单入门
date: 2017-09-29 13:10
tags: [前端, vue, vuex]
---


首先vuex的中文文档https://vuex.vuejs.org/zh-cn/

<!--more-->
## 首先vuex是什么，官方解释是
Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。Vuex 也集成到 Vue 的官方调试工具 devtools extension，提供了诸如零配置的 time-travel 调试、状态快照导入导出等高级调试功能。

## 我的理解是
就是vue组件之间的数据管理

## event bus
对于vue组件之间的数据传递，父子之间的简单的传递还算是简单，然后你要在各个组件之间传递的话就不太方便了，有两种解决方案暂时我只接触到了两种，其中一种就是用event bus,在入口js中定义一个bus（巴士）

new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App },
  data: {
    Bus: new Vue()
  }
});
 

然后你可以放数据放在bus中

 this.$root.Bus.$emit('tab',-1);
 

然后你也可以随时提取

this.$root.Bus.$on('tab',(data)=>{
    console.log(data);
})
 

你可以理解为全局变量，但是由于设置变量的地方比较随意，然后用的多或者是数据量大的话不利于管理。

## vuex
于是这时候vuex出来了。

先介绍一下vuex中几个关键点，这张图介绍了vuex的处理机制。



 


state：既然vuex是用来储存数据的，那么我们的储存地点就是这里。

mutations：对数据的处理都是在这里进行。

actions：专门用来提交mutations的。

getters：获得到state上的数据的。

所以总的来说就是建立一个state，然后调用actions来提交mutations处理state中的数据，最后用getters得到state中的数据。

至于为什么要用actions来提交mutations处理state中的数据，原因是mutation 必须是同步函数，所以通过actions来调用mutations

首先npm install vuex一下，然后在src里新建一个store的文件夹，用来写vuex的文件，里面创建一个index.js，然后在main.js引入你创建的index.js并在new Vue中声明一下你引入的index文件。
```
import store from './store/index'

new Vue({
// 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
然后你可以在index里面写你的vuex文件了

 

import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex);
//储存
const state = {
　　i: 100
};
//处理state
const mutations = {
　　ADD(state) {
　　　　state.i++;
　　}
};
//提交mutations
const actions = {
　　add : function ({commit, state}) {
　　　　commit(ADD)
　　}
}
//获得state
const getters = {
　　getdata : state => state.notes.i
}
// 挂载
export default new Vuex.Store({
　　state,
　　mutations,
　　actions,
　　getters
})
```

 

 

 然后你在组件中可以调用getters获得对应的值

this.$store.getters.getdata
然后你可以在组件中调用actions

this.$store.dispatch('add')
 以上是对vuex的最简单的一个demo的介绍

然后个人看到了几个比较好的简单的项目可以看看

https://github.com/ToWorkit/VUEX

https://github.com/coligo-io/notes-app-vuejs-vuex

还有的是如果你在actions和mutations中要传递值的话可以

//调用actions时传值
store.dispatch('add', {
  data: 10
})
//调用mutations时传值
store.commit('increment', {
   data: 10 
})
最后还有一个module讲一下，如果你的Vuex有两个模块要储存的话你可以通过这种方式储存
```
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```