title: vue 使用 fastclick 和 lazyload 图片懒加载
author: Elijah Zheng
tags:
  - vue
  - fastclick
  - lazyload
categories: []
date: 2017-10-13 13:41:00
---
#### 使用fastclick


fastclick用来处理移动端 click 事件 300 毫秒延迟。

##### 为什么会存在延迟？
根据谷歌开发者文档
> ...mobile browsers will wait approximately 300ms from the time that you tap the button to fire the click event. The reason for this is that the browser is waiting to see if you are actually performing a double tap.

从点击屏幕上的元素到触发元素的 ``click`` 事件，移动浏览器会有大约 300 毫秒的等待时间。为什么这么设计呢？ 因为它想看看你是不是要进行双击（double tap）操作。

<!--more-->

npm安装 

``` bash
npm install --save fastclick```

在根目录的src/main.js中

``` js
import fastclick from 'fastclick'
fastclick.attach(document.body)
```

#### 使用 lazyload 图片懒加载

lazyload可以延迟加载长页面中的图片。在浏览器可视区域外的图片在初始状态下不会被载入，直到用户将页面滚动到它们所在的位置。

npm安装
``` bash
npm install --save lazyload```

``` js
import VueLazyload from 'vue-lazyload'
Vue.use(VueLazyload, {
  loading: require('@/common/images/defalut.png')
})
```

然后在需要懒加载的``<img>``加上``v-lazy属性``即可。
```
<img v-lazy="图片url">
```