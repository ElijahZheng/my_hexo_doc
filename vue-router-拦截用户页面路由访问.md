title: vue-router 拦截用户页面路由访问
author: Elijah Zheng
tags:
  - vue
  - vue-router
categories:
  - Vue
  - ''
date: 2020-08-15 19:05:00
---
在对需要做登录的系统前端页面访问做控制时，可以用`router`的`beforeEach`方式来对跳转到的路径做管理。如果访问的路径在路径白名单内，则可以继续访问，如果不在，则判断用户是否登录，如果未登录，则被强制跳转到登录页面。

一般在前端，用户登录完成之后，我们会将管理用户状态的`token`值存在Cookie或者session中，所以可以用是否有`token`值来判断用户是否已经做了登录。

权限判断逻辑


```js
// permission.js
// 引入 router
import router from './router'
// 引入用户状态管理用的 store
import store from './store'
// 引入 Cookies 插件
import Cookies from 'js-cookie'

// 路由白名单，访问白名单页面不需要做登录
const whiteList = ['/login']

router.beforeEach(async(to, from, next) => {
    // 获取系统用户 Token，存的 Cookie 名由自己定
    const hasToken = Cookies.get('MySystemToken')
    
    // 判断是否已经登录
    if (hasToken) {
        if (to.path === '/login') {
            // 如果已经是登录状态，访问登录页面，则强制跳转到主页
            next({ path: '/'})
        } else {
            // 判断 store 里是否已经已经有信息，如果没有则需要调接口获取
            const hasGetUserInfo = store.getters.name
            if (hasGetUserInfo) {
                // 有了信息的话，正常执行
                next()
            } else {
                // 如果 store 没有信息，则调接口获取
                try {
                    // 调用 store 里面定义的 user/getInfo 方法获取 user 信息
                    await store.dispatch('user/getInfo')
                    next()
                } catch (error) {
                    // 如果获取用户状态失败，一般是指 token 已获取或者错误，则清除 token，跳转到登录页重新登录
                    await store.dispatch('user/resetToken')
                    console.log(error)
                    next(`/login?redirect=${to.path}`)
                }
            }
        }
    } else {
        // 如果在白名单页面，则继续；否则跳转到登录页面
        if (whiteList.indexOf(to.path) != -1) {
            next()
        } else {
            next(`/login?redirect=${to.path}`)
        }
    }
})
```

写完权限判断逻辑，只需在`vue`框架入口的`main.js`中引入即可，
```js
// main.js

import ./permission

```