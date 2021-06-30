title: http到https 并用cdn加速资源
author: Elijah Zheng
tags:
  - https
categories:
  - 服务器
date: 2017-08-03 13:18:00
---
#### 为什么要用https	

https协议的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。

<!--more-->

#### 选择ssl证书
 
有付费的和免费的。列举两个提供免费的证书，Let's Encrypt 和 现在限免的TrustAsia（文章写于2017.08.03）。


1.Let's Encrypt	
这个免费、自动化、开放的证书签发服务。Let's Encrypt 得到了 Mozilla、Cisco、Akamai、Electronic Frontier Foundation 和 Chrome 等众多公司和机构的支持，发展十分迅猛。证书有效期为90天，但可以通过脚本定期更新。

使用 Let's Encrypt 的请参考QuQu大佬的 
> Let's Encrypt，免费好用的 HTTPS 证书	
https://imququ.com/post/letsencrypt-certificate.html

> ps: 选择 Let's Encrypt 的请跳过下面购买证书和使用证书的步骤，因为QuQu大佬已经很详细的说明了 Let's Encrypt 的使用和配置了。

2.亚洲诚信（TrustAsia®）
是亚数信息科技（上海）有限公司应用于信息安全领域的品牌。
可以通过七牛云免费申请，证书有效期一年（博主懒人癌犯了，所以选择了这个，- -）	

我的博客 用的TrustAsia，申请流程	

1) 注册登录 > 七牛云	
   然后点击 SSL证书服务 > 购买证书	
> 附链接：https://portal.qiniu.com/certificate/apply

2) 填写信息	
   信息自己填写，其中需要注意的有域名所有权验证方式：	
* 域名所有权验证方式有两种：
	* DNS验证 ：DNS验证需要到自己的域名管理里添加TXT验证。	
	* 文件验证：在文件夹中添加文件/.well-known/pki-validation/fileauth.txt，验证文件值由证书提供。	

两种方式二选一。		
  
填完信息就是到域名所有权验证方式了，完成验证操作之后，需要等待差不多几分钟的时间就可以看到订单状态是已签发了，这时我们就拿到证书了。
 
#### 使用ssl证书
点击 证书管理 > 我的证书，将刚才申请到的证书下载到本地，将文件更名为`my_cert`(也可以换成自己喜欢的)。

我的系统用的是 Ubuntu 14.04.1 ，代理用的是nginx，nginx目录文件为 `/etc/nginx` （这里未提供其他代理的配置）。

在 `/etc/nginx` 新建目录 `cert` 用于存放证书，然后放入刚才下载并更名好的文件夹`my_cert`

然后配置代理

```bash
server {
    server_name 你的域名;
    listen 443;
    ssl on;
    ssl_certificate /etc/nginx/cert/my_cert/你下载的文件名.crt;
    ssl_certificate_key /etc/nginx/cert/my_cert/你下载的文件名.key;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:port;
    }

    access_log /var/log/nginx/blog.access_log;
    error_log /var/log/nginx/blog.error_log;
}


server {
    listen 80;
    client_max_body_size 50m;
    server_name 你的域名;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```

`https`用的是443端口，`ssl_certificate`及`ssl_certificate_key`填的是存放证书的路径
```bash
server {
    listen 80;
    client_max_body_size 50m;
    server_name 你的域名;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```
的作用是将http转向https。

这时访问你的域名就会被添加https了。

#### 使用cdn加速资源

让站点飞起来~

首先使用cdn加速的话，我们可以重新申请一个证书，专门存放需要加速的资源的。

比如我申请了一个cdn.zhengxiangling.com的ssl证书，因为只是存放加速资源，所以就不用下载证书然后在nginx配置了。

同样以七牛云举例，点击 资源主页 > 立即添加 融合 CDN 加速 （首先得实名和冲10元才能使用该功能 -）

为了加速，去吧，10元（皮卡丘）

然后填写信息，

![](https://cdn.zhengxiangling.com/17-8-3/55247823.jpg)

有证书了当然选择https通信协议啦

源站配置就自己新建一个对象存储，（点击 资源主页 > 立即添加 对象存储）
![](https://cdn.zhengxiangling.com/17-8-3/10207993.jpg)
  
其他信息按自己喜好而外自己选

选好之后点击创建，创建过程需要等待一小段时间，具体多长时间说不准。

创建完成之后需要进入下一步，配置CNAME

![](https://cdn.zhengxiangling.com/17-8-3/21173131.jpg)

配置完成之后也需要等待一段时间，正确配置之后就会显示发布成功啦~

然后就可以在之前创建的对象存储中上传需要加速的资源了

![](https://cdn.zhengxiangling.com/17-8-3/48429336.jpg)

然后修改默认域名，改成自己的域名
![](https://cdn.zhengxiangling.com/17-8-3/72760733.jpg)

点击小眼睛就可以查看加速资源的路径了，就可以拿到需要引用的地方引用加速资源了

#### 使用极简图床绑定七牛云上传图片

极简图床支持截图粘贴、拖拽图片上传，其中截图粘贴上传时我最喜欢的一点原因，同时支持Chrome插件。
![](https://cdn.zhengxiangling.com/17-8-4/82594167.jpg)

![](https://cdn.zhengxiangling.com/17-8-4/29476783.jpg)

使用方式很简单，点击注册登录，

点击右上角的设置按钮

然后配置 空间名称，AK，SK，和域名即可。

![](https://cdn.zhengxiangling.com/17-8-4/17778041.jpg)