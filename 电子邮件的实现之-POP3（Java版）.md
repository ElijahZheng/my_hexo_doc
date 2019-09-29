title: 电子邮件的实现之 POP3（Java版）
author: Elijah Zheng
tags:
  - 电子邮件
  - Java
  - POP3
  - ''
categories: []
date: 2017-11-25 10:47:00
---
POP3，全名为“Post Office Protocol - Version 3”，即“邮局协议版本3”。是TCP/IP协议族中的一员，由RFC1939 定义。本协议主要用于支持使用客户端远程管理在服务器上的电子邮件。提供了SSL加密的POP3协议被称为POP3S。
POP 协议支持“离线”邮件处理。其具体过程是：邮件发送到服务器上，电子邮件客户端调用邮件客户机程序以连接服务器，并下载所有未阅读的电子邮件。这种离线访问模式是一种存储转发服务，将邮件从邮件服务器端送到个人终端机器上，一般是PC机或 MAC。一旦邮件发送到 PC 机或MAC上，邮件服务器上的邮件将会被删除。但目前的POP3邮件服务器大都可以“只下载邮件，服务器端并不删除”，也就是改进的POP3协议。
<!--more-->

POP3协议默认端口：110	
POP3协议默认传输协议：TCP	
POP3协议适用的构架结构：C/S	
POP3协议的访问模式：离线访问	

POP3命令码

命令 | 描述
----| ----
USER [username]|处理用户名
PASS [password]|处理用户密码
APOP [Name,Digest]|认可Digest是MD5消息摘要
STAT|处理请求服务器发回关于邮箱的统计资料，如邮件总数和总字节数
UIDL [Msg#]|处理返回邮件的唯一标识符，POP3会话的每个标识符都将是唯一的
LIST [Msg#]|处理返回邮件数量和每个邮件的大小
RETR [Msg#]|处理返回由参数标识的邮件的全部文本
DELE [Msg#]|处理服务器将由参数标识的邮件标记为删除，由quit命令执行
RSET|处理服务器将重置所有标记为删除的邮件，用于撤消DELE命令
TOP [Msg# n]|处理服务器将返回由参数标识的邮件前n行内容，n必须是正整数
NOOP|处理服务器返回一个肯定的响应
QUIT|终止会话

代码
```js
package POP3;
import java.io.IOException;
import java.util.Properties;
import javax.mail.BodyPart;
import javax.mail.Folder;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Multipart;
import javax.mail.Part;
import javax.mail.Session;
import javax.mail.Store;
import javax.mail.internet.MimeMessage;

public class POP3 {
	public static void main(String[] args) {
		String protocol = "pop3";// 使用pop3协议
		String host = "pop.sina.cn";// 163邮箱的pop3服务器
		int port = 110;// 端口
		String username = "your_name@sina.cn";// 用户名
		String password = "your_password";// 密码
		/*
		 * Properties是一个属性对象，用来创建Session对象
		 */
		Properties props = new Properties();
		props.put("mail.pop3.host", host);
		props.put("mail.pop3.port", port);
		/*
		 * Session类定义了一个基本的邮件对话。
		 */
		Session session = Session.getDefaultInstance(props);
		/*
		 * Store类实现特定邮件协议上的读、写、监视、查找等操作。 通过Store类可以访问Folder类。
		 * Folder类用于分级组织邮件，并提供照Message格式访问email的能力。
		 */
		Store store = null;
		Folder folder = null;
		try {
			store = session.getStore(protocol);
			store.connect(username, password);

			folder = store.getFolder("INBOX");
			folder.open(Folder.READ_ONLY);// 在这一步，收件箱所有邮件将被下载到本地
			// int size = folder.getMessageCount();// 获取邮件数目
			Message[] messages = folder.getMessages();
			// Message message = folder.getMessage(size);// 取得最新的那个邮件
			// 解析邮件内容
			for (int i = 0, count = 2; i < count; i++) {
				MimeMessage msg = (MimeMessage) messages[i];
				parseMessage(msg);
			}

		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if (folder != null) {
					folder.close(false);
				}
				if (store != null) {
					store.close();
				}
			} catch (Exception e) {
				e.printStackTrace();
			}
		}
		System.out.println("接收完毕！");
	}
	public static void getMailTextContent(Part part, StringBuffer content)
			throws MessagingException, IOException {
		// 如果是文本类型的附件，通过getContent方法可以取到文本内容，但这不是我们需要的结果，所以在这里要做判断
		boolean isContainTextAttach = part.getContentType().indexOf("name") > 0;
		if (part.isMimeType("text/*") && !isContainTextAttach) {
			content.append(part.getContent().toString());
		} else if (part.isMimeType("message/rfc822")) {
			getMailTextContent((Part) part.getContent(), content);
		} else if (part.isMimeType("multipart/*")) {
			Multipart multipart = (Multipart) part.getContent();
			int partCount = multipart.getCount();
			for (int i = 0; i < partCount; i++) {
				BodyPart bodyPart = multipart.getBodyPart(i);
				getMailTextContent(bodyPart, content);
			}
		}
	}

	public static void parseMessage(MimeMessage msg) throws MessagingException,
			IOException {
		// TODO Auto-generated method stub
		System.out.println("------------------解析第" + msg.getMessageNumber()
				+ "封邮件-------------------- ");

		String from = msg.getFrom()[0].toString();
		String subject = msg.getSubject();
		java.util.Date date = msg.getSentDate();

		System.out.println("From: " + from);
		System.out.println("Subject: " + subject);
		System.out.println("Date: " + date);
		StringBuffer content = new StringBuffer(30);
		getMailTextContent(msg, content);
		System.out.println("邮件正文："
				+ (content.length() > 100 ? content.substring(0, 100) + "..."
						: content));
		System.out.println("------------------第" + msg.getMessageNumber()
				+ "封邮件解析结束-------------------- ");
		System.out.println();
	}
}
```

参数分析

CAPA：开始与 POP3 Server 送出的第一个指令，用于取得此服务器的功能选项清单	
Capability list follows 返回指令	
USER：与 POP3 Server 送出帐户名	
PASS：与 POP3 Server 送出密码	
STAT：取得服务器上本帐户存在的信件数量	
NOOP：服务器返回一个肯定的响应，用于测试连接是否成功	
TOP n m：取得第n封信件前m行的内容	
TCP Spurious Retransmission：TCP虚假重传。 	
当抓到2次同一包数据时，wireshark判断网络发生了重传，同时，wireshark抓到初传包的反馈ack，因此wireshark判断初传包实际并没有丢失，因此称为虚假重传。	
IMF： Internet Message Format	
RETR n：取得第n封信件完整内容	
octet = 1 Byte =  8 bits	
QUIT 告知 POP3 服务器即将说再见．	