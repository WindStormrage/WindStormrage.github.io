---
title: 对vue双向绑定的思考
date: 2018-05-24 20:27
tags: [前端, vue, 双向绑定]
---


### 对于数组
直接更改数组里面的项的值是不会有view响应的，如：
<!--more-->
```
<ul>
  <li v-for="item in test">
    {{ item }}
  </li>
</ul>
<button @click="test()">click</button>
    
export default {
    data () {
        return {
            test:[1,2,3]
        }
    },
    methods:{
        test(){
            this.test[0] -= 1;
        },
    }
}
```
你可以通过以下方法更改数组值来使他响应

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```


### 对于对象
在对象中的数据如果在js中被改变，他就会同时在对应的视图层中改变。

但是这只适用于最开始出现在data里面的，如果是后来添加的项，并不会动态改变。
> 受现代 JavaScript 的限制 (以及废弃 Object.observe)，Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化过程，所以属性必须在 data 对象上存在才能让 Vue 转换它，这样才能让它是响应的。

如：

```
var vm = new Vue({
  data:{
  a:1
  }
})

// `vm.a` 是响应的

vm.b = 2
// `vm.b` 是非响应的
```


虽然说有的地方是非响应的，但是如果其他地方有视图更新的话，那么非响应的地方的视图也会更新

而且如果是后来加在对象上也想使他响应的话可以调用
```
Vue.set(object, key, value)
```
或

```
this.$set(object, key, value)
```

### vue数据渲染过程
![image](https://cn.vuejs.org/images/data.png)


### 到了现在，就只有两个问题需要深究一下

为什么监听不到数组的变化

为什么其他的view变化后数组之间的变化也会渲染在视图上