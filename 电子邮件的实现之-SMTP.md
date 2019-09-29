title: 电子邮件的实现之 SMTP（Java版）
author: Elijah Zheng
tags:
  - 电子邮件
  - SMTP
  - JAVA
categories: []
date: 2017-11-22 23:24:00
---
1.SMTP协议简介

SMTP（Simple Mail Transfer Protocol）即简单邮件传输协议，目标是向用户提供高效、可靠的邮件传输。它的一个重要特点是它能够在传送中接力传送邮件，即邮件可以通过不同网络上的主机接力式传送。通常它工作在两种情况下：一是邮件从客户机传输到服务器；二是从某一个服务器传输到另一个服务器。SMTP 是一个请求/响应协议，它监听 25 号端口，用于接收用户的 Mail 请求，并与远端 Mail 服务器建立 SMTP 连接。
<!--more-->

2.SMTP协议工作机制

SMTP通常有两种工作模式。发送SMTP和接收SMTP。具体工作方式为：发送SMTP在接收到用户的邮件请求后，判断此邮件是否为本地邮件，若是直接投送到用户的邮箱，否则向DNS查询远端邮件服务器的MX记录，并建立与远端接收SMTP之间的一个双向传送通道，此后SMTP命令由发送SMTP发出，由接收SMTP接收，而应答则反方向传送。一旦传送通道建立，SMTP发送者发送MAIL命令指明邮件发送者。如果SMTP接收者可以接收邮件则返回OK应答。SMTP发送者再发出RCPT命令确认邮件是否接收到。如果SMTP接收者接收，则返回OK应答；如果不能接收到，则发出拒绝接收应答（但不中止整个邮件操作），双方将如此反复多次。当接收者收到全部邮件后会接收到特别的序列，入伏哦接收者成功处理了邮件，则返回OK应答。

3.SMTP的连接和发送过程
（a）建立TCP连接
（b）客户端发送HELO命令以标识发件人自己的身份，然后客户端发送MAIL命令；服务器端正希望以OK作为响应，表明准备接收。
（c）客户端发送RCPT命令，以标识该电子邮件的计划接收人，可以有多个RCPT行；服务器端则表示是否愿意为收件人接收邮件
（d）协商结束，发送邮件，用命令DATA发送
（e）以.表示结束输入内容一起发送出去
（f）结束此次发送，用QUIT命令退出。

4.准备工作	
1）注册新浪邮箱	
2）准备包文件 		
activation-1.1.jar 以及 javax.mail-1.4.4.jar	
下载链接	
[activation-1.1.jar](https://cdn.zhengxiangling.com/activation-1.1.jar) 
[javax.mail-1.4.4.jar](https://cdn.zhengxiangling.com/activation-1.1.jar)	
3)导入包文件	

在项目中新建``lib``文件夹，将两个包文件放入文件夹中
![](https://cdn.zhengxiangling.com/17-11-23/10495710.jpg)

右击包文件，选择 Build Path -> Configure Build Path，然后出现下图
![](https://cdn.zhengxiangling.com/17-11-23/6832403.jpg)
点击 Libralies -> Add JARS -> 选中``lib``文件夹中的 ``.jar``文件，重复步骤导入两个包文件

5.代码

```js
package SMTP;
import javax.mail.Address;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import com.sun.mail.util.MailSSLSocketFactory;
import java.security.GeneralSecurityException;
import java.util.Date;
import java.util.Properties;
public class SMTP {  
	public static void main(String[] args) throws MessagingException,
			GeneralSecurityException {
		Properties props = new Properties();
		// 开启debug调试
		props.setProperty("mail.debug", "true");
		// 发送服务器需要身份验证
		props.setProperty("mail.smtp.auth", "true");
		// 设置邮件服务器主机名
		props.setProperty("mail.host", "smtp.sina.cn");
		// 发送邮件协议名称
		props.setProperty("mail.transport.protocol", "smtp");
		Session session = Session.getInstance(props);
		Message msg = new MimeMessage(session);
		msg.setSubject("STMP 测试");
		StringBuilder builder = new StringBuilder();
	
		builder.append("My name is Elijah");
		msg.setText(builder.toString());
		msg.setFrom(new InternetAddress("your_number@sina.cn"));
		Transport transport = session.getTransport();
		transport.connect("smtp.sina.cn", "xxx@sina.cn", "your_password");
		transport.sendMessage(msg, new Address[] { new InternetAddress("xxx@sina.cn") });
		transport.close();
	}  
} 
```

参数分析

 SMTP的响应，它的一般形式是：XXX  Readable Illustration。XXX是三位十进制数；Readable Illustration是可读的解释说明，用来表明命令是否成功等。XXX具有如下的规律：以2开头的表示成功，以4和5开头的表示失败，以3开头的表示未完成（进行中）
 
 C: client S: server
 
域服务器准备好了，之后的为域服务器自动返回的信息

HELO 客户端为标识自己的身份而发送的命令（通常带域名）

250 请求动作完成

AUTH LOGIN 请求认证

等待用户输入验证信息

dxNlcm5hbWU6  服务器的响应——经过base64编码了的“Username”= 

334 UGFzc3dvcmQ6  经过BASE64编码了的"Password:"= 

235 auth successfully 认证成功   

MAIL FROM: 发送者邮箱

RCPT TO:  接收者邮箱 

DATA  请求发送数据  

开始邮件输入，以.结束

<CR><LF> 回车 + 换行

QUIT 终止会话

服务器关闭