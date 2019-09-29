title: 网站服务状态及ssl证书状态在线检测工具
author: Elijah Zheng
tags:
  - webpack
  - vue
  - tornado
  - ssl
  - python
  - mongo
categories: []
date: 2018-03-17 15:19:00
---
为了检测多个网站及检测网站ssl证书有效性，写了一个在线测试检测工具。
<!--more-->

##### 实现功能
自由添加和编辑站点信息 	
单选及多选检测 	
支持http和https检测    	
当有ssl证书时，检测证书的有效状态以及过期时间   	 
查看检测历史 	


##### 环境    
python3 node-8.9.1 


##### 框架及数据库
后端：``tornado `` 	
前端：``vue``	
数据库：``mongo``

##### http检测状态实现原理
通过 python ``requests`` 库
```python
import requests

try:
    web_url = 'http://...'
    # timeout - 请求超时时间
    r = requests.get(web_url, timeout=10)
    print('请求成功')
    ...
    return True;

except Exception as e:

    # 输出错误原因
    print(e)
    return False;
```

##### https 检测状态及ssl证书状态测试实现原理
运用 ``socket`` 库建立连接，``ssl``库获取ssl证书的公钥，在通过``openssl``的``crypto``通过公钥解密获取ssl证书里的信息。
```python
import ssl
import socket
from OpenSSL import crypto

web_url = 'https://...'
# ssl.PROTOCOL_TLS - ssl支持协议
context = ssl.SSLContext(ssl.PROTOCOL_TLS)
# 建立 socket 连接   
# socket.AF_INET 为 IPv4 网络协议的套接字类型
sock = socket.socket(socket.AF_INET)
# 设置超时时间 10s
sock.settimeout(10)
wrappedSocket = context.wrap_socket(socket.socket(socket.AF_INET), server_hostname=web_url

try:
    wrappedSocket.connect(web_url, 443))
    # 获取 ssl 证书公钥
    pem_cert = ssl.DER_cert_to_PEM_cert(wrappedSocket.getpeercert(True))
    wrappedSocket.close()
    # 解密公钥
    io_cert = crypto.load_certificate(crypto.FILETYPE_PEM, pem_cert)
    # 证书过期时间
    ssl_time = io_cert.get_notAfter().decode()[:-1]
    ssl_not_after = ssl_time[0:4] + '年' + ssl_time[4:6] + '月' + ssl_time[6:8] + '日' + ssl_time[8:10] + '时' + ssl_time[10:12] + '分' + ssl_time[12:14] + '秒'
    # 证书有效状态
    ssl_expired = io_cert.has_expired()
    
    return True;

except Exception as e:

    # 连接失败，输出错误原因
    print(e)
    return False;
```

##### 遇到问题
1.``webpack``打包时遇到
```bash 
You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build. 
```

解决办法，在``webpack``配置中增加
```js
resolve: {
    alias: {
        vue: 'vue/dist/vue.js',
    }
}
```

##### 资料		

> ssl — TLS/SSL wrapper for socket objects - https://docs.python.org/3.2/library/ssl.html#ssl.SSLSocket.getpeercert