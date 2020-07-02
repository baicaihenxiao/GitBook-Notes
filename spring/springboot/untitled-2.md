# 面试官：为什么SpringBoot的 jar 可以直接运行？

[https://mp.weixin.qq.com/s/h2EogQizFKRwiAB3EqFodg](https://mp.weixin.qq.com/s/h2EogQizFKRwiAB3EqFodg)  


SpringBoot提供了一个插件spring-boot-maven-plugin用于把程序打包成一个可执行的jar包。在pom文件里加入这个插件即可：

```text
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

打包完生成的executable-jar-1.0-SNAPSHOT.jar内部的结构如下：

```text
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── spring.study
│           └── executable-jar
│               ├── pom.properties
│               └── pom.xml
├── lib
│   ├── aopalliance-1.0.jar
│   ├── classmate-1.1.0.jar
│   ├── spring-boot-1.3.5.RELEASE.jar
│   ├── spring-boot-autoconfigure-1.3.5.RELEASE.jar
│   ├── ...
├── org
│   └── springframework
│       └── boot
│           └── loader
│               ├── ExecutableArchiveLauncher$1.class
│               ├── ...
└── spring
    └── study
        └── executablejar
            └── ExecutableJarApplication.class
```

然后可以直接执行jar包就能启动程序了：

`java -jar executable-jar-1.0-SNAPSHOT.jar`

打包出来fat jar内部有4种文件类型：

* META-INF文件夹：程序入口，其中MANIFEST.MF用于描述jar包的信息
* lib目录：放置第三方依赖的jar包，比如springboot的一些jar包
* spring boot loader相关的代码
* 模块自身的代码

MANIFEST.MF文件的内容：

```text
Manifest-Version: 1.0
Implementation-Title: executable-jar
Implementation-Version: 1.0-SNAPSHOT
Archiver-Version: Plexus Archiver
Built-By: Format
Start-Class: spring.study.executablejar.ExecutableJarApplication
Implementation-Vendor-Id: spring.study
Spring-Boot-Version: 1.3.5.RELEASE
Created-By: Apache Maven 3.2.3
Build-Jdk: 1.8.0_20
Implementation-Vendor: Pivotal Software, Inc.
Main-Class: org.springframework.boot.loader.JarLauncher
```

我们看到，它的Main-Class是org.springframework.boot.loader.JarLauncher，当我们使用java -jar执行jar包的时候会调用JarLauncher的main方法，而不是我们编写的SpringApplication。

那么JarLauncher这个类是的作用是什么的？

它是SpringBoot内部提供的工具Spring Boot Loader提供的一个用于执行Application类的工具类\(fat jar内部有spring loader相关的代码就是因为这里用到了\)。相当于Spring Boot Loader提供了一套标准用于执行SpringBoot打包出来的jar

### Spring Boot Loader抽象的一些类

抽象类Launcher：各种Launcher的基础抽象类，用于启动应用程序；跟Archive配合使用；目前有3种实现，分别是JarLauncher、WarLauncher以及PropertiesLauncher

Archive：归档文件的基础抽象类。JarFileArchive就是jar包文件的抽象。它提供了一些方法比如getUrl会返回这个Archive对应的URL；getManifest方法会获得Manifest数据等。ExplodedArchive是文件目录的抽象

JarFile：对jar包的封装，每个JarFileArchive都会对应一个JarFile。JarFile被构造的时候会解析内部结构，去获取jar包里的各个文件或文件夹，这些文件或文件夹会被封装到Entry中，也存储在JarFileArchive中。如果Entry是个jar，会解析成JarFileArchive。

比如一个JarFileArchive对应的URL为：

```text
jar:file:/Users/format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/
```

它对应的JarFile为：

```text
/Users/format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar
```

这个JarFile有很多Entry，比如：

```text
META-INF/
META-INF/MANIFEST.MF
spring/
spring/study/
....
spring/study/executablejar/ExecutableJarApplication.class
lib/spring-boot-starter-1.3.5.RELEASE.jar
lib/spring-boot-1.3.5.RELEASE.jar
...
```

JarFileArchive内部的一些依赖jar对应的URL\(SpringBoot使用org.springframework.boot.loader.jar.Handler处理器来处理这些URL\)：

```text
jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-starter-web-1.3.5.RELEASE.jar!/jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-loader-1.3.5.RELEASE.jar!/org/springframework/boot/loader/JarLauncher.class
```

我们看到如果有jar包中包含jar，或者jar包中包含jar包里面的class文件，那么会使 用 !/ 分隔开，这种方式只有org.springframework.boot.loader.jar.Handler能处 理，它是SpringBoot内部扩展出来的一种URL协议。

### JarLauncher的执行过程

JarLauncher的main方法：

```text
public static void main(String[] args) {
    // 构造JarLauncher，然后调用它的launch方法。参数是控制台传递的
    new JarLauncher().launch(args);
}  
```

JarLauncher被构造的时候会调用父类ExecutableArchiveLauncher的构造方法。

ExecutableArchiveLauncher的构造方法内部会去构造Archive，这里构造了JarFileArchive。构造JarFileArchive的过程中还会构造很多东西，比如JarFile，Entry …

```text
JarLauncher的launch方法：
protected void launch(String[] args) {
  try {
    // 在系统属性中设置注册了自定义的URL处理器：org.springframework.boot.loader.jar.Handler。如果URL中没有指定处理器，会去系统属性中查询
    JarFile.registerUrlProtocolHandler();
    // getClassPathArchives方法在会去找lib目录下对应的第三方依赖JarFileArchive，同时也会项目自身的JarFileArchive
    // 根据getClassPathArchives得到的JarFileArchive集合去创建类加载器ClassLoader。这里会构造一个LaunchedURLClassLoader类加载器，这个类加载器继承URLClassLoader，并使用这些JarFileArchive集合的URL构造成URLClassPath
    // LaunchedURLClassLoader类加载器的父类加载器是当前执行类JarLauncher的类加载器
    ClassLoader classLoader = createClassLoader(getClassPathArchives());
    // getMainClass方法会去项目自身的Archive中的Manifest中找出key为Start-Class的类
    // 调用重载方法launch
    launch(args, getMainClass(), classLoader);
  }
  catch (Exception ex) {
    ex.printStackTrace();
    System.exit(1);
  }
}// Archive的getMainClass方法
// 这里会找出spring.study.executablejar.ExecutableJarApplication这个类
public String getMainClass() throws Exception {
  Manifest manifest = getManifest();
  String mainClass = null;
  if (manifest != null) {
    mainClass = manifest.getMainAttributes().getValue("Start-Class");
  }
  if (mainClass == null) {
    throw new IllegalStateException(
        "No 'Start-Class' manifest entry specified in " + this);
  }
  return mainClass;
}// launch重载方法
protected void launch(String[] args, String mainClass, ClassLoader classLoader)
    throws Exception {
      // 创建一个MainMethodRunner，并把args和Start-Class传递给它
  Runnable runner = createMainMethodRunner(mainClass, args, classLoader);
      // 构造新线程
  Thread runnerThread = new Thread(runner);
      // 线程设置类加载器以及名字，然后启动
  runnerThread.setContextClassLoader(classLoader);
  runnerThread.setName(Thread.currentThread().getName());
  runnerThread.start();
}
```

MainMethodRunner的run方法：

```text
@Override
public void run() {
  try {
    // 根据Start-Class进行实例化
    Class<?> mainClass = Thread.currentThread().getContextClassLoader()
        .loadClass(this.mainClassName);
    // 找出main方法
    Method mainMethod = mainClass.getDeclaredMethod("main", String[].class);
    // 如果main方法不存在，抛出异常
    if (mainMethod == null) {
      throw new IllegalStateException(
          this.mainClassName + " does not have a main method");
    }
    // 调用
    mainMethod.invoke(null, new Object[] { this.args });
  }
  catch (Exception ex) {
    UncaughtExceptionHandler handler = Thread.currentThread()
        .getUncaughtExceptionHandler();
    if (handler != null) {
      handler.uncaughtException(Thread.currentThread(), ex);
    }
    throw new RuntimeException(ex);
  }
}
```

Start-Class的main方法调用之后，内部会构造Spring容器，启动内置Servlet容器等过程。这些过程我们都已经分析过了。

### 关于自定义的类加载器LaunchedURLClassLoader

LaunchedURLClassLoader重写了loadClass方法，也就是说它修改了默认的类加载方式\(先看该类是否已加载这部分不变，后面真正去加载类的规则改变了，不再是直接从父类加载器中去加载\)。LaunchedURLClassLoader定义了自己的类加载规则：

```text
private Class<?> doLoadClass(String name) throws ClassNotFoundException {  // 1) Try the root class loader
  try {
    if (this.rootClassLoader != null) {
      return this.rootClassLoader.loadClass(name);
    }
  }
  catch (Exception ex) {
    // Ignore and continue
  }  // 2) Try to find locally
  try {
    findPackage(name);
    Class<?> cls = findClass(name);
    return cls;
  }
  catch (Exception ex) {
    // Ignore and continue
  }  // 3) Use standard loading
  return super.loadClass(name, false);
}
```

加载规则：

* 如果根类加载器存在，调用它的加载方法。这里是根类加载是ExtClassLoader
* 调用LaunchedURLClassLoader自身的findClass方法，也就是URLClassLoader的findClass方法
* 调用父类的loadClass方法，也就是执行默认的类加载顺序\(从BootstrapClassLoader开始从下往下寻找\)

LaunchedURLClassLoader自身的findClass方法：

```text
protected Class<?> findClass(final String name)
     throws ClassNotFoundException
{
    try {
        return AccessController.doPrivileged(
            new PrivilegedExceptionAction<Class<?>>() {
                public Class<?> run() throws ClassNotFoundException {
                    // 把类名解析成路径并加上.class后缀
                    String path = name.replace('.', '/').concat(".class");
                    // 基于之前得到的第三方jar包依赖以及自己的jar包得到URL数组，进行遍历找出对应类名的资源
                    // 比如path是org/springframework/boot/loader/JarLauncher.class，它在jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-loader-1.3.5.RELEASE.jar!/中被找出
                    // 那么找出的资源对应的URL为jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-loader-1.3.5.RELEASE.jar!/org/springframework/boot/loader/JarLauncher.class
                    Resource res = ucp.getResource(path, false);
                    if (res != null) { // 找到了资源
                        try {
                            return defineClass(name, res);
                        } catch (IOException e) {
                            throw new ClassNotFoundException(name, e);
                        }
                    } else { // 找不到资源的话直接抛出ClassNotFoundException异常
                        throw new ClassNotFoundException(name);
                    }
                }
            }, acc);
    } catch (java.security.PrivilegedActionException pae) {
        throw (ClassNotFoundException) pae.getException();
    }
}
```

下面是LaunchedURLClassLoader的一个测试：

```text
// 注册org.springframework.boot.loader.jar.Handler URL协议处理器
JarFile.registerUrlProtocolHandler();
// 构造LaunchedURLClassLoader类加载器，这里使用了2个URL，分别对应jar包中依赖包spring-boot-loader和spring-boot，使用 "!/" 分开，需要org.springframework.boot.loader.jar.Handler处理器处理
LaunchedURLClassLoader classLoader = new LaunchedURLClassLoader(
        new URL[] {
                new URL("jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-loader-1.3.5.RELEASE.jar!/")
                , new URL("jar:file:/Users/Format/Develop/gitrepository/springboot-analysis/springboot-executable-jar/target/executable-jar-1.0-SNAPSHOT.jar!/lib/spring-boot-1.3.5.RELEASE.jar!/")
        },
        LaunchedURLClassLoaderTest.class.getClassLoader());// 加载类
// 这2个类都会在第二步本地查找中被找出(URLClassLoader的findClass方法)
classLoader.loadClass("org.springframework.boot.loader.JarLauncher");
classLoader.loadClass("org.springframework.boot.SpringApplication");
// 在第三步使用默认的加载顺序在ApplicationClassLoader中被找出
classLoader.loadClass("org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration");
```

### Spring Boot Loader的作用

SpringBoot在可执行jar包中定义了自己的一套规则，比如第三方依赖jar包在/lib目录下，jar包的URL路径使用自定义的规则并且这个规则需要使用org.springframework.boot.loader.jar.Handler处理器处理。

它的Main-Class使用JarLauncher，如果是war包，使用WarLauncher执行。这些Launcher内部都会另起一个线程启动自定义的SpringApplication类。

这些特性通过spring-boot-maven-plugin插件打包完成。

**想知道更多？扫描下面的二维码关注我**

后台回复”加群“获取公众号专属群聊入口

【原创系列 \| 精彩推荐】

* \*\*\*\*[**Paxos、Raft不是一致性算法嘛？**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489451&idx=1&sn=fea7a5138f0497015532e8cf6bec3c37&chksm=fb0bfd3fcc7c74291b91eb5800dce0ea393baf1b2c5c4f91de2067785431f607a6b794f4ceb9&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**越说越迷糊的CAP**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489488&idx=1&sn=53409fced85f40494f38d703e91c6928&chksm=fb0bfd44cc7c7452ea1a71072b68b3b099cacaf73128e5a27a8c7258aacce8b72c8f717c0fd6&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**分布式事务科普——初识篇**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489849&idx=1&sn=cbac2a6ad99ac466f2ba8d69507fd2fe&chksm=fb0bf3adcc7c7abb565a9865e14b357888f7b7b78874b74c18bfdc5a4278ec2503b258c27730&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**分布式事务科普——终结篇**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489868&idx=1&sn=ef0b6afc60a89d49959ddb414ab20068&chksm=fb0bf3d8cc7c7ace7948f14b58d0a1562cf1449a2767964836eb47d669915206079760b0cb8e&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**面试官居然问我Raft为什么会叫做Raft!**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489611&idx=1&sn=407f2a3d9e5190c89064752eeded6e5d&chksm=fb0bf2dfcc7c7bc93fed2f746b6f7c128ea6d56f33163bb6796c7bd3f4adf2f69acda4948c1a&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**面试官给我挖坑：URI中的//有什么用**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489633&idx=1&sn=0616e86b4fb44df85c52f7422dad7b4e&chksm=fb0bf2f5cc7c7be34c48b555db5d704923319e43932ef5d65346c934b8daf7b6789d5eb15a16&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**面试官给我挖坑：a\[i\]\[j\]和a\[j\]\[i\]有什么区别？**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489899&idx=1&sn=7646d24134d58bbd9abccb38efd5b4d7&chksm=fb0bf3ffcc7c7ae978923cbc0a699a30351ee5563b3143a1b7ff51a13442438d52afbfb0aae5&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**面试官给我挖坑：单机并发TCP连接数到底有多少？**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247490002&idx=1&sn=ef4bb28e50a2375060efc46e55e27b77&chksm=fb0bf346cc7c7a50580c1b83c9c4d8a8d11b5018c6143c6444b7bfdb0fa06cef83497a708ba4&scene=21#wechat_redirect) ****
* \*\*\*\*[**网关Zuul科普**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489653&idx=1&sn=b2ed7b657b67c147483571ae01cb9aae&chksm=fb0bf2e1cc7c7bf7cf67599cf5ced169f71724b3ce6d00280f05528d03b3d5b623d74ef617dd&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**网关Spring Cloud Gateway科普**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489737&idx=1&sn=84dbee4cc343848c78a8505cd3cdadbd&chksm=fb0bf25dcc7c7b4b2d155d14d3ba72b872242cdcc40e60bbce950ad8070655b53249b9c84deb&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**Nginx架构原理科普**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247489921&idx=1&sn=0137f8212d94062987d83767836514ac&chksm=fb0bf315cc7c7a031935a531d556274c3158869bfb6d37d37d6804cce9af9596b8c24cc97f03&scene=21#wechat_redirect)\*\*\*\*
* \*\*\*\*[**OpenResty概要及原理科普**](http://mp.weixin.qq.com/s?__biz=MzU0MzQ5MDA0Mw==&mid=2247490049&idx=1&sn=47e297afd0cb938ab74936e8cc4cc905&chksm=fb0bf095cc7c7983af37418e1adf17fd9d0a0fddbaf5584af639d2fd5bf2fd422096764362c1&scene=21#wechat_redirect) ****

**朕已阅** 

