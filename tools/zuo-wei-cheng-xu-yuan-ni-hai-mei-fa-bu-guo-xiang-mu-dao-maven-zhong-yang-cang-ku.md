# 作为程序员，你还没发布过项目到Maven中央仓库

[https://mp.weixin.qq.com/s/EaBdgdNIuWBNuKwwNRINqw](https://mp.weixin.qq.com/s/EaBdgdNIuWBNuKwwNRINqw)

[https://mp.weixin.qq.com/mp/appmsgalbum?\_\_biz=MzI0NDAzMzIyNQ==&action=getalbum&album\_id=1328385793563639809&subscene=159&subscene=&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FEaBdgdNIuWBNuKwwNRINqw\#wechat\_redirect](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzI0NDAzMzIyNQ==&action=getalbum&album_id=1328385793563639809&subscene=159&subscene=&scenenote=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FEaBdgdNIuWBNuKwwNRINqw#wechat_redirect)

在Maven项目中，90%以上的jar包是通过pom文件直接从开源仓库中获取依赖jar包文件，然后在项目中进行集成使用。

此时如果你有一个开源项目，那么如何将该开源项目发布到Maven中央仓库，让其他人可以方便的使用，而不是先下载jar，然后install的本地？

本文将通过一步步的操作带领大家讲自己的开源项目发布到Maven中央仓库\(Maven Central Repository\)中，[https://mvnrepository.com/。](https://mvnrepository.com/。)

Maven中央仓库并不支持直接发布jar包，需要将jar包发布到一些指定的第三方Maven仓库，然后该仓库再将jar包同步到Maven中央仓库，Sonatype便是这样的角色。

本文系统配置如下：1、操作系统macOS 10.14.2；2、JDK1.8.0\_192；3、Maven：3.5.4。

## 准备工作

注册GitHub的账户，地址：[https://github.com。既然是开源项目，肯定需要有一个地方托管，这里采用GitHub。](https://github.com。既然是开源项目，肯定需要有一个地方托管，这里采用GitHub。)

然后创建项目，上传对应的项目代码。

很多朋友已经有了GitHub的账户及开源项目，这几乎是程序员必备的一个平台，如果没有，赶紧去开通一个。

## 注册Sonatype的账户

Sonatype通过JIRA来管理OSSRH仓库。JIRA是一个项目管理服务，类似于国内的Teambition。

注册地址：[https://issues.sonatype.org/secure/Signup!default.jspa](https://issues.sonatype.org/secure/Signup!default.jspa)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100917391-100917.jpg)

其中，Username便是登录名称，Password为登录密码，它俩是一套。

【友情提示】密码校验非常苛刻，最好用记事本记录备忘。

注册完成，首次登录会让选择语言，这里本人选择了中文。

## 项目的发布申请

首次登录Sonatype会直接跳转到创建页面，之后登录创建发布项目申请需点击导航栏的“新建”进行项目或问题的创建发布。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100917496-100917.jpg)

通过在JIRA上创建issue来申请发布新的jar包，Sonatype的工作人员会进行审核，一般按照要求填写不会有问题。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100917644-100917.jpg)

上图问中文版本时填写信息，其中项目要选择“Community Support”项，对应的问题类型选择“New Project”。按照上述选择，才会在下面展示出对应GroupId和Project信息。

Project URL便是项目的URL地址，也就是你访问到GitHub项目时的浏览器URL，比如：[https://github.com/secbr/fastdfs-client-plus](https://github.com/secbr/fastdfs-client-plus)

SCM url是基于Https形式访问源代码的链接。我们知道获取GitHub的源代码可以有多种形式，下载zip包、通过“Use ssh”，“Use Https”等形式下载，这里的SCM URL便是Https的url地址，比如：[https://github.com/secbr/fastdfs-client-plus.git](https://github.com/secbr/fastdfs-client-plus.git)

其他非必填信息根据需要选择填写即可。

提交完成之后，会创建一个Issues，内容显示如下： 

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100917752-100917.jpg)

很显然，目前处于“待解决”状态，等审核。在写这篇文章十多分钟之后便受到官方审核人员的回复“Waiting for Response”。同时，在Issues下方会出现对应的提示注释信息。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918036-100918.jpg)

这里主要是为验证上面的GroupId，来确定对应的域名是否是你所拥有的，比如这里填写的GroupId为top.folen。那么需要验证的域名便是：folen.top。

注释中提供了两种验证方式，一种是按照要求配置DNS，一种是直接使用GitHub的二级域名。

这里本人最终选择使用GitHub的二级域名，作为GroupID，此时，官方回复如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918179-100918.jpg)

平台为了验证是否拥有GitHub的账户权限，因此需要申请者在GitHub上创建一个名称为“OSSRH-59503”的项目。在GitHub上创建这么一个空项目，然后在评论区回复即可。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918290-100918.jpg)

回复完成之后，稍微等待一下，便审核完成。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918403-100918.jpg)

