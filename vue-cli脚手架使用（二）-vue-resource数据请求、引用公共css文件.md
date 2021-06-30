title: vue-cli脚手架使用（二）-- vue-resource数据请求、引用公共css文件
author: 给你灵感奶昔
tags:
  - vue
  - scss
categories:
  - 爬坑
thumbnail: >-
  https://cdn.zhengxiangling.com/68747470733a2f2f7261776769742e636f6d2f7675656a732f617765736f6d652d7675652f6d61737465722f6c6f676f2e706e67.png
date: 2017-07-27 12:39:00
---
vue与后台请求数据有自己的请求方式，这里用的是``vue-resource``。
当写有公共``css``时，可以引用``@import``到组件中使用。

### vue-resource

#### 安装.

	$ npm install vue-resource
    
<!--more-->

#### 引用

src/main.js

``` js
import Vue from 'vue'
import App from './App'
import router from './router'
// 引用vue-resource
import VueResource from 'vue-resource'
```
#### 注册

``` js
import router from './router'
// 引用vue-resource
import VueResource from 'vue-resource'

// 注册
Vue.use(VueResource)
```

#### 请求数据

官方给的例子
``` js
{
  // GET /someUrl
  this.$http.get('/someUrl').then(response => {

    // get body data
    this.someData = response.body;

  }, response => {
    // error callback
  });
}
```

error callback 可以不写

App.vue
``` js
export default {
  data () {
    return {
      my_value: {}
    }
  },
  created () {
    this.$http.get(url).then(response => {
      this.my_value = response.body.data  
    })
  }
}
```

调用``created``钩子，当``created``时就请求数据。

``response.body.data`` 返回的是一个对象

用``my_value``来接收数据。

将父组件值传递给子组件

如父组件有一个子组件

	<v-header></v-header>

用``v-bind``将值传递给子组件

	<v-header v-bind:my_value="my_value"></v-header>


(v-bind:my_value可以缩写为 :my_value)

子组件要用props来接受值的传递
``` js
export default {
  data() {
    return {}
  },
  props: {
    my_value: {
      type: Object
    }
  }
}
```
就可以和``data``里面的数据一样用 大胡子语法 来使用了。

引用公共``css``文件
如有 common/scss/common.scss 文件

引用方式 （@import）
```css
<style lang="scss" rel="stylesheet/scss">
  @import "..路径/common/scss/common.scss";
  ...
</style>
```