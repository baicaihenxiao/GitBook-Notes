# Jenkins安装并部署Java项目完整流程

[https://www.toutiao.com/i6774653293302907395/?group\_id=6774653293302907395](https://www.toutiao.com/i6774653293302907395/?group_id=6774653293302907395)



> 介绍Centos系统上安装Jenkins部署Spring Boot项目流程，并通过github Webhooks通知Jenkins代码更新信息并自动重新部署项目。

## 准备环境

### **JDK1.8**

```text
  yum install java-1.8.0-openjdk* -y
  java -version
```

能显示版本即安装成功，无需再配置环境

### **Git**

```text
  yum install git
```

能显示版本即安装成功，无需再配置环境

### **Maven**

个人的Maven安装目录是**/usr/bin/maven**：

```text
mkdir /usr/bin/maven
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar xzvf apache-maven-3.6.3-bin.tar.gz
cd /usr/bin/maven/apache-maven-3.6.3/conf
```

将settings.xml仓库更改为阿里云仓库并配置服务器仓库路径，如本地已有Maven配置文件，直接把本地文件上传到服务器并更改仓库路径即可\([推荐一款有理想的国产SSH工具-FinalShell](https://www.toutiao.com/i6774645023196578318/?group_id=6774645023196578318)，可直接拖到文件到指定文件夹\)

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/a0c92b1088134bddbfd1a20c711348b5-125840.jpeg)

配置Maven环境变量\(**vi /etc/profile**\)：

```text
export M2_HOME=/usr/bin/maven/apache-maven-3.6.3
PATH=$M2_HOME/bin:$PATH
```

保存退出,**source /etc/profile**重载环境变量，mvn -v能正确显示maven版本即配置成功

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/f9265915cbe748a8bb329ecc4a436c32-125840.jpeg)

### **Jenkins**

Jenkins安装命令：

```text
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install -y jenkins
```

修改配置文件**vi /etc/sysconfig/jenkins**并将端口号配置为8500:JENKINS\_PORT=“8500” jenkins启动相关指令:

```text
service jenkins start #启动
service jenkins restart #重启
service jenkins stop#停止
```

初始密码在**/var/lib/jenkins/secrets/initialAdminPassword**文件中，访问[http://ip:8500/](http://ip:8500/)安装推荐插件。

## Jenkins配置（Manage Jenkins）

### **System Configuration\(系统管理\)**

配置maven、git、email并Save，其中maven与git配置为必须，maven配置用于jenkins找到mvn指令位置，git用于jenkins从仓库拉取文件：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/8cccb6fad1754b769e04866f7ccd18a7-125840.jpeg)

### **Global Tool Configuration\(全局工具管理\)**

配置全局工具路径，主要配置JDK、Git、Maven并Save：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/c78190f95d6d4a04b78dd13b66f62a28-125841.jpeg)

注：java地址可通过**ls -lrt /etc/alternatives/java**查找，截取jre之前的路径。

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/8b12901c2c4f400cab6b7090abfd4188-125841.jpeg)

### **Manage Plugins\(插件管理\)**

如果需要构建Jenkins的Maven项目则需安装Maven Integration插件，但使用jenkins maven插件创建的项目执行mvn install后就会显示sucess，如需再执行指令设置构建unstable的话使用Freestyle项目也可。由于Freestyle的把控性更强且更自由，配置了maven环境后也可通过maven指令构建而不需再按照Jenkins Maven Integration插件，所以该文着重介绍Freestyle且更推荐使用Freestyle构建。

## Jenkins项目配置（New Item）

### **1. 配置项目github地址，git Credentials可使用用户名密码或SSH key**

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/1229ee379071411f9d48bce1c2a67e80-125842.jpeg)

### **2. 配置执行脚本**

```text
BUILD_ID=DONTKILLME
# 获取正在运行的ecs-application程序pid
pid=$(ps -aux|grep ecs-application | grep -v grep| gawk '{print $2}')
if [ ${#pid} != 0 ]
    then kill -9 $pid
fi
cd /var/lib/jenkins/workspace/ecs-application
mvn clean package
nohup java -jar /var/lib/jenkins/workspace/ecs-application/target/ecs-application.jar -Xmx512m -Xms512m -Xss4m &
pid=$(ps -aux|grep ecs-application | grep -v grep| gawk '{print $2}')
# 获取正在运行的ecs-application的pid并判断其字符串长度，0为不存在(即构建失败)
if [ ${#pid} == 0 ]
    then
     echo "*****  BUILD FAILED  ******"
     exit 1
     else
     echo "*****  BUILD SUCCESS  *****"
fi
```

需要注意的是Jenkins执行脚本中若不添加**BUILD\_ID=DONTKILLME**的话则会在执行完脚本后会把脚本中的程序关闭。jenkins创建的所有项目都可以在**/var/lib/jenkins/workspace**中找到。为了避免程序运行失败结果却显示成功，当执行程序后pid不存在则**exit 1**，且需设置build unstable符号值，如下图：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/a425b60557da42839f3ad55482065258-125842.jpeg)

