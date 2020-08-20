# 基于Spring Boot快速实现发送邮件功能

[https://mp.weixin.qq.com/s/j2hOU3jSv2UUQiY9gukHkw](https://mp.weixin.qq.com/s/j2hOU3jSv2UUQiY9gukHkw)

邮件在项目中经常会被用到，比如用邮件发送通知。比如，通过邮件注册、认证、找回密码、系统报警通知、报表信息等。本篇文章带大家通过SpringBoot快速实现一个发送邮件的功能。

#### 邮件协议

下面先简单了解一下常见的邮件协议。常用的电子邮件协议有SMTP、POP3、IMAP4，它们都隶属于TCP/IP协议簇，默认状态下，分别通过TCP端口25、110和143建立连接。

**SMTP协议**

SMTP的全称是 “Simple Mail Transfer Protocol”，即简单邮件传输协议。它是一组用于从源地址到目的地址传输邮件的规范，通过它来控制邮件的中转方式。它的一个重要特点是它能够在传送中接力传送邮件，即邮件可以通过不同网络上的主机接力式传送。

SMTP认证，简单地说就是要求必须在提供了账户名和密码之后才可以登录SMTP服务器，这就使得那些垃圾邮件的散播者无可乘之机。增加SMTP认证的目的是为了使用户避免受到垃圾邮件的侵扰。SMTP已是事实上的E-Mail传输的标准。

**POP协议**

POP邮局协议负责从邮件服务器中检索电子邮件。它要求邮件服务器完成下面几种任务之一：从邮件服务器中检索邮件并从服务器中删除这个邮件；从邮件服务器中检索邮件但不删除它；不检索邮件，只是询问是否有新邮件到达。

POP协议支持多用户互联网邮件扩展，后者允许用户在电子邮件上附带二进制文件，如文字处理文件和电子表格文件等，实际上这样就可以传输任何格式的文件了，包括图片和声音文件等。在用户阅读邮件时，POP命令所有的邮件信息立即下载到用户的计算机上，不在服务器上保留。

POP3\(Post Office Protocol 3\)即邮局协议的第3个版本,是因特网电子邮件的第一个离线协议标准。

**IMAP协议**

互联网信息访问协议（IMAP）是一种优于POP的新协议。和POP一样，IMAP也能下载邮件、从服务器中删除邮件或询问是否有新邮件，但IMAP克服了POP的一些缺点。例如，它可以决定客户机请求邮件服务器提交所收到邮件的方式，请求邮件服务器只下载所选中的邮件而不是全部邮件。客户机可先阅读邮件信息的标题和发送者的名字再决定是否下载这个邮件。

通过用户的客户机电子邮件程序，IMAP可让用户在服务器上创建并管理邮件文件夹或邮箱、删除邮件、查询某封信的一部分或全部内容，完成所有这些工作时都不需要把邮件从服务器下载到用户的个人计算机上。

支持IMAP的常用邮件客户端有：ThunderMail,Foxmail,Microsoft Outlook等。

#### SpringBoot集成Mail功能

如果未使用SpringBoot，需要自己去封装消息体等信息，实现起来还是比较复杂的。但基于Spring Boot进行邮件发送，几乎可以说只用引入spring-boot-starter-mail就可以轻松完成邮件的发送。

从本质上来说是由于Spring推出了关于Mail的JavaMailSender类，基于该类Spring Boot又进一步封装，从而实现了轻松发送邮件的集成。而且JavaMailSender类提供了强大的邮件发送能力，支持各种类型的邮件发送。

**依赖配置**

集成步骤非常简单，在项目中添加如下依赖：

```text
<dependency> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

**配置文件**

在application.properties中添加邮箱配置，对应参数项已经内置好，根据具体情况进行配置即可。

```text
# 邮箱服务器地址
spring.mail.host=smtp.qq.com
spring.mail.username=admin@choupangxia.com
spring.mail.password=123456
spring.mail.default-encoding=UTF-8
```

其中第一个host（邮件服务器地址）参数，不同的邮箱有所不同，上面是QQ邮箱的host。163邮箱为smtp.163.com、126邮箱为smtp.126.com。

username和password项为邮箱对应的用户名和密码，密码并不是登录密码，而是开启POP3之后设置的客户端授权密码。

以QQ邮箱为例，进行密码的配置和获取。首先登录QQ邮箱，找“设置”，“账户”。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/20/640-20200820104510542-104511.jpg)

在下面找到“POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务”，选择第二项中的“IMAP/SMTP服务”，进行开启。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/20/640-20200820104510803-104511.jpg)

开启成功，会显示如下页面：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/20/640-20200820104511827-104511.jpg)

页面中便包含了授权码，也就是我们项目中的密码。此时将该授权码复制到password处即可。

**发送文本邮件**

完成了上面的配置，发送功能的实现便极其简单了，直接在项目中注入JavaMailSender然后调用其send方法便可进行邮件的发送。

以单元测试的形式发送邮件如下：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class MailTests {

  @Resource
  private JavaMailSender javaMailSender;

  @Test
  public void sendMail() {
    SimpleMailMessage msg = new SimpleMailMessage();
    msg.setFrom("123@qq.com");
    msg.setTo("123@qq.com");
    msg.setSubject("程序新视界");
    msg.setText("技术分享");
    javaMailSender.send(msg);
  }
}
```

程序很简单，创建一个SimpleMailMessage对象，设置从哪个（from）账号发送到（to）哪个账号，邮件的标题（subject）和内容（text）分别是什么。

执行单元测试，稍等片刻，邮箱便收到了邮件。如果执行的过程中出现权限验证相关的异常，则需要检查一下相关的配置是否正确。

如果需要抄送其他人，通过如下格式新增一个或多个收件地址。

```text
// 抄送邮箱
msg.setCc("abc@126.com","def@126.com");
```

**发送富文本邮件**

正常来说，我们的邮件会有不同的格式，使用上面的SimpleMailMessage不能够很好的丰富邮件内容，也不支持html的解析。

Spring Boot支持使用HTML发送邮件是通过MimeMessage来完成的。看具体的示例代码：

```text
@Test
public void sendHtmlMail() {
  String content="<html>\n" +
      "<body>\n" +
      "    <h3>hello world ! 这是一封html邮件!</h3>\n" +
      "</body>\n" +
      "</html>";

  MimeMessage message = javaMailSender.createMimeMessage();
  try {
    // 第二个参数true表示需要创建一个multipart message
    MimeMessageHelper helper = new MimeMessageHelper(message, true);
    helper.setFrom("123@qq.com");
    helper.setTo("123@qq.com");
    helper.setSubject("程序新视界");
    helper.setText(content, true);

    javaMailSender.send(message);
  } catch (MessagingException e){
    System.out.println("发送邮件异常");
  }
}
```

此处使用了MimeMessageHelper来设置对应的参数信息，但在调用MimeMessageHelper对应的setter方法时会抛出MessagingException异常，需要进行特殊处理。

上面的content的内容，如果使用SimpleMailMessage对象进行发送，邮件的内容是包含html标签的内容，而不是直接呈现html标签所需要展示的格式。

MimeMessageHelper支持发送复杂邮件模板，支持文本、附件、HTML、图片等。比如需要发送附件，则在上面的代码中通过调用helper的addAttachment\(fileName, file\)方法即可。

我们这里就不再拓展其他功能，大家可自行进行尝试。

#### 其他扩展

上面只是通过单元测试的形式展示了基于Spring Boot发送邮件，当然，在生产环境中的应用场景要比上面的复杂的多。比如，要考虑邮件模板、对外接口、异常处理、成功率等问题。大家可在此基础上进行拓展。

源码地址：[https://github.com/secbr/springboot-learn/tree/master/springboot-mail](https://github.com/secbr/springboot-learn/tree/master/springboot-mail)

最后，本人关于Spring Boot的书已出版，大家多多支持。

