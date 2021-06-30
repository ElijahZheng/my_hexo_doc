title: css居中
author: 给你灵感奶昔
tags:
  - css
categories: []
date: 2017-07-08 10:34:00
---
css的居中方式有很多种
* 文本居中 text-align
* margin居中 0 auto
* 定位属性居中 

这里说的是margin来居中，建议减少定位position的使用。

<!--more-->

会用到的css属性

``` css
# 最小宽度
min-width: 1200px;

# 最大宽度
max-width: 1920px;
```

效果图
![upload successful](https://cdn.zhengxiangling.com/60886452.jpg)

banner及footer属性
``` css
width: 100%;
min-width: 1200px;
max-width: 1920px;
```

公用容器类container：
``` css
.container {
  width: 1000px;
  margin: 0 auto;
}
```

> 源码（放在github，自行clone下来）
  git 地址 https://github.com/zzzzzzxl/css_
