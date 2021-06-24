<!--
 * @Author: tangdaoyong
 * @Date: 2021-06-23 23:22:46
 * @LastEditors: tangdaoyong
 * @LastEditTime: 2021-06-23 23:41:39
 * @Description: Vite动态导入
-->
# Vite动态导入

[Vite动态导入](https://juejin.cn/post/6951557699079569422/)
[Vite动态导入](https://www.cnblogs.com/ljx20180807/p/14665505.html)

## 介绍

`webpack`构建项目时，可以如下分包动态导入。
```js
// 普通导入
import HelloWorld from '../components/HelloWorld.vue'
// 动态导入-webpack重命名
const HelloWorld = () => import(/* webpackChunkName: "group-user" */ './UserDetails.vue')
```
`Vite`中如何实现动态导入呢？

## 问题
// 问题代码 () => import( /* @vite-ignore */ `${fileSrc}`)
routerList.push({
  path: `/${routerPath}`,
  name: `${routerName}`,
  component: () => import( /* @vite-ignore */ `${fileSrc}`)
})
复制代码
之前使用 webpack 构建项目一直使用动态导入 require.context API 自动化注册组件及路由；
转移到 vite 之后，开发习惯当然不能变；随即使用的是 import.meta.globEager 完成动态导入；
本地开发过程中很舒服没问题，打包后部署到服务器报错找不到动态导入的文件；裂开~~~
经过这几天陆陆续续的尝试最终解决，总结了以下几种方案
三、需求
主要项目结构
├── components                    // 公共组件存放目录
└── views                         // 路由视图存放目录
    └── Home                      // Home 页面
        └── components            // Home 页面封装组件存放文件
            └── HomeHeader.vue    // Home 页面头部组件
        └── index.vue             // Home 主入口
        └── types                 // Home页面专属类型
    └── Login                     // Login 页面
        └── components            // Login 页面封装组件存放文件
            └── LoginHeader.vue   // Login 页面头部组件
        └── index.vue             // Login 主入口
        └── types                 // Login页面专属类型
    ....
    ....
复制代码
文件内部
export default defineComponent({
  name: 'home',
  isRouter: true,
  isComponents: false,
  setup() {
    ...
  }
})
复制代码
组件内部通过定义

name: 路由组件名称
isRouter： 是否自动为路由；
isComponents： 是否自动注册为公共组件

四、 解决 （推荐方案二）
vite 动态导入有两种方式


import.meta.glob: 通过动态导入默认懒加载，通过遍历加 then 方法可拿到对应的模块文件详情信息


import.meta.globEager: 直接引入所有的模块, 即静态 import; 我的就是使用该方案打包部署报错


以下方案有需要自行取舍
4.1 方案一
使用 import.meta.glob
缺点：

不使用 then 方法拿不到模块信息，无法进行判断是否需要自动注册组件或路由；
使用了 then 方法成异步的了， 路由渲染的时候文件还没获取成功注册不到

但是你可以用单独文件夹来区分，我认为限制性太大不够优雅；
// global.ts
export const vueRouters = function (): Array<RouteRecordRaw> {
  let routerList: Array<RouteRecordRaw> = [];
  const modules = import.meta.glob('../views/**/*.vue')
  Object.keys(modules).forEach(key => {
    const nameMatch = key.match(/^\.\.\/views\/(.+)\.vue/)
    if(!nameMatch) return
    const indexMatch = nameMatch[1].match(/(.*)\/Index$/i)
    let name = indexMatch ? indexMatch[1] : nameMatch[1];
    // 首字母转小写 letterToLowerCase 首字母转大写 letterToUpperCase
    routerList.push({
      path: `/${letterToLowerCase(name)}`,
      name: `${letterToUpperCase(name)}`,
      component: modules[key]
    });
  })
  return routerList
};
复制代码
使用 router.ts
import { vueRouters } from '../services/global'
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'Login',
    component: () => import('@/views/Login/index.vue')
  },
  ...vueRouters()
]

复制代码
4.2 方案二  “推荐”
使用 import.meta.glob 和 import.meta.globEager

import.meta.glob： 因为 import.meta.glob 获取到的文件就是懒加载的，避免了使用 import 语句， 所以打包后不会报错不存在动态引入了
import.meta.globEager：不使用 then 也能获取到文件全局上下文进行有需要的判断

// global.ts
function getModules() {
  const components = import.meta.glob('../views/**/*.vue')
  return components
}
function getComponents() {
  const components = import.meta.globEager('../views/**/*.vue')
  return components
}
// 自动注册组件
export const asyncComponent = function (app: App<Element>): void {
  const modules = getModules();
  const components = getComponents();
  Object.keys(modules).forEach((key: string) => {
    const viewSrc = components[key];
    const file = viewSrc.default;
    if (!file.isComponents) return
    const AsyncComponent = defineAsyncComponent(modules[key])
    app.component(letterToUpperCase(file.name), AsyncComponent)
  });
  // console.log(app._component.components)
};

// 自动注册路由
export const vueRouters = function (): Array<RouteRecordRaw> {
  let routerList: Array<RouteRecordRaw> = [];
  const modules = getModules();
  const components = getComponents();
  Object.keys(modules).forEach(key => {
     const viewSrc = components[key];
     const file = viewSrc.default;
     if (!file.isRouter) return
     // 首字母转小写 letterToLowerCase 首字母转大写 letterToUpperCase
     routerList.push({
       path: `/${letterToLowerCase(file.name)}`,
       name: `${letterToUpperCase(file.name)}`,
       component: modules[key]
     });
  })
  return routerList
}  
 
复制代码
使用 router.ts （路由注册）
import { vueRouters } from '../services/global'
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'Login',
    component: () => import('@/views/Login/index.vue')
  },
  ...vueRouters()
]

复制代码
使用 main.ts （组件注册）
import { asyncComponent } from './services/global';
export const app = createApp(App)
asyncComponent(app)
复制代码
4.3 方案三
使用 import.meta.glob 的 then 方法， 加上路由内置 addRoute（） 方法注册
缺点：

由于文件懒加载获取， 页面加载有明显的的卡顿
不适用于注册组件
addRoute() 方法适用于根据后台接口返回的路由权限注册鉴权路由

// global.ts
export const vueRouters = function (router: Router): void {
  let routerList: Array<RouteRecordRaw> = [];
  const modules = import.meta.glob('../views/**/*.vue')
   for (const path in modules) {
    modules[path]().then((mod) => {
      const file = mod.default;
      if (!file.isRouter) return
      // 首字母转小写 letterToLowerCase 首字母转大写 letterToUpperCase
      router.addRoute({
        path: `/${letterToLowerCase(file.name)}`,
        name: `${letterToUpperCase(file.name)}`,
        component: file
      })
    })
  }
};
复制代码
使用 router.ts
import { vueRouters } from '../services/global'
const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'Login',
    component: () => import('@/views/Login/index.vue')
  }
]
const router = createRouter({
  history: createWebHashHistory(),
  routes
})
vueRouters(router)