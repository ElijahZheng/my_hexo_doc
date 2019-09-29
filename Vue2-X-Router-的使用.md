title: Vue2.X Router 的使用
author: Elijah Zheng
tags:
  - Vue
  - vue-router
categories: []
date: 2017-10-14 10:23:00
---
用 Vue.js + vue-router 创建单页应用，是非常简单的。使用 Vue.js ，我们已经可以通过组合组件来组成应用程序，当你要把 vue-router 添加进来，我们需要做的是，将组件(components)映射到路由(routes)，然后告诉 vue-router 在哪里渲染它们。

<!--more-->
##### 安装

默认vue-cli初始化（init）的时候会提示是否选择安装（选择yes），安装就完成了。

##### 使用
###### 基本使用
vue
```html
<template>
  <div id="app">
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/index">Go to Index</router-link>
    <!-- 路由出口 -->
    <!-- 路由匹配到的组件将渲染在这里 -->
    <router-view></router-view>
  </div>
</template>
```

``router/index.js``
```js
import Vue from 'vue'
import Router from 'vue-router'
// import index.vue 页面
import Index from '@/components/index'

Vue.use(Router)

export default new Router({
  routes: [
  {
    path: '/index',
    name: 'index',
    component: Index
  }]
})
```

这样就会在``<router-link to="/index">Go to Index</router-link>``中渲染一个a标签，点击跳转到``/index``，然后在``<router-view></router-view>``中渲染组件``index.vue``

###### 配置重定向
```js
export default new Router({
  routes: [
  {
    path: '/',
    redirect: '/index'
  },
  {
    path: '/index',
    name: 'index',
    component: Index
  }]
})
```
这样在访问根路径为``/``时，会跳转到``/index``路径，达到访问默认域名时，直接跳转到首页的效果。

###### 配置点击路由时，路径的默认class类名（linkActiveClass）
```js
export default new Router({
  routes: [
  {
    path: '/',
    redirect: '/index'
  },
  {
    path: '/index',
    name: 'index',
    component: Index
  }]
  ,
  linkActiveClass: 'nav-active'
})
```
如果不配置，默认值为``router-link-exact-active``

###### 配置路由的子路由及动态路由器配
```js
import ArticleDetail from '@/components/article_detail/article_detail'
export default new Router({
  routes: [
  {
    path: '/',
    redirect: '/index'
  },
  {
    path: '/index',
    name: 'index',
    component: Index,
    children: [
      {
        path: '/index/:id',
        component: ArticleDetail
      },
      {
        path: '/search',
        component: Search
      }
    ]
  }]
  ,
  linkActiveClass: 'nav-active'
})
```
配置子路由，只需在父路由里添加children属性，然后配置子路由即可，并在子路由的页面添加``<router-link></router-link>``。	
动态路由``/index/:id/``的 ``id``值可以用``this.$route.params.id``取到。