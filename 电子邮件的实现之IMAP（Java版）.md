title: 电子邮件的实现之IMAP（Java版）
author: Elijah Zheng
tags:
  - 电子邮件
  - IMAP
  - Java
categories: []
date: 2017-11-25 11:05:00
---
IMAP（Internet Mail Access Protocol，Internet邮件访问协议）以前称作交互邮件访问协议（Interactive Mail Access Protocol）。IMAP是斯坦福大学在1986年开发的一种邮件获取协议。它的主要作用是邮件客户端（例如MS Outlook Express)可以通过这种协议从邮件服务器上获取邮件的信息，下载邮件等。当前的权威定义是RFC3501。IMAP协议运行在TCP/IP协议之上，使用的端口是143。它与POP3协议的主要区别是用户可以不用把所有的邮件全部下载，可以通过客户端直接对服务器上的邮件进行操作。

<!--more-->

与POP3协议类似，IMAP（Internet消息访问协议）也是提供面向用户的邮件收取服务。常用的版本是IMAP4。
IMAP4改进了POP3的不足，用户可以通过浏览信件头来决定是否收取、删除和检索邮件的特定部分，还可以在服务器上创建或更改文件夹或邮箱。它除了支持POP3协议的脱机操作模式外，还支持联机操作和断连接操作。它为用户提供了有选择的从邮件服务器接收邮件的功能、基于服务器的信息处理功能和共享信箱功能。IMAP4的脱机模式不同于POP3，它不会自动删除在邮件服务器上已取出的邮件，其联机模式和断连接模式也是将邮件服务器作为“远程文件服务器”进行访问，更加灵活方便。IMAP4支持多个邮箱。
IMAP4的这些特性非常适合在不同的计算机或终端之间操作邮件的用户（例如你可以在手机、PAD、PC上的邮件代理程序操作同一个邮箱），以及那些同时使用多个邮箱的用户。

各参数返回的邮件信息：

参数| 代表信息
----|----
ALL|只返回按照一定格式的邮件摘要，包括邮件标志、RFC822.SIZE、自身的时间和信封信息。IMAP客户机能够将标准邮件解析成这些信息并显示出来。
BODY|只返回邮件体文本格式和大小的摘要信息。IMAP客户机可以识别这些细腻，并向用户显示详细的关于邮件的信息。其实是一些非扩展的BODYSTRUCTURE的信息。
FAST|只返回邮件的一些摘要，包括邮件标志、RFC822.SIZE、和自身的时间。
FULL|同样的还是一些摘要信息，包括邮件标志、RFC822.SIZE、自身的时间和BODYSTRUCTURE的信息。
BODYSTRUCTUR| 是邮件的[MIME-IMB]的体结构。这是服务器通过解析[RFC-2822]头中的[MIME-IMB]各字段和[MIME-IMB]头信息得出来 的。包括的内容有：邮件正文的类型、字符集、编码方式等和各附件的类型、字符集、编码方式、文件名称等等。
ENVELOPE|信息的信封结构。是服务器通过解析[RFC-2822]头中的[MIME-IMB]各字段得出来的，默认各字段都是需要的。主要包括：自身的时间、附件数、收件人、发件人等。
FLAGS|此邮件的标志。
INTERNALDATE|自身的时间。
RFC822.SIZE|邮件的[RFC-2822]大小
RFC822.HEADER|在功能上等同于BODY.PEEK[HEADER]，
RFC822|功能上等同于BODY[]。
RFC822.TEXT|功能上等同于BODY[TEXT]
UID|返回邮件的UID号，UID号是唯一标识邮件的一个号码。
BODY[section] ``<<partial>>``|返回邮件的中的某一指定部分，返回的部分用section来表示，section部分包含的信息通常是 代表某一部分的一个数字或者是下面的某一个部分：HEADER, HEADER.FIELDS, HEADER.FIELDS.NOT, MIME, and TEXT。如果section部分是空的话，那就代表返回全部的信息，包括头信息。
BODY[HEADER]|返回完整的文件头信息。
BODY[HEADER.FIELDS ()]|在小括号里面可以指定返回的特定字段。
BODY[HEADER.FIELDS.NOT ()]|在小括号里面可以指定不需要返回的特定字段。
BODY[MIME]|返回邮件的[MIME-IMB]的头信息，在正常情况下跟BODY[HEADER]没有区别。
BODY[TEXT]|返回整个邮件体，这里的邮件体并不包括邮件头。


代码
```js
package IMAP;

import java.util.Properties;

import javax.mail.Folder;
import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Store;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeUtility;

import POP3.POP3;

import com.sun.mail.imap.IMAPMessage;

public class IMAP {

	public static void main(String[] args) throws Exception {
		// 准备连接服务器的会话信息
		Properties props = new Properties();
		props.setProperty("mail.store.protocol", "imap");
		props.setProperty("mail.imap.host", "imap.sina.cn");
		props.setProperty("mail.imap.port", "143");

		// 创建Session实例对象
		Session session = Session.getInstance(props);

		// 创建IMAP协议的Store对象
		Store store = session.getStore("imap");

		// 连接邮件服务器
		store.connect("your_name@sina.cn", "your_password");

		// 获得收件箱
		Folder folder = store.getFolder("INBOX");
		// 以读写模式打开收件箱
		folder.open(Folder.READ_WRITE);

		// 获得收件箱的邮件列表
		Message[] messages = folder.getMessages();

		// 打印不同状态的邮件数量
		System.out.println("收件箱中共" + messages.length + "封邮件!");
		System.out.println("收件箱中共" + folder.getUnreadMessageCount() + "封未读邮件!");
		System.out.println("收件箱中共" + folder.getNewMessageCount() + "封新邮件!");
		System.out.println("收件箱中共" + folder.getDeletedMessageCount()
				+ "封已删除邮件!");

		System.out
				.println("------------------------开始解析邮件----------------------------------");

		// 解析邮件
		for (int i = 0, count = 2; i < count; i++) {
			MimeMessage msg = (MimeMessage) messages[i];
			String subject = MimeUtility.decodeText(msg.getSubject());
			System.out.println("[" + subject + "]未读，是否需要阅读此邮件（yes/no）？");
			POP3.parseMessage(msg); // 解析邮件

		}
		// 关闭资源
		folder.close(false);
		store.close();
	}
}
```

参数分析

版本4rev1（IMAP4rev1）允许一个客户端访问和操作在一个服务器上的电子邮件。
SELECT INBOX： 选择收件箱	
EXISTS： 3封存在	
FETCH`` <mail id><datanames>``	
ENVELOPE：信息的信封结构。	
INTERNALDATE：自身的时间。	
RFC822.SIZE：邮件的[RFC-2822]大小。	
BODYSTRUCTUR： 是邮件的[MIME-IMB]的体结构。这是服务器通过解析[RFC-2822]头中的[MIME-IMB]各字段和[MIME-IMB]头信息得出来 的。包括的内容有：邮件正文的类型、字符集、编码方式等。	
BODY[TEXT]：返回整个邮件体，这里的邮件体并不包括邮件头。	
FLAGS：此邮件的标志。	
EXAMINE：以只读方式打开邮箱，参数是需要打开的邮箱的名字，使用EXAMINE命令打开的邮箱不允许对邮件进行改动，因此不能增加或删除邮件的标志。	
CLOSE：表示Client结束对当前Folder（文件夹/邮箱）的访问，关闭邮箱该邮箱中所有标志为、DELETED的邮件就被从物理上删除。CLOSE没有命令参数。随后可以SELECT另一Folder。	
LOGOUT：结束本次IMAP会话。	