如有邮箱发送需求也可在**Post-build Actions**设置**Email Notification**

### **3.构建**

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/d3b443191be3437dac4b7c05e890a946-125842.jpeg)

直接在项目Build Now或在jenkins主页面中点击项目最右侧符号即可。通过项目Workspace可查看**/var/lib/jenkins/workspace**当前项目中的所有文件，将日志文件配置为该workspace目录下文件时即可查看当前程序的运行日志，如在jenkins查看当前项目日志**/var/lib/jenkins/workspace/ecs-application/logs/info.log**：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/7f1108fce6e04922bffdbf170747ca49-125843.jpeg)

附日志配置文件logback-spring.xml\(为了出错更容易定位添加error.log配置\)：

```text
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true" scan="true">

    <!-- 文件输出格式 -->
    <property name="PATTERN"
              value="%-12(%d{yyyy-MM-dd HH:mm:ss.SSS}) %-5level [%thread] %c [%L] : %msg%n"/>
    <!-- log文件路径 -->
    <property name="LOG_PATH" value="/var/lib/jenkins/workspace/ecs-application/logs"/>

    <!--彩色日志定义-->

    <property name="CONSOLE_LOG_PATTERN"
              value="%date{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) %([%thread])  %cyan(%logger) : %msg%n"/>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <charset>UTF-8</charset>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="info" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_PATH}/info.log</File>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符 -->
            <pattern>${PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/info.%d{yyyy-MM-dd}.log.zip</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
    </appender>

    <appender name="error" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_PATH}/error.log</File>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <charset>UTF-8</charset>
            <pattern>${PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_PATH}/error.%d{yyyy-MM-dd}.log.zip</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
    </appender>

    <springProfile name="dev">
        <root level="info">
            <appender-ref ref="console"/>
            <appender-ref ref="info"/>
            <appender-ref ref="error"/>
        </root>
    </springProfile>
</configuration>
```

### **4. 配置Webhook**

每次推完代码之后都要上Jenkins重新点击启动项目，这肯定是很麻烦的，但可以通过安装Webhook插件，让github或gitlab接收到代码更新后把该信息发送到服务器jenkins上，让jenkins自动去拉代码重新部署项目。在Jenkins插件管理安装**Generic Webhook Trigger Plugin**插件：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/db6d93d27893484aae6189442be27437-125844.jpeg)

当然了，只装不看就没有用了，点击插件看一下了解一下用法：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/445683725a074f5a936a25d8c21b997f-125844.jpeg)

该Webhook插件的一个规则是接收所有HTTP请求，接收地址为**JenkinsURL/generic-webhook-trigger/invoke** 。既然有接收地址自然有发送地址，github配置Webhook入口在项目的**Settings**菜单下，配置如下：

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/a6c98df2095f4d7d92d53a2fff0d9caf-125845.jpeg)

虽然配置了github Webhooks，但Jenkins接收到github的消息后仍不知要更新哪个项目，因为可能jenkins下有多个同一github地址下的项目，此时就需要对Jenkins的项目进行Webhook配置了。上图中的URL地址栏添加了token参数，该参数是根据Jenkins的Webhook插件规则配置的。Jenkins安装Webhook插件后Jenkins项目**Configure**中的**Build Triggers**中会出现**Generic Webhook Trigger**选项，勾选该选项Jenkins即可监听到该项目对应的github仓库代码更新后自动重新部署项目。为了提高安全性可以在地址栏参数或header添加Token。

![Jenkins&#x5B89;&#x88C5;&#x5E76;&#x90E8;&#x7F72;Java&#x9879;&#x76EE;&#x5B8C;&#x6574;&#x6D41;&#x7A0B;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/60194e4c20884767b5908f9212a9331a-125846.jpeg)

注：如果没有硬件防护建议开启防火墙只暴露有需要的端口，避免服务器被挖矿\(亲身体验\)，相关指令如下\(在本文中的服务器是进行了8500端口的开放\)： 重启：firewall-cmd --reload 永久开放端口：firewall-cmd --zone=public --add-port={port}/tcp --permanent 永久关闭端口：firewall-cmd --remove-port={port}/tcp --permanent 查看开放端口：firewall-cmd --list-ports 关闭防火墙命令：systemctl stop firewalld.service 开启防火墙：systemctl start firewalld.service 更多指令可通过tab键或firewall-cmd -h查看

## 结语

在实际生产中如果项目只有几个人负责没有更多的要求的话按以上流程就基本可以完成项目的自动部署流程了\(曾经工作过的小公司大致流程跟以上差不多，开发公众号时自己一个负责也就没有太多的Jenkins鉴权管理\)，当然git的分支也可能需要切换，当项目有一定规模开发人员有一定数量时则必须做好权限的管理了，把分支的整合部署集权在少数人上，避免每次的小更改都需重新部署。以上流程看看就好，因为没必要记住自己吃过多少面包片，了解一些吃法就行。

