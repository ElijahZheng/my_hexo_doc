title: express 搭建后台
author: Elijah Zheng
tags:
  - node
  - express
  - vue
categories: []
date: 2018-01-18 09:52:00
---
安装 express
```js
express sudo cnpm i express -g
```

安装 express 生成器 
```js
cnpm install express-generator -g
```

安装 mongodb 数据库 
```js
#mac
brew install mongodb 
```

<!--more-->

[MongoDB 极简实践入门](https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/MongoDB%20%E6%9E%81%E7%AE%80%E5%AE%9E%E8%B7%B5%E5%85%A5%E9%97%A8.md)

启动mongodb 的方式
```#mac js 	
mongod —config /usr/local/etc/mongod.conf
(登录权限需要自行设置)
```

下载 mongodb GUI 工具 Robo 3T	
自行安装

安装 mongoose	
node.js异步环境下对mongodb进行便捷操作的对象模型工具
``` js
cnpm install mongoose --save
```

创建一个名为server的应用	
``` js
express server
```

安装依赖
``` js
cd server
cnpm install
```

启动应用的方式
``` js
# Mac or Linux	
DEBUG=myapp npm start 
# Windows	
set DEBUG=myapp & npm start 
```

操作mongodb，增加数据库myapp及集合goods
```js 
# 增加数据库
use myapp

# 插入数据
db.goods.insert({'name': 'apple', 'price': '5'})
```

在server目录下创建文件夹，用于保存模型models
如：
创建一个goods模型
``` js
goods.js
let mongoose = require('mongoose');
let Schema = mongoose.Schema;

let productSchema = new Schema({
    'name': String,
    'price': Number
});
module.exports = mongoose.model('Good', productSchema);
// 会自动将Good与goods集合相匹配，或者可以用下面的方法指定需要匹配的集合
//module.exports = mongoose.model('Good', productSchema, 'goods');
```

然后在router中查询goods的数据
``` js
var express = require('express');
var router = express.Router();
var mongoose = require('mongoose');
var Goods = require('../models/goods');
//连接数据库
// mongoose.connect('mongodb://127.0.0.1:27017/myapp');
mongoose.connect('mongodb://localhost/konggu');
mongoose.connection.on('connected', () => {
    console.log('MongoDB connected success.')
})

mongoose.connection.on('error', () => {
    console.log('MongoDB connected fail.')
})

mongoose.connection.on('disconnected', () => {
    console.log('MongoDB connected disconneted.')
})

router.get('/', function(req, res, next) {
    // res.send('respond with a resource of goods');
    Goods.find({}, (err, doc) => {
        if (err) {
            res.json({
                status: '1',
                mes: err.message
            })
        } else {
            res.json({
                status: '0',
                msg: 'success',
                result: {
                    count: doc.length,
                    list: doc
                }
            })
        }
    })
});

module.exports = router;
```

chrome json格式化插件
[jsonview](https://github.com/gildas-lormeau/JSONView-for-Chrome)