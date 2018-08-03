---
layout: post
title: 'SpringBoot 发送邮件'
date: 2018-07-17
author: yunfei
tags: SpringBoot Mail
---

### 1、添加依赖
````xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-mail</artifactId>
   </dependency>
````
 

### 2、添加配置
````yaml
# 设置邮箱主机
spring.mail.host= smtp.qq.com
# 设置用户名,需要注意的是username是登陆邮箱是的用户名而不是邮箱，password是登陆邮箱的密码
spring.mail.username=邮箱登录用户名
# 设置密码
spring.mail.password=登录密码
# 设置是否需要认证，如果为true,那么用户名和密码就必须的，
#如果设置false，可以不设置用户名和密码，当然也得看你的对接的平台是否支持无密码进行访问的。
spring.mail.properties.mail.smtp.auth=ture
# STARTTLS[1]  是对纯文本通信协议的扩展。它提供一种方式将纯文本连接升级为加密连接（TLS或SSL），而不是另外使用一个端口作加密通信。
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
````
 

### 3、测试代码       
````java
   public class mail{
              /**
              * 修改application.properties的用户，才能发送。
              */
     
             @Test
             @Ignore
             public void sendSimpleEmail() {
                       SimpleMailMessage message = new SimpleMailMessage();
                       message.setFrom("xxxx@126.com ");// 发送者.
                       message.setTo("xxxx@126.com");// 接收者.
                       String[] ccList = new String[] { "xxxx@126.com", "yang.junming@xxxx.com" };
                       // 这里添加抄送人名称列表
                       message.setCc(ccList);
                       String[] bccList = new String[] { "yyyy@126.com", "yjmyzz@xxxx.com" };//
                       // 这里添加密送人名称列表
                       message.setBcc(bccList);
                       message.setSubject("测试邮件（邮件主题）");// 邮件主题.
                       message.setText("这是邮件内容");// 邮件内容.
                       mailSender.send(message);// 发送邮件
             }
     
             /**
              * 测试发送附件.(这里发送图片.)
              *
              * @throws MessagingException
              */
     
             @Test
             @Ignore
             public void sendAttachmentsEmail() throws MessagingException {
                       // 这个是javax.mail.internet.MimeMessage下的，不要搞错了。
                       MimeMessage mimeMessage = mailSender.createMimeMessage();
                       MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true, "UTF-8");
                       // 基本设置.
                       helper.setFrom("xxxx@126.com ");// 发送者.
                       helper.setTo(new String[] { " xxxx@126.com ", " xxxx@163.com " });// 接收者.
     
                       // helper.setTo("1354737677@163.com");// 接收者.
                       helper.setSubject("测试附件（邮件主题）");// 邮件主题.
                       helper.setText("这是邮件内容（有附件哦.）");// 邮件内容.
                       // org.springframework.core.io.FileSystemResource下的:
                       // 附件1,获取文件对象.
                       FileSystemResource file1 = new FileSystemResource(new File("E:/文档/图片/git.png"));
                       // 添加附件，这里第一个参数是在邮件中显示的名称，也可以直接是head.jpg，但是一定要有文件后缀，不然就无法显示图片了。
                       helper.addAttachment("git.png", file1);
                       // 附件2
                       // FileSystemResource file2 = new FileSystemResource(
                       // new File("E:/spring boot/Spring Boot中使用Swagger2构建强大的RESTful
                       // API文档.docx"));
                       File file2 = new File("E:/技术/spring boot/Test邮件附件测试.docx");
                       String filename = null;
                       try {
                                filename = MimeUtility.encodeWord(file2.getName());
                                // filename = MimeUtility.encodeText(file2.getName());
                       } catch (UnsupportedEncodingException e1) {
                                e1.printStackTrace();
                       }
     
                       byte bytes[] = { (byte) 0xC2, (byte) 0xA0 };
                       String UTFSpace = null;
                       try {
                                UTFSpace = new String(bytes, "utf-8");
                       } catch (UnsupportedEncodingException e) {
                                // TODO Auto-generated catch block
                                e.printStackTrace();
                       }
                       filename = filename.replaceAll(UTFSpace, "&nbsp;");
                       filename = filename.replaceAll("\r", "").replaceAll("\n", "");
                       System.out.println("fileName>" + file2.getName());
                       helper.addAttachment(filename, file2);
                       // 邮件正文显示图片 第二个参数指定发送的是HTML格式,同时cid:是固定的写法
                       helper.setText("<body>这是图片：<img src='cid:head' /></body>", true);
                       FileSystemResource file = new FileSystemResource(new File("E:/文档/图片/spring.png"));
                       helper.addInline("head", file);
                       mailSender.send(mimeMessage);
             }
     
             
             /**
              * 邮件中使用静态资源.
              *
              * @throws Exception
              */
     
             @Test
             @Ignore
             public void sendInlineMail() throws Exception {
                       MimeMessage mimeMessage = mailSender.createMimeMessage();
                       MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);
                       // 基本设置.
                       helper.setFrom("xxxx@126.com ");// 发送者.
                       helper.setTo("xxxx@126.com ");// 接收者.
                       helper.setSubject("测试静态资源（邮件主题）");// 邮件主题.
                       // 邮件内容，第二个参数指定发送的是HTML格式
                       // 说明：嵌入图片<img src='cid:head'/>，其中cid:是固定的写法，而aaa是一个contentId。
                       helper.setText("<body>这是图片：<img src='cid:head' /></body>", true);
                       FileSystemResource file = new FileSystemResource(new File("E:/文档/图片/spring.png"));
                       helper.addInline("head", file);
                       mailSender.send(mimeMessage);
             }
   }
````