整个上述过程用大概40分钟，官方回复的还比较及时，由于是下午四五点进行操作的，不确定大家在操作时是否会遇到时差问题。大家可在主面板上查看一下最近其他人提交的Issues的回复情况来确认是否等待。

## 安装并配置GPG

发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。因此，我们需要在本地安装并配置GPG。

本人采用Mac操作系统，关于其他操作系统的安装大家自行搜索。

MacBook安装GPG非常简单，下载并安装GPG Suite即可：[https://gpgtools.org/](https://gpgtools.org/)

安装完成可进入创建GPG密钥对的操作界面，Mac下安装完成弹出如下页面：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918668-100918.jpg)

生成密钥时将需要输入name、email以及password。秘钥password在之后的步骤需要用到，请记下来。

公钥创建完成，会自动弹出上传到公共的密钥服务器，这样其他人才可以通过公钥来验证jar包的完整性。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100918831-100918.jpg)

如果忘记了公钥信息可执行gpg --list-keys命令查看本地公钥信息。

```text
192:~ zzs$ gpg --list-keys
/Users/zzs/.gnupg/pubring.kbx
-----------------------------
pub   dsa2048 2010-08-19 [SC] [过期于：2020-06-15]
      85E38F69046B44C1EC9FB07B76D78F0500D026C4
uid           [ 已过期 ] GPGTools Team <team@gpgtools.org>

pub   rsa4096 2020-05-04 [SC] [有效至：2024-05-03]
      B97E9964ACAD1907970D37CC8A9E3745558E41AF
uid           [ 未知 ] GPGTools Support <support@gpgtools.org>
sub   rsa4096 2020-05-04 [E] [有效至：2024-05-03]

pub   rsa4096 2020-07-27 [SC] [有效至：2024-07-27]
      74C31F28121A99A6E28C234148FEC679B82EF754
uid           [ 绝对 ] secbro <secbro2@gmail.com>
sub   rsa4096 2020-07-27 [E] [有效至：2024-07-27]
```

也可以通过如下形式将公钥信息上传到服务器：

```text
gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys B97E9964ACAD1907970D37CC8A9E3745558E41AF
```

其中B97E9964ACAD1907970D37CC8A9E3745558E41AF便是上面查询到的。

如果是Windows操作系统，同样安装完软件，打开cmd命令行，执行gpg --gen-key生成key，执行gpg --list-keys查看key列表，执行上述命令上传key。

## 配置Maven的setting.xml

setting.xml为Maven的全局配置文件，路径为$MAVEN\_HOME/conf/settings.xml，需要注册Sonatype的账户时配置的Username和Password添加到servers标签中，这样才能将jar包部署到Sonatype OSSRH仓库：

```text
<server>
  <id>sonatype-nexus-snapshots</id>
  <username>Sonatype账号</username>
  <password>Sonatype密码</password>
</server>
```

## 配置项目的pom.xml

根据Sonatype OSSRH的要求，以下信息都必须配置：

* Supply Javadoc and Sources
* Sign Files with GPG/PGP
* Sufficient Metadata
* Correct Coordinates
* Project Name, Description and URL
* License Information
* Developer Information
* SCM Information

增加开源许可协议，SCM信息，开发者信息等待根据自己信息填写即可。

```text
<licenses>
    <license>
      <name>BSD 3-Clause</name>
      <url>https://spdx.org/licenses/BSD-3-Clause.html</url>
    </license>
  </licenses>
  <scm>
    <connection>https://github.com/secbr/fastdfs-client-plus.git</connection>
    <url>https://github.com/secbr/fastdfs-client-plus</url>
  </scm>
  <developers>
    <developer>
      <name>secbr</name>
      <email>secbro2@gmail.com</email>
      <roles>
        <role>Developer</role>
      </roles>
      <timezone>+8</timezone>
    </developer>
  </developers>
```

如果发布Release版本，需要添加Release的相关profile配置，distributionManagement节和maven-compiler-plugin节的配置信息根据自己的实际情况做修改。

