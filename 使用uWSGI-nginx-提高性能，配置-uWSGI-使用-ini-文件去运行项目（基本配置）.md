title: 使用uWSGI + nginx 提高性能，配置 uWSGI 使用 .ini 文件去运行项目（基本配置）
author: Elijah Zheng
tags:
  - uWSGI
  - nginx
  - 服务器
categories: []
date: 2017-10-27 14:16:00
---
快速配置 uWSGI + nginx 运行Django 项目

<!--more-->

1.新建 Django 项目
``` bash
django-admin startproject site_uwsgi
cd site_uwsgi
```

2.对网站进行 nginx 配置

目录中新建 ``site_uwsgi.conf`` 文件

```js
server {
    listen  8001;
    server_name your_server_name.com;
    client_max_body_size 50m;


    location /static/ {
        alias /data/www/vhosts/site_uwsgi/static/;
    }

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        uwsgi_pass  web;
        include /data/www/vhosts/site_uwsgi/uwsgi_params;
    }
    access_log /var/log/nginx/site_uwsgi.access_log;
    error_log /var/log/nginx/site_uwsgi.error_log;
}

upstream web {
    server unix:///data/www/vhosts/site_uwsgi/socket.sock;
}
```
连接进 ``/etc/nginx/sites-enabled/``中
```bash 
sudo ln -s /data/www/vhosts/site_uwsgi/site_uwsgi.conf /etc/nginx/sites-enabled/
```

3.编辑 ``site_uwsgi/settings.py`` 文件及收集静态资源

```js
ALLOWED_HOSTS = ['...'] # 有服务器的话需增加服务器ip
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

收集静态资源
```bash
python manage.py collectstatic
```

4.目录中新建 ``uwsgi_params`` 文件
```js
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

5.修改 ``/etc/nginx/nginx.conf`` 的 ``user`` 为 ``root`` 用户

重启 nginx 
```bash
sudo /etc/init.d/nginx restart
```

6.配置 uWSGI 使用 .ini 文件去运行项目
``site_uwsgi.ini``
```js
# site_uwsgi.ini file
[uwsgi]
# Django-related settings
# the base directory (full path)
chdir           = /data/www/vhosts/site_uwsgi/
# Django's wsgi file
module          = site_uwsgi.wsgi
# the virtualenv (full path)
# process-related settings
# master
master          = true
# maximum number of worker processes
processes       = 30
# the socket (use the full path to be safe
socket          = /data/www/vhosts/site_uwsgi/socket.sock
# ... with appropriate permissions - may be needed
chmod-socket    = 664
# clear environment on exit
vacuum          = true
# pidfile
pidfile         = /data/www/vhosts/site_uwsgi/site_uwsgi.pid
# logger
# daemonize       = /data/www/vhosts/site_uwsgi/access.log
```

7.后台挂载 uwsgi:
```
nohup uwsgi --ini mysite_uwsgi.ini &
```

输入 ``ip:8001``, ``done``

![](https://cdn.zhengxiangling.com/17-10-27/97231926.jpg)

8.挂载之后，每次修改models.py，需要 reload uwsgi，和重启数据库
```bash
uwsgi --reload yxpp.pid
service mysql restart
```