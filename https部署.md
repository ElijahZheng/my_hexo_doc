title: ssh 连接远程服务器
author: 给你灵感奶昔
tags:
  - ssh
  - 服务器
categories: []
date: 2017-09-06 19:29:00
---
登上远程服务器，除了使用登录密码之外，我们还可以使用公钥key来免手动输入密码进行登录。
<!--more-->
这里使用linux服务器，默认启用root账户。

先在远程服务器操作

1.启用root权限用户
```bash 
sudo -i 
```

2.新建存放公钥的.ssh文件夹和放公钥文件
```bash
cd ~
mkdir .ssh
cd .ssh
vim authorized_keys
```
按``a``，然后输入你的公钥``id_rsa.pub``，完成后按``esc`` ``:`` ``w`` ``q``保存退出

3.本地连接远程服务器

在本地电脑
```bash
cd ~/.ssh
vim config
```
添加
```bash
Host remote
    HostName 远程服务器的ip
    Port 22
    User root
    ServerAliveInterval 60
```
``esc`` ``:`` ``w`` ``q`` 保存退出

然后就可以用 
``` bash
ssh root@remote
```
直接访问远程服务器了。