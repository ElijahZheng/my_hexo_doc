title: CSS-Sticky-Footer 布局
author: 给你灵感奶昔
tags:
  - CSS布局
  - ''
categories:
  - CSS
thumbnail: 'https://cdn.zhengxiangling.com/pexels-photo-532571.jpeg'
date: 2017-07-27 22:32:00
---
这个布局是用来实现，让底部信息（例如版权）始终保持在页面最底部的效果。

<!--more-->

类似于下面图片的效果，让关闭按钮始终保持在最底部。

![](https://cdn.zhengxiangling.com/10717719.jpg)

代码：
```html
<!-- body --> 
<div class="container">
    <div class="content">content</div>
    <div class="push"></div>
</div>
<div class="footer">footer</div>
```

``container``的负底部外边距与``footer``和``push``的高度相同。``push``的作用是将``footer``推下去，否则``content``内容太多时会与``footer``重叠。

```css
<style>
* {
    margin: 0;
}

html,
body {
    height: 100%;
}

.container {
    min-height: 100%;
    margin: 0 auto -100px;
}

.footer,
.push {
    height: 100px;
}
</style>
```