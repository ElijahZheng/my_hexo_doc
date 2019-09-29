title: Django 实现登录，注册，用 Pillow 添加验证码
author: Elijah Zheng
tags:
  - django
  - pillow
categories: []
date: 2018-02-13 14:52:00
---
用 Django 作为后端，Django 有自己的一套登录和注册方式（User Model），可以对 User Model 进行扩展，新作一张表 user_ext 。	

然后用 Pillow 生成验证码，防止刷用户注册。

[github 传送门](https://github.com/ElijahZheng/django-register.git)