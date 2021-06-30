title: hexo博客搭建
author: 给你灵感奶昔
tags: []
categories:
  - 爬坑
date: 2017-06-26 21:56:00
---
一、技术栈
 1. Markdown
 2. Hexo
 3. git
 
二、工具
 1. 图床 -- [极简图床](https://jiantuku.com/)
 
三、开始搞事情

>> 注：此教程使用的命令基于 Linux, 在Window系统上的安装方式会稍有不同。

<!--more-->

1.Hexo是什么？
简单地说，Hexo是一个轻量级的Node.js博客框架，反正我们搭建博客的框架就是他了。

2.install git
```bash 
sudo apt-get install git-core
// 测试安装成功，输出版本号即安装成功
git version
```

3.install node
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.6/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

command -v nvm

nvm install stable
// 测试安装成功，输出版本号即安装成功
node --version
```

4.国内的npm速度受限，可以使用淘宝镜像加速。
```bash
sudo npm install cnpm -g --registry=https://registry.npm.taobao.org
// 测试安装成功，输出版本号即安装成功
cnpm version
```

5.install hexo
```bash
sudo cnpm install -g hexo-cli
// 测试安装成功，输出版本号即安装成功
hexo version
```

6.注册 极简图床 和 Coding 账号

7.开始搭建	
```bash
// 新建用于存放博客的文件夹
mkdir hexo-start
cd hexo-start

// 初始化hexo
hexo init
cnpm install
```

目录结构
```
.
├── _config.yml	
├── package.json	
├── scaffolds	
├── source	
|   ├── _drafts	
|   └── _posts	
└── themes 	
```

8.run起来
```bash
hexo g
hexo server
```
然后访问http://localhost:4000/，访问成功说明成功了第一步了。

9.使用hexo-admin可视化编写博客	

1)install
```bash
cnpm install --save bcrypt-nodejs
cnpm install --save hexo-admin
```
2)创建admin的密码
```bash
node
> const bcrypt = require('bcrypt-nodejs')
// undefined
> bcrypt.hashSync('your_ password')
//'$2a$10$cHnT1Ri... 这是密码的哈希映射值'
```

3）编辑_config.yml，配置用户名和密码，加入
```js
admin:
  username: user_name
  password_hash: 填入前面生成的密码的哈希值
  secret: a secret something (用于产生cookie值的，可以不用变)
```
重新
```bash
hexo clean
hexo g
hexo server
```

此三句命令是重复的，我们加到``.sh``文件然后每次用``bash``命令执行即可。

新建``restart.sh``文件
```bash
hexo clean
hexo g
hexo server
```
然后每次执行
```bash
bash restart.sh
```
即可

然后访问http://localhost:4000/admin，出现下图说明admin安装成功
![](https://cdn.zhengxiangling.com/17-11-4/24341704.jpg)

10.实现RRS订阅功能
```bash
cnpm install hexo-generator-feed --save
```

然后配置_config.yml
```js
# feed
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
   content
```
```
bash restart.sh
```
即http://localhost:4000/atom.xml

11.更换主题	
https://hexo.io/themes/	
其他主题自选，这里使用NexT主题作为演示

1)install
```bash
git clone https://github.com/theme-next/hexo-theme-next.git
```
2)启用主题
修改_config.yml
```bash
// theme: landscape
theme: next
```
然后

```bash
bash restart.sh
```

重新访问，主题更换了说明更换主题成功。	
NexT主题更多配置（修改样式及第三方服务）自行查看文档。
http://theme-next.iissnan.com/getting-started.html

12.将静态页面部署到 coding 上
1)install hexo-deployer-git 插件
```bash
cnpm install hexo-deployer-git --save
```

2)coding新建项目，hexo-start	

3)然后修改``_config.yml``

修改 root，将root改为coding上面的项目名称
```bash
# URL
...
root: /hexo-start/
...
permalink: :year/:month/:day/:title/
permalink_defaults:
```
修改远程coding的地址
![](https://cdn.zhengxiangling.com/17-11-4/21989627.jpg)
``repo``要填的仓库地址如图所示的位置，点击复制即可
```bash
deploy:
  type: git
  repo: 仓库地址
  branch: master
  message: blog update
```

4)修改完之后，部署到 coding 上
```bash
hexo deploy
```

5)开启 Pages 服务

![](https://cdn.zhengxiangling.com/17-11-4/88756341.jpg)

点击 Pages服务，部署来源选择 master 分支，点击 保存。显示 Coding Pages 已经运行在 http://... ,说明已经成功将博客部署在 coding 上了。

![](https://cdn.zhengxiangling.com/59551e572a02a.jpg)

>> 代码地址 https://github.com/ElijahZheng/hexo-start