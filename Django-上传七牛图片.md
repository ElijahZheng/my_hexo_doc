title: Django 运用 七牛云 作为图床上传图片
author: Elijah Zheng
tags:
  - Django
categories: []
date: 2017-08-25 22:40:00
---
用 七牛云 作为图床，是一个不错的选择，有一定免费量的存储空间。本文介绍怎么使用 七牛云 来存储图片。

<!-- more -->

#### 1.注册七牛云个人账户

选择个人账户

#### 2.创建存储空间

注册成功在资源主页 点击对象存储 -->立即添加，存储名称 （按自己喜好），这里为了后面方便先起名为chuangyi（创意），存储区域默认，访问空间选择公开空间。--> 确定创建

#### 3.获取有用的信息。
1）秘钥
左边栏，进入个人中心，点击秘钥管理，然后会有公钥和密钥。	

|AccessKey|SecretKey
|: ------------- :|:-------------:|
|AK:a40Z71LFfbz_kDJnfOJflyI**|SK:JnfOJflyIOYpMy6eOnXMbsP**|

这里的公钥和密钥后面会有用。
2）测试域名
  找到存储空间列表，点击chuangyi，空间概览里右边有测试域名。
  我这里是 ``opjb0b***.bkt.clouddn.com`` 每个人的都会不一样。

#### 4.在django项目中引入七牛。
在有 manager.py的目录下，执行
```bash
pip install qiniu
```
或
```bash
easy_install qiniu
```
ps：如果受权限限制就用sudo执行。


#### 5.配置七牛
1)在cyzc的settings.py中

```python
import qiniu

qn = qiniu.Auth(
    'a40Z71LFfbz_kDJnfOJflyI**',
    'JnfOJflyIOYpMy6eOnXMbsP**'
)
```

（说明：Auth中填入的是之前说的公钥和秘钥。每个人的公钥和密钥都不一样，请自行替换）

2）运用七牛	
从views.py中传token值到页面中。	
引入qn	
views.py

```python
from cyzc.settings import qn

def qn_test(request):
	data = {
        'upload_token': qn.upload_token("chuangyi")
    }
    return data
```

将token值返回给html，因为用七牛上传图片时需要上传的权限，这个token值就是上传时需要的证明。
其中upload_token()中的值为对应的七牛上面创建的存储空间的名字。


3）在html页面给input绑定这个token值

``` html
{% if upload_token %}
<form id="qn" method="post" action="http://upload.qiniu.com/" enctype="multipart/form-data">
    <input name="token" type="hidden" value="{{ upload_token }}">
    <input id="qnUpload" name="file" type="file" accept="image/*" style="display: none">
</form>
{% endif %}
```
这个直接复制粘贴就可以了。

4）给需要上传图片的按钮增加class="upload-btn"

我们需要用到的是先创建一个隐藏的input，其中name对应着要上传的字段的名字。

```html
<input type="hidden" class="my_upload" name="img">
```

5）然后给按钮绑定一个点击事件。

``` js
$('.upload-btn').click(function () {
    $('#qnUpload').click();
});
```

其中upload-btn 为点击按钮的class名，qnUpload 为隐藏的七牛上传文件的input的Id名。
就相当于点击自定义按钮，实际上触发的是七牛的input的上传按钮。

6）当上传图片时，触发change事件。

``` js
$('#qnUpload').change(function() {
    if (this.value) {
        $.ajax({
            url: 'http://up-z2.qiniu.com/',
            type: 'POST',
            data: new FormData($('#qn')[0]),
            cache: false,
            contentType: false,
            processData: false
        }).done(function(data) {
            var img_src = 'http://opjb0b**.bkt.clouddn.com/' + data.hash;
           $(".my_upload").val(img_src); 
        });
    }
})
```
这里的也直接复制就好。（也可以自行修改class或id属性值）。
``` js
done(function(data) {

})
```
data为上传成功时返回的图片数据，data.hash则为图片的名称。

7）然后我们可以直接获取上传成功的图片路径然后渲染在html页面上。
图片的路径为
``` js
var img_src = 'http://opjb0***.bkt.clouddn.com/' + data.hash;
```
其中``http://~~``为之前说的测试域名（ps：拼接路径名时别忘记了 '/' ）。


#### 6. 需要上传的字段的信息，已经存储在input的value里了，post的时候就可以获取到图片的路径，然后就可以存储到数据库中了。