```text
<profiles>
    <profile>
      <id>release</id>
      <build>
        <resources>
          <resource>
            <directory>src/main/java</directory>
            <includes>
              <include>**/*.properties</include>
              <include>**/*.sample</include>
            </includes>
          </resource>
        </resources>
        <plugins>
          <!-- Source -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.2.1</version>
            <executions>
              <execution>
                <phase>package</phase>
                <goals>
                  <goal>jar-no-fork</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <!-- Javadoc -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>2.9.1</version>
            <configuration>
              <show>private</show>
              <nohelp>true</nohelp>
              <charset>UTF-8</charset>
              <encoding>UTF-8</encoding>
              <docencoding>UTF-8</docencoding>
              <additionalparam>-Xdoclint:none</additionalparam>
              <javadocExecutable>/Library/Java/JavaVirtualMachines/jdk1.8.0_151.jdk/Contents/Home/bin/javadoc</javadocExecutable>
              <!-- TODO 临时解决不规范的javadoc生成报错,后面要规范化后把这行去掉 -->
            </configuration>
            <executions>
              <execution>
                <phase>package</phase>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <!-- GPG -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>1.6</version>
            <executions>
              <execution>
                <phase>verify</phase>
                <goals>
                  <goal>sign</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <!--Compiler -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
              <source>1.8</source>
              <target>1.8</target>
              <fork>true</fork>
              <verbose>true</verbose>
              <encoding>UTF-8</encoding>
              <showWarnings>false</showWarnings>
            </configuration>
          </plugin>
          <!--Release -->
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-release-plugin</artifactId>
            <version>2.5.1</version>
          </plugin>
        </plugins>
      </build>
      <distributionManagement>
        <snapshotRepository>
          <id>sonatype-nexus-snapshots</id>
          <name>Sonatype Nexus Snapshots</name>
          <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        </snapshotRepository>
        <repository>
          <id>sonatype-nexus-snapshots</id>
          <name>Nexus Release Repository</name>
          <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
        </repository>
      </distributionManagement>
    </profile>
  </profiles>
```

其中snapshotRepository便是在setting.xml中定义的server的id。

## 发布Jar包

完成上述配置，则可通过命令进行打包上传，即可将jar包发布到Sonatype OSSRH仓库。

```text
mvn clean deploy -P release
```

执行上述命令，打包并上传。release便是上面配置的profile元素的id。

在执行的过程中需要输入上面设置的key的密码。执行成功控制台显示情况如图。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100919151-100919.jpg)

【友情提示】如果打包过程中出现了401类的错误，可能是因为Maven的配置文件中Server节点配置的用户名和密码不正确，或者Issue还未审核通过。

此时访问上面的任何一个链接，便查询对应的信息。比如将url中的具体文件去掉，只留如下路径：[https://oss.sonatype.org/content/repositories/snapshots/com/github/secbr/fastdfs-client-plus/1.0.0-SNAPSHOT/](https://oss.sonatype.org/content/repositories/snapshots/com/github/secbr/fastdfs-client-plus/1.0.0-SNAPSHOT/)

访问上述路径，便可查看到所有上传的文件信息。

## 查看发布jar包

此时进入[https://oss.sonatype.org/\#stagingRepositories查看发布好的构件，点击左侧的Staging](https://oss.sonatype.org/#stagingRepositories查看发布好的构件，点击左侧的Staging) Repositories，可以使用Group Id或其他信息搜索自己的项目。

如果弹出用户名或密码，则输入注册sonatype时对应的用户和密码信息。

此时需注意，如果项目中版本信息为1.0.0-SNAPSHOT，即SNAPSHOT为后缀，则发布的项目位于Snapshots目录下。在左上角的Artifact Search中能够搜到。

如果是Release后缀，则可直接在Staging Repositories中看到（有可能要稍等一下，等待平台处理）。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100919325-100919.jpg)

选中对于的repository之后，点击的close，close时会检查发布的构件是否符合要求。若符合要求，则close成功，成功之后点击箭头所指的release，即可正式将jar包发布到Sonatype OSSRH仓库。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100919558-100919.jpg)

当点击Release之后，邮件中会收到Issues变化的信息，提示同步已经被激活，正常10分钟就可以更新同步。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100919843-100919.jpg)

release成功大概2个小时之后，该构件就会同步到Maven中央仓库，届时会有邮件通知。

实践过程中发现十分钟之内已经成功同步到[https://repo1.maven.org/的中央仓库当中。](https://repo1.maven.org/的中央仓库当中。) 

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/29/640-20200729100920349-100920.jpg)

逐步的其他Maven仓库，包括阿里镜像都会进行同步。

## 收尾

当发布到Maven中央仓库完成，可以看到对应的Jar包时，可以对自己提交的Issue增加Comment，留言致谢并表示发布已经完成，请工作人员关闭Issue。有始有终。

作为程序员，终于在Maven中央仓库有一套自己的代码是不是很兴奋的一件事！分享、点赞、在看来一波。

