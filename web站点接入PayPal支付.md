title: web站点接入PayPal支付
author: Elijah Zheng
tags:
  - PayPal
  - 支付
categories: []
date: 2018-01-25 18:38:00
---
官方教程：
[PayPal Express Checkout](https://developer.paypal.com/docs/integration/direct/express-checkout/integration-jsv4/)

根据官方教程整理了一下具体步骤。	

<!--more-->
模板：	
``` js

<!DOCTYPE html>

<head>
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="https://www.paypalobjects.com/api/checkout.js"></script>
</head>

<body>
  <div id="paypal-button"></div>

  <script>
      paypal.Button.render({
          locale: 'zh_CN', // or en_US
          env: 'production', // or sandbox

          commit: true, // Show a 'Pay Now' button
          client: {
              sandbox:    '***',
              production: '***'
          },
          style: {
              size: 'small',
              color: 'silver',
              shape: 'pill',
              label: 'checkout',
              tagline: false
          },

          payment: function(data, actions) {
              return actions.payment.create({
                  payment: {
                      transactions: [
                          {
                              amount: { total: '填入金额', currency: 'USD' }
                          }
                      ]
                  }
              });
          },

          onAuthorize: function(data, actions) {
              return actions.payment.execute().then(function(payment) {
                  $.ajax({
                    type: 'POST',
                    url: '/',
                    data: {}
                  }).done(function (data) {
                      if (data == '0') {
                          alert('The payment is complete!');
                          window.location.reload();
                      }else {
                          alert('pay fail')
                      }
                  })
              });
          },

          onCancel: function(data, actions) {
              /*
               * Buyer cancelled the payment
               */
          },

          onError: function(err) {
              /*
              * An error occurred during the transaction
              */
          }
      }, '#paypal-button');
  </script>
</body>
```

##### 1. 必须 js
```js
<script src="https://www.paypalobjects.com/api/checkout.js"></script>
```

##### 2. 支付按钮
``<div id="paypal-button"></div>``，``paypal.Button.render``绑定对应的id(也可以是class)。

##### 3. render 参数	
1)``env``： 运行环境	
有两种：

类型|说明
----|----
sandbox| 沙盒，用于测试，用添加的sandbox账号测试能否交易成功
production|生产环境，部署上线时使用的环境

2)locale： 语言版本	
配套有多国语言，中文选用 ``zh_CN``，美式英文选用 ``en_US``

3）``client``： 客户端``id``	
获取方式：登录 -> [Applications](https://developer.paypal.com/developer/applications/)  -> 选择 REST API apps -> create App
![](https://cdn.zhengxiangling.com/18-1-25/637642.jpg)

创建成功后，可以从创建的应用获取``Sandbox`` 和 ``Live`` 的 ``client ID``，填入
```js
client: {
  	sandbox:    '',
  	production: ''
},
```

4)``style``： 定义支付按钮样式，参考 [Customize Checkout Button](https://developer.paypal.com/docs/integration/direct/express-checkout/integration-jsv4/customize-button/)

5)触发函数：

函数|说明
----|----
payment| 点击支付时触发，total填入需要支付的金额，currency填入支付的货币类型
onAuthorize|支付成功时触发，当支付成功时可以用Ajax提交数据修改订单支付状态为已支付。
onCancel|当用户关闭支付页面时触发
onError|当支付出错时触发

##### 4. 创建沙盒账号用于测试
[Sandbox accounts](https://developer.paypal.com/developer/accounts/)
创建两个账号，BUSSINESS 以及 PERSONAL。
创建完成后登录沙盒账号测试是否登录成功（红线按钮登录）
![](https://cdn.zhengxiangling.com/18-1-25/63582846.jpg)

##### 5. 使用sandbox 账号测试支付
当``env``环境为``sandbox``时，点击支付按钮时，使用``PERSONAL``账号来登录支付（测试账号默认有余额 $9999），当支付成功时会调用函数``onAuthorize``,可以弹窗``alert('pay success')``来测试是否支付成功。若成功，上线时将``env``转为``production``环境即可。