title: 如何在邮件中写html
author: Elijah Zheng
date: 2017-12-21 18:43:31
tags:
---
当我们需要在邮件中显示想要的自己定义的格式的时候，就需要在 email 中编写 html，以下将列举一些制作过程中个人的一些总结。
<!--more-->

1.Doctype	
使用这个是为了兼容性，但是就意味着不能使用HTML5。
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">

<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Google Email</title>
    <meta name="viewport" content="width=device-width">
</head>

</html>
```

2.布局	
全局使用table进行布局
``` html
<body style="margin:0;padding:0">
    <div>
        <table width="680" align="center">
            <tbody>
                <tr>
                    <td width="680">
                        <table>
                        	<tbody></tbody>
                        </table>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</body>
```

3.样式写在标签里面，图片统一用外链引入
``` html
<img src="http://ou1bzq6p5.bkt.clouddn.com/icon_date@2x.png" alt="date: " style="width:35px;height:35px;vertical-align:middle">
```

4.不同邮件服务商对邮件的支持程度不同，注意兼容性问题。	
1)页面宽度请设定在600到800px（像素）以内，高度根据内容的需求调整高度。

2)html代码在15kb以内。	

3)单张图片尺寸不要超过1728px,否则outlook显示不全。

4)邮件内容图文比例4:6，否则容易进入垃圾箱。	

5)table标签上如果不需要边框和间距请加上 border="0" cellpadding="0" cellspacing="0"	

6)文字行高line-height、元素间距要定义在块状元素上（p、td、h、li等）才能起作用。	

7)尽量写行内样式。		

8)不要使用背景图片或gif动态图。

9)尽量不要用float、position来写邮件效果。

10)邮件中的按钮尽量不要用图片，可写个一行一列的表格，里面放个a标签。可能发到客户邮箱未被允许加载图片。	

11)img标签要给alt属性，再图片未加载的情况，这个提示的文字就会显示比较重要。	

12)一定要给p标签和h系列标签指定一个margin和padding（也可全都设置margin:0;padding:0;），不然不同的邮箱收到的邮件，间距不一致。font-size、font-weight也要指定，不然显示也不一致。

13)英文、数字不折行显示的话，给包裹的td加上word-break:break-all。

14)Outlook会自动为table cell 添加1px border，请在邮件顶部的内联样式中加上 table td { border-collapse: collapse; }	



5.当使用响应式布局时，参考地址
[Free Responsive Email Template](https://www.emailonacid.com/blog/article/email-development/emailology_a_free_responsive_email_template_using_media_queries_-_part_i/)	

1响应式设计可使用@media 媒体查询语句或百分比布局。

2)移动设备邮件模板宽度建议480px。

3)模板通常是1－3列式布局，推荐1-2列式。
