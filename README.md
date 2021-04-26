####  为什么选 Vite（快）

- 快(用过你就知道了），为什么快。

  Vite 通过在一开始将应用中的模块区分为 **依赖** 和 **源码** 两类，改进了开发服务器启动时间。

  - **依赖** 大多为纯 JavaScript 并在开发时不会变动。一些较大的依赖（例如有上百个模块的组件库）处理的代价也很高。依赖也通常会以某些方式（例如 ESM 或者 CommonJS）被拆分到大量小模块中。

    Vite 将会使用 [esbuild](https://esbuild.github.io/) [预构建依赖](https://cn.vitejs.dev/guide/dep-pre-bundling.html)。Esbuild 使用 Go 编写，并且比以 JavaScript 编写的打包器预构建依赖快 10-100 倍。

  - **源码** 通常包含一些并非直接是 JavaScript 的文件，需要转换（例如 JSX，CSS 或者 Vue/Svelte 组件），时常会被编辑。同时，并不是所有的源码都需要同时被加载。（例如基于路由拆分的代码模块）。

    Vite 以 [原生 ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) 方式服务源码。这实际上是让浏览器接管了打包程序的部分工作：Vite 只需要在浏览器请求源码时进行转换并按需提供源码。根据情景动态导入的代码，即只在当前屏幕上实际使用时才会被处理。

####  使用

​	在某个目录下执行下列命令，然后按照提示操作即可。

```bash
npm init @vitejs/app

yarn create @vitejs/app
```

还可以通过附加的命令行选项直接指定项目名称和你想要使用的模板。例如，要构建一个 Vite + Vue 项目，运行:

```bash
# npm 6.x
npm init @vitejs/app my-vue-app --template vue

# npm 7+, 需要额外的双横线：
npm init @vitejs/app my-vue-app -- --template vue

# yarn
yarn create @vitejs/app my-vue-app --template vue
```

支持的模板预设包括：

- vanilla

- vue

- vue-ts 

- react

- react-ts

- preact

- preact-ts

- lit-element

- lit-element-ts
  查看 [@vitejs/create-app](https://github.com/vitejs/vite/tree/main/packages/create-app) 获取每个模板的更多细节。

#### vite 的配置

vite配置的文件所在的目录 vite.config.js / vite.config.ts

  - **别名**

  ```ts
  import {defineConfig} from 'vite' //引入后又语法提示
  import vue from '@vitejs/plugin-vue'
  // @ts-ignore
  import path from 'path'
  
  // https://vitejs.dev/config/
  export default defineConfig({
  
    plugins: [vue()],
    resolve: {
      alias: {
        // 别名 即@代表src目录
        "@": path.resolve(__dirname, "src"),
        "comps": path.resolve(__dirname, "src/components"),
        "apis": path.resolve(__dirname, "src/apis"),
        "views": path.resolve(__dirname, "src/views"),
        "utils": path.resolve(__dirname, "src/utils"),
        "routes": path.resolve(__dirname, "src/routes"),
        "styles": path.resolve(__dirname, "src/styles"),
        "assets": path.resolve(__dirname, "src/assets"),
      }
    },
  })
  
  ```

  具体作用如下图 就是@ = src 路径

- **配置服务器代理**

```ts
 export default defineConfig({

  plugins: [vue()],
  server:{  //代理
    proxy:{
      // 字符串简写写法
      '/foo': 'http://localhost:4567/foo',
      // 选项写法
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      },
      // 正则表达式写法
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, '')
      }
    }
  }
})

 

```

- **jsx支持**   

  感觉就是让人写react一样写vue3。

  [详情请见](https://www.pipipi.net/vite/plugins/#%E5%AE%98%E6%96%B9%E6%8F%92%E4%BB%B6)

```vue
 npm i @vitejs/plugin-vue-jsx //仅仅在单文件中使用 而不是.jsx
 
  vite.config.js //

import vueJsx from '@vitejs/plugin-vue-jsx'
export default defineConfig({
  plugins: [vue(),vueJsx()],
  resolve: {
    alias: {
      // 别名
      "@": path.resolve(__dirname, "src"),
      "comps": path.resolve(__dirname, "src/components"),
      "apis": path.resolve(__dirname, "src/apis"),
      "views": path.resolve(__dirname, "src/views"),
      "utils": path.resolve(__dirname, "src/utils"),
      "routes": path.resolve(__dirname, "src/routes"),
      "styles": path.resolve(__dirname, "src/styles"),
      "assets": path.resolve(__dirname, "src/assets"),
    }
  },
})

// comp.vue
<script lang="jsx">
  export default {
    render() {
      return <h1> hello</h1>
    }
  }
</script>

// or
<script lang="jsx">
  import {ref} from 'vue'

  export default {
    setup() {
      const counter = ref(0);
      const onclick = () => {
        counter.value++
      };
      return () => (
        <>
          <h1> hello</h1>
          <p onClick={onclick}>{counter.value}</p>
        </>
      )
    }

  }
```

- **配置mock数据**

```js
yarn add mockjs  vite-plugin-mock -D
 //  vite.config.js 
import { viteMockServe } from 'vite-plugin-mock';
 
plugins: [
    vue(),
    vueJsx(),
    viteMockServe({
      supportTs: false    // 关闭ts
    })
  ],
// 目录结构
  -mock
    -getUser.js
  -src

//getUser.js
export default [
    {
        url: "/api/getUsers",
        method: 'get',
        response: ({ body }) => {
            console.log('body', body);
            return {
                code: 0,
                message: 'ok',
                data: ['tom', 'jerry']
            }
        }
    }
]

// xx.vue

<template>
  <div>
    <button @click="getApi">click</button>
  </div>
</template>

<script setup>
const getApi = () => {
  fetch("/api/getUsers")
    .then((res) => res.json())
    .then((data) => console.log(data));
};
</script>
```

- **vue-router4 和 vuex4**

```js
npm install vue-router@4  vuex@next -S

vue-router4 配置

// router index.js

import {createRouter, createWebHashHistory} from "vue-router";
const routes = [
    {path: '/', component:  () => import('../view/home.vue')},
]
const router = createRouter({
    // 4. 内部提供了 history 模式的实现。为了简单起见，我们在这里使用 hash 模式。
    history: createWebHashHistory(),
    routes, // `routes: routes` 的缩写
})

export default router

//app.vue
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
<!--  <HelloWorld msg="Hello Vue 3 + Vite" />-->
  <router-view></router-view>
</template>


//main.js
import router from "./router";


createApp(App).use(router).mount('#app')



vuex4 配置
 
//store index.js
import {createStore} from "vuex";

export  default createStore({
        state: {
            counter: 0
        },
        mutations:{
            add(state){
                state.counter++
            }
        }
    }
)
//main.js
import store from './store'
createApp(App).use(router).use(store).mount('#app')
//xx.vue
 <h1 @click="$store.commit('add')">{{$store.state.counter}}</h1>
```

