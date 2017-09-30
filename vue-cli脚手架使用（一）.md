title: vue-cli脚手架使用（一）-- 起步、组件、路由
author: 给你灵感奶昔
tags:
  - vue
categories:
  - 爬坑
date: 2017-07-25 13:23:00
thumbnail: https://cdn.zhengxiangling.com/47848384.jpg
---
使用vue-cli脚手架之前，建议对vue有一定了解之后再使用构建工具。	
  
  ### 起步
``` bash
# 全局安装 vue-cli
$ npm install --global vue-cli
# 创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
# 安装依赖，走你
$ cd my-project
$ npm install
$ npm run dev

```

<!--more-->
#### vue init webpack my-project 

执行完 ``vue init webpack my-project``之后会有下列提示
![](https://cdn.zhengxiangling.com/44601160.jpg)

其中ESlint为js代码规范检测，不符合代码规范的会直接报错，而不是警告。

Karma + Mocha + e2e 都是测试工具，用不到可以不用安装。

脚手架中npm run 分两种模式，一种dev开发模式，另一种bulid生产模式

#### npm install
之后玩之后的文件夹目录结构
![](https://cdn.zhengxiangling.com/47660429.jpg)

其中	
*  config 	
index.js 有修改npm run dev 的端口号 (port).
*  src	
components放vue模板	
router里index.js为路由的相关配置	
App.vue 为主	.vue	
main.js 为主 .js配置
*	根目录	
	index.html为html主入口
   
### .vue 中使用css预处理器
例如安装sass	
法一：找到根路径的package.json
``` js
"dependencies": {
    ...
    "sass-loader": "^6.0.6",
    ...
  }
```
找到dependencies加入``"sass-loader": "^6.0.6"``，	^代表版本大于等于所填版本号。	
然后执行	
	
    $ npm install	

法二：

	npm install sass-loader  --save

然后就可以在 .vue中使用sass了
``` css
<style lang="scss" rel="stylesheet/scss">
  h1 { color: #666 }	
</style>
```

### 使用组件
在components中新建header文件夹，里面新建header.vue文件	
``` html
<template>
  <div class="header">
	我是header
  </div>
</template>

<script type="text/ecmascript-6">
  export default {}
</script>

<style lang="scss" rel="stylesheet/scss">
	
</style>
```

注意，要在script写``export default {}``导出该模板

然后在App.vue中引入该组件
``` js
<script>
// 引入 header.vue 组件
import Header from './components/header/header.vue'
export default {
  name: 'app',
  // 注册该组件，标签名为 v-header
  components: {
    'v-header': Header
  }
}
</script>
```
然后就可以在template中使用该组件了

``` html
<template>
  <div id="app">
    <div class="header">I am header</div>
    <v-header></v-header>
  </div>
</template>
```

### 使用路由

在router中index.js配置路由
``` js
import Goods from '@/components/goods/goods'
import Ratings from '@/components/ratings/ratings'

Vue.use(Router)

export default new Router({
  routes: [{
      path: '/goods',
      name: 'goods',
      component: Goods
    },
    {
      path: '/ratings',
      name: 'ratings',
      component: Ratings
    }
  ]
})
```
其中Goods和Ratings为自己写好的放在components的组件。
其中
* @代表文件夹根路径，(可以在build/webpack.base.conf.js中resolve找到)。
* path 表示要注册的网页子链接
* name 代表引用链接的别名，当链接很长时，使用name来写router-link时会很方便。
* component 代表点击链接时router-view要显示的组件。

然后就可以在App.vue中使用路由了

// App.vue中
``` html
<div class="tab">
    <div class="tab-item">
        <router-link :to="{ name: 'goods'}">商品</router-link>
    </div>
    <div class="tab-item">
        <router-link to="/ratings">评论</router-link>
    </div>
</div>

<router-view></router-view>

```

```
<router-link></router-link>
```
会被渲染成``<a></a>``标签	
链接填入有两种方式，直接填入型
* 	to="/ratings" 为直接链接
*	:to"{ name='goods' }" 为取链接name型，name为router中index.js配置好的链接别名

```
<router-view></router-view>
```
为点击链接后要显示内容

点击链接时，链接会默认有一个class类名router-link-exact-active，要想点击切换到自己自定义的class类名（方便修改样式），可以在src/router/index.js中配置
``` js
export default new Router({
  routes: [{
      path: '/goods',
      name: 'goods',
      component: Goods
      }
    ...
    ],
  linkActiveClass: 'active'
})
```
其中``linkActiveClass: active``的active为自定义的类名，点击链接时就会额外加入自己自定义的class了。
