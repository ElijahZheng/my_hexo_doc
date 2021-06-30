title: Django Admin 使用富文本插件 Quill 定制后台
author: Elijah Zheng
tags:
  - Django
categories:
  - 爬坑
date: 2017-08-25 17:22:00
---
#### 为什么选择 Quill

自从网站的出现，内容创作已经成为了核心。文本域`textarea`对几乎所有的网站应用提供了一个基础的解决方法。但是在某些情况下你可能需要对文本输入做一些格式设置。这就出现了富文本编辑器。现在拥有很多的解决途径，但是 Quill 会给你一些 Modern Ideas 来考虑去使用它。

<!-- more -->

图例

![](https://cdn.zhengxiangling.com/17-8-25/16063078.jpg)

##### 在admin.py中注册你需要增加富文本插件的表

``` python
from django.contrib import admin
from app import models

@admin.register(models.YourTable)
class YourTableAdmin(admin.ModelAdmin):
    search_fields = ('name', 'desc', 'type__name')
    list_display = ('name', 'desc', 'type')
    ordering = ('-id', )

    class Media:
        js = (
            'http://cdn.quilljs.com/1.2.4/quill.min.js',
            '静态路径/my_admin.js',
        )
        css = {
            'all': (
                'http://cdn.quilljs.com/1.2.4/quill.snow.css',
            )
        }
```
![](https://cdn.zhengxiangling.com/17-8-25/85990711.jpg)

|    要点     | 解释           
|: ------------- :|:-------------:|
| @admin.register()      | 括号里填你要运用富文本的数据表 |
| search_fields| 加入搜索功能，括号里填表中拥有的字段，搜索内容会从填入字段中进行查询      | 
| list_display | 填入表中拥有的字段，定义在admin中显示的字段信息      |
| ordering      | 排序关键字，-id代表按id倒序排列，也就是最新的会排在上面 |
| class Media      | 引入外部资源，引入我们需要用到的Quill的css和js文件，其中 静态路径/my_admin.js是我们自己新建的js文件，用来自定义Quill所需要的样式 |

##### 在my_admin.js 中定义Quill样式及富文本要显示的位置

```js
django.jQuery(function() {
    var $ = django.jQuery

    $('#id_desc').css({
        'opacity': '0',
        'height': '1em',
    }).after('<div id="descEditor" style="height: 600px"></div>')

    var toolbarOptions = [
        ['bold', 'italic', 'underline', 'strike'], // toggled buttons
        ['blockquote', 'code-block'],

        [{ 'list': 'ordered' }, { 'list': 'bullet' }],
        [{ 'indent': '-1' }, { 'indent': '+1' }], // outdent/indent

        [{ 'header': [1, 2, 3, 4, 5, 6, false] }],

        [{ 'color': [] }], // dropdown with defaults from theme
        [{ 'font': [] }],
        [{ 'align': [] }],
        ['clean'], // remove formatting button
        ['image']
    ]

    var editor = new Quill('#descEditor', {
        modules: {
            toolbar: {
                container: toolbarOptions,
            }
        },
        theme: 'snow',
    });

    var $textarea = document.querySelector('#id_desc')
    editor.on('text-change', function(delta, oldDelta, source) {
        $textarea.value = editor.root.innerHTML
    })
    editor.root.innerHTML = $textarea.value
})
```

|    要点     | 解释           
|------------- |:-------------:|
| var $ = django.jQuery      | Django自带jquery，赋值给$比较符合平常使用规范 |
| $('#id_desc')| 选中需要运用富文本元素的id（id为admin自己生成），所有id需要自行修改    | 
|id="descEditor"|id="descEditor"为需要插入的富文本插件的id，可自行修改  |
| editor.on('text-change') | 官方api，当富文本的内容改变时，调用该接口      |

editor.on()的作用是，将富文本编辑好的源代码保存在原来的``textarea``中，这样点击保存修改时才会将富文本的源代码存回到数据库中。

##### 点击上传图片按钮时，将图片保存到 七牛云

首先，我们要获取token值	

>> token值是什么，可以参考我的文章 -- [DJANGO 运用 七牛云 作为图床上传图片](https://blog.zhengxiangling.com/Django-%E4%B8%8A%E4%BC%A0%E4%B8%83%E7%89%9B%E5%9B%BE%E7%89%87/)
定义获取token值的api

urls.py

``` python
url(r'^api/qntoken$', views.api_qn_token),
```

views.py中
``` python
def api_qn_token(request):

    data = {
        'qntoken': qn.upload_token('对象存储名')
    }
    return HttpResponse(json.dumps(data), content_type='application/json')
```

然后再my_admin.js中增加
``` js
django.jQuery(function() {
    var $ = django.jQuery
	...
    var editor = new Quill('#descEditor', {
        modules: {
            toolbar: {
                container: toolbarOptions,
                handlers: {
                    image: function() {
                        $('#qnUpload').click()
                    }
                }
            }
        },
        theme: 'snow',
    });

    $.get('/api/qntoken', function(data){
        $('body').append('<form id="qn" method="post" action="http://upload.qiniu.com/" enctype="multipart/form-data">' +
            '<input name="token" type="hidden" value="'+ data.qntoken +'">' +
            '<input id="qnUpload" name="file" type="file" style="display: none"/>' +
        '</form>')

        $('#qnUpload').change(function(){
            $.ajax({                        
                url: 'http://up-z2.qiniu.com/',
                type: 'POST',
                data: new FormData($('#qn')[0]),
                cache: false,
                contentType: false,
                processData: false,
            }).done(function( data ) {
                var imgSrc = 'http://ov8c***.bkt.clouddn.com/' + data.hash
                var range = editor.getSelection();
                editor.insertEmbed(range.index, 'image', imgSrc, Quill.sources.USER);
            });
        })
    })
})
```

点击图片按钮，就可以将图片上传到 七牛云 了。


到此，富文本插件就完成了。