title: Git入门
author: 给你灵感奶昔
tags:
  - git
categories:
  - 爬坑
date: 2017-07-07 10:36:00
---
### 一、git实现简单的文件上传
***
（这里会用到coding）[->](https://coding.net/)

#### 创建自己的项目
* 基础选项
	* 项目名称 -- 建议用全英文
    * 属性 -- 公开
*  初始化仓库
	* 启用README.md文件初始化项目
    
<!--more-->

#### clone到本地仓库
![upload successful](https://cdn.zhengxiangling.com/17-8-2/99368394.jpg)
选择要clone到的文件夹，比如我要clone到桌面，我的桌面名字为Desktop

``` bash
$ cd Desktop
$ git clone 你刚才复制的地址
# clone完成之后进入clone下来的文件夹里
$ cd [your_dir]
```

#### 在clone下来的文件夹里新建文件

* 然后进入clone好的文件夹里新建一个自定义文件，比如新建一个666.txt


![upload successful](https://cdn.zhengxiangling.com/17-8-2/28532100.jpg)

#### 查看此时本地仓库的状态
``` bash 
# 使用 git status 命令
$ git status
```

![upload successful](https://cdn.zhengxiangling.com/17-8-2/76557526.jpg)

#### 将修改文件添加到暂缓区（index）
``` bash
# 使用 git add 命令
$ git add 666.txt
```

#### 确认无误后添加到本地仓库（repository）
``` bash
# 使用 git commit 命令 （-m 表示提交的信息）
$ git commit -m 'my_message' (注意 -m 前后面都有一个空格)
```

附：(没有则跳过)
如果提示出现please tell me who you are，按照求输入就行了.

```
git config --global user.email '填写你的邮箱'
```

```
git config --global user.name '你的名字'
```


#### 本地修改完成后提交到远程仓库（remote）
``` bash
# 使用 git push 命令
$ git push
```

###  二、修改工作区文件，传到远程仓库
***
* 修改666.txt的内容,添加信息 hello world

![upload successful](https://cdn.zhengxiangling.com/17-8-2/44073607.jpg)

* 添加到暂缓区
``$ git add 666.txt``

* 添加到本地仓库
``$ git commit -m '我添加了一些信息'``

* 上传到远程仓库
``$ git push``


![upload successful](https://cdn.zhengxiangling.com/17-8-2/70902125.jpg)

然后就可以看到上传的文件和备注的一些信息了。

> 分享
廖雪峰的官方网站-Git教程
http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000


>	如有疑惑，欢迎留言提出~