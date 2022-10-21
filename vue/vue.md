## Vue中遇到的问题

**`this.$nextTick`用法**

用来渲染dom元素改变之后的改变值。

```html
<template>
    <button ref="tar" 
	    type="button" 
	    name="button" 
	    @click="testClick">{{content}}</button>
</template>
 
<script>
    export default {
        data () {
            return {
                content: '初始值'
            }
        }
　　　　 methods: {
　　　　　　 testClick(){
　　　　　　　　 this.content = '改变了的值'
　　　　　　　　 // 这时候直接打印的话，由于dom元素还没更新
　　　　　　　　 // 因此打印出来的还是未改变之前的值
　　　　　　　　 console.log(this.$refs.tar.innerText) // 初始值
　　　　　　 }
　　　　 }
--------------------------------------------------------------------------------------     
     methods：{
         testClick() {
             this.content = '改变了的值'
             this.$nextTick(() => {
                 // dom元素更新后执行，因此这里能正确打印更改之后的值
                 console.log(this.$refs.tar.innerText) // 改变了的值
             })
         }
		}
    }
</script>
```

**Html中setInterval()用法**

```js
setInterval(code,millisec[,"lang"])
```

| **参数** |                        **描述**                        |
| :------: | :----------------------------------------------------: |
|   code   |          必需。要调用的函数或要执行的代码串。          |
| millisec | 必须。周期性执行或调用 code 之间的时间间隔，以毫秒计。 |

setInterval() 方法会不停地调用函数，直到 clearInterval() 被调用或窗口被关闭。由 setInterval() 返回的 ID 值可用作 clearInterval() 方法的参数。

### Vue中路由问题

```javascript
//定义路由index.js页面 
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
//创建路由实例
const routes = [{
  path: '/screen/demo',
  name: 'screenDemo',
    //TODO: 
  component: () => import('@/views/screen/demo/index')
}]
//传入路由实例
const router = new VueRouter({
  //mode: 'history',
  mode: 'hash',
  routes
})
//导出模块，一个js文件只能导出一个默认模块，可以导出多个模块；导出之后可以在其它vue页面使用import导入
export default router
-----------------------------------------------------------------------------------
 //main.js
import Vue from 'vue';
import App from './App.vue';
import router from '@/screen/router/index';
import store from '@/screen/store/index';
        //创建和挂载根实例。通过 router 配置参数注入路由，从而让整个应用都有路由功能
        var vm = new Vue({
          router,
          store,
          render: (h) => h(App),
        }).$mount('#app');
------------------------------------------------------------------------------------
//App.vue页面
        //经过上面的配置之后呢，路由匹配到的组件将会渲染到App.vue里的<router-view></router-view>
        //那么这个App.vue里应该这样写：
<template>
  <div id="app">
   <router-view />
  </div>
</template>

<style lang="scss">
body {
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>
--------------------------------------------------------------------------------------
//index.html页面
        //index.html里呢要这样写：
            <body>
                <div id="app"></div>
            </body>
        //这样就会把渲染出来的页面挂载到这个id为app的div里了。
```

