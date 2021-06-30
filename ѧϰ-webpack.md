title: 学习 webpack
author: Elijah Zheng
tags:
  - webpack
categories: []
date: 2018-03-11 12:18:00
---
##### learn-webpack
学习webpack

##### 环境
node 8.9.1  
npm 5.7.1   
webpack 4   

##### script
开发模式 - npm run dev  
打包 - npm run build    
运行打包后的文件 - npm run server   

##### 实现
1. 多入口 html - 配置入口 `config/entry.json`   
2. base64 处理limit限制以下的图片   
3. css预处理    
4. dev模式下热重载  
5. babel-loader 转es5
6. image-webpack-loader 压缩图片    
7. SplitChunksPlugin 提取公共js
8. postcss-loader 自动增加css3前缀
9. html-withimg-loader 增加html识别图片路径
10. 分类css img js 到不同的文件夹中

[github传送门](https://github.com/ElijahZheng/learn-webpack.git)