title: 使用uWSGI + nginx 提高性能，部署Django项目（入门）
author: Elijah Zheng
tags:
  - nginx
  - Django
  - 性能
categories: []
date: 2017-10-26 21:03:00
---
##### 概念

Web服务器是面向外部世界的。它可以直接从文件系统中提供（HTML, images, CSS 等）服务文件。然而，它不能直接和Django应用进行通信；它需要可以运行应用程序的东西，从web客户端（如浏览器）中请求它和返回响应。

Web服务网关接口（Web Server Gateway Interface） - WSGI - 扮演着这个角色。WSGI 是基于Python的一个标准。

uWSGI 是一个Web服务器，实现了 WSGI、uwsgi、http等协议。

<!--more-->

##### 使用 uWSGI 前的准备工作
具体代码基于 Linux(Ubuntu)，window 和 mac 的环境安装方法和引用的文件路径可能会存在偏差。
###### virtualenv（自选）
Python的虚拟环境，可以切换不同python的版本。这里不介绍virtualenv的安装。
``` bash
virtualenv uwsgi-tutorial
cd uwsgi-tutorial
source bin/activate
```

###### Django

安装 Django
```bash
pip install Django
django-admin.py startproject mysite
cd mysite
```
##### uWSGI 基本安装和配置

###### 安装 uWSGI
```bash
pip install uwsgi
```
当然还有其他的方法来安装uWSGI，不过``pip``来安装是最好的安装方法之一。

###### 基本测试

创建 ``test.py`` 文件
```
# test.py
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"] # python3
    #return ["Hello World"] # python2
```

>>	Python3 返回字符串时要求用bytes()

运行 uWSGI:
```bash
uwsgi --http :8000 --wsgi-file test.py
```
选项参数说明：

| 参数        |   解释   |    
| ------------- |:-------------:| 
| http:8000	      | 使用http协议，端口8000 |
| wsgi-file test.py    | 加载指定的文件，test.py |

###### 测试你的Django项目

现在我们用 uWSGI 去运行Django网站。

先测试你的``mysite``项目已经可以运行:
```bash
python manage.py runserver 0.0.0.0:8000
```
如果已经可以运行，换用uWSGI来运行:
```bash
uwsgi --http :8000 --module mysite.wsgi
```

``module mysite.wsgi``： 加载指定的wsgi模块

如果能够成功进行访问，说明uWSGI已经能够对Django应用服务。

##### nginx 基本配置
###### 安装 nginx
```bash
sudo apt-get install nginx
sudo /etc/init.d/nginx start    # start nginx
```

###### 对网站进行 nginx 配置

现在需要 ``uwsgi_params``文件，在 nginx 目录中 uWSGI分配时会用来。可以从 https://github.com/nginx/nginx/blob/master/conf/uwsgi_params 中复制此文件放在项目的目录中。然后在 nginx 中将会使用到它。

现在创建一个 ``mysite_nginx.conf`` 文件，将下列信息填入：
```js
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    #server unix:///data/www/vhosts/mysite/socket.sock; #for a file socket
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name your_server_name.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /data/www/vhosts/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /data/www/vhosts/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /data/www/vhosts/mysite/uwsgi_params; # the uwsgi_params file you installed
    }
}
```
这个配置文件告诉nginx去服务从系统文件来的媒体和静态文件，和处理来自Django干预的请求。对于大规模部署的项目，需要很好的考虑让一个服务去处理静态/媒体资源，另一个去处理Django的应用，但是现在这个配置已经够用了。

然后用链接符号``ln``将该文件放到``/etc/nginx/sites-enabled``中
```bash
sudo ln -s /data/www/vhosts/mysite/mysite_nginx.conf /etc/nginx/sites-enabled/
```
###### 部署静态文件

在运行 nginx 之前，你需要手机 Django 的静态文件放到静态文件夹中。
编辑 ``mysite/settings.py`` 增加：
```python
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```
然后执行
```
python manage.py collectstatic
```

###### nginx 基本测试
重启 nginx：
```bash
sudo /etc/init.d/nginx restart
```
在``mysite``项目新建``media``文件夹，在其中放入一张``media.png``图片，访问 http://0.0.0.0:8000/media/media.png ，如果成功访问，说明 nginx 已成功对文件进行服务。

如果访问失败，看错误日志然后做相应的修改，然后重启 nginx。

##### 使用 Unix sockets 来代替端口
到目前为止，我们已经使用了TCP端口的套接字（socket），因为它比较的简单，但实际上使用 Unix socket 会更好 - 开销较少。

编辑 `` mysite_nginx.conf ``
取消下列注释
```
server unix:///data/www/vhosts/mysite/mysite.sock; # for a file socket
```

然后重启 nginx
然后重新运行 uWSGI：
```bash
uwsgi --socket mysite.sock --wsgi-file test.py
```

这次是使用 ``socket``选项去告诉配置 uWSGI 。

如果启动失败，检查 nginx 错误日志 （/var/log/nginx/error.log）

如果出现
``connect() to unix:///data/www/vhosts/mysite/mysite.sock failed (13: Permission denied)``
修改 ``/etc/nginx/nginx.conf``的``user``为``root``用户

##### 配置 uWSGI 使用 .ini 文件去运行项目
创建 ``mysite_uwsgi.ini`` 文件：
```js
# mysite_uwsgi.ini file
[uwsgi]

# Django-related settings
# the base directory (full path)
chdir           = /data/www/vhosts/mysite/
# Django's wsgi file
module          = mysite.wsgi
# the virtualenv (full path)

# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 30
# the socket (use the full path to be safe
socket          = /data/www/vhosts/mysite/socket.sock

# ... with appropriate permissions - may be needed
chmod-socket    = 664
# clear environment on exit
vacuum          = true
# pidfile
pidfile         = /data/www/vhosts/mysite/mysite.pid
# logger
# daemonize       = /data/www/vhosts/mysite/access.log
```

然后使用该文件运行 uswgi：

``` bash
uwsgi --ini mysite_uwsgi.ini # the --ini option is used to specify a file
```


>> 参考文献：Setting up Django and your web server with uWSGI and nginx  
http://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html#basic-uwsgi-installation-and-configuration