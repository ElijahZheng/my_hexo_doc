title: linux 安装 python3.6 及 使用 Virtualenv 管理python版本
author: Elijah Zheng
tags:
  - python
  - linux
categories: []
date: 2017-08-27 14:59:00
---
购买的云服务器装机的linux会自带python2.7和python3的版本，但是python3一般都不是最新版本的，所以想升级到最新版本，还得手动升级。文章主要讲怎么安装python3.6版本，使用Virtualenv管理python版本，使用豆瓣镜像的pip提速pip下载，以及一些安装时会遇到的bug。

<!--more-->

1.安装必要的依赖

``` bash
sudo apt-get update
apt-get install yum
yum -y groupinstall development
yum -y install zlib-devel
yum -y install sqlite-devel
apt-get install make
```

2.安装 gcc、make 和 zlib 压缩/解压缩库：

``` bash
aptitude -y install gcc make zlib1g-dev
```

3.安装python3.6

``` bash
//已经cdn加速
 wget https://cdn.zhengxiangling.com/Python-3.6.2.tar.xz
 tar xJf Python-3.6.2.tar.xz
 cd Python-3.6.2
 ./configure --with-ssl
 make && make altinstall
```
``--with-ssl`` 是为了给 python3 增加 ssl模块

4.查看python3的版本

``` bash
 python3 -V
```

5.配置pip，使用豆瓣镜像加速下载

```
cd ~/.pip
vim pip.conf
```
  然后在``pip.conf``中添加内容，
  按 `a`
  然后添加
``` bash
[global]
trusted-host = pypi.douban.com
index-url = http://pypi.douban.com/simple
```
  写好后按``wq``保存

6.下载virtualenv

``` bash
pip install virtualenv
//涉及到权限时，使用 sudo 执行
sudo pip install virtualenv
```

7.创建python虚拟环境

上面安装的python3.6的文件夹路径默认是在 ``/usr/python3``中


``` bash
//进到用到存放虚拟环境文件夹的路径，一般放在 ~ 中
cd ~
virtualenv -p /usr/local/bin/python3.6 env3.6
```
8.激活virtualenv
 
``` bash
cd env3.6
source ./bin/activate
```
  
  这样就可以激活 virtualenv了

  如果在其他文件夹操作时，想激活虚拟环境，也可以执行
  
  ``` bash
source ~/env3.6/bin/activate
```
  来进行激活

9.关闭virtualenv

``` bash
deactivate
```