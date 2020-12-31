---
title: 在 uni-app 中使用vuex
date: 2019-12-31 09:06:00
tags: [vue, uni-app, vuex]
---

由于 uni-app 已经内置了 vuex，所以只要正确引入即可。

1、在项目的根目录下，创建一个名为 store 的文件夹然后在该文件夹下创建一个 index.js 的 js 文件

2、在该 js 文件下定义公共的数据以及方法函数，并且把它导出。

```JavaScript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
const store = new Vuex.Store({
    state: {},
    mutations: {},
    actions: {}
})
export default store
```

3、在入口文件即：main.js 挂载 vuex

```JavaScript
import Vue from 'vue'
import App from './App'
//引入vuex
import store from './store'
//把vuex定义成全局组件
Vue.prototype.$store = store
Vue.config.productionTip = false

App.mpType = 'app'

const app = new Vue({
    ...App,
    //挂载
    store
})
app.$mount()
```

4、在单页面里使用 vuex

```JavaScript
export default {
    created () {
        console.log(this.$store)
    }
}
```
