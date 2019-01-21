## 前言

Vuex是专为Vue.js项目开发的状态管理模式；简单地说就是一种数据仓库管理的方法，主要是为了解决组件的共享数据的依赖和变更的问题；事实上我们需要权衡Vuex带来的效益，有时候我们使用一个简单的store模式就可以满足我们对共享数据的管理了；[https://cn.vuejs.org/v2/guide/state-management.html](https://cn.vuejs.org/v2/guide/state-management.html "Store状态管理")
![Vuex状态](https://vuex.vuejs.org/vuex.png)

### 起步

对于一个Vuex应用，其核心就是数据仓库store，仓库中包含了整个应用中大部分的状态state，store中的状态有两个特点：

- 响应式，从读取角度看，从store中读取的数据都是响应式的，同步更新的；
- 显式提交commit，从写入角度看，要改变store中的状态，唯一的方式就是显式提交变异mutation，这样可以保证每次变更状态都是可追溯的。

一般性的Vuex结构：

![Vuex目录](https://i.imgur.com/bk8viYq.png)

我们一般单独把store放在一个目录下，在整个项目的main.js中import，这样在业务代码中可以直接使用`this.$store.state.xxx`