# Docker+Jenkins+Nginx+SpringBoot自动化部署项目

[https://www.toutiao.com/i6779098800825827852](https://www.toutiao.com/i6779098800825827852)



> Docker通过linux的namespace实现资源隔离、cgroups实现资源控制，通过写时复制机制\(copy-on-write\)实现了高效的文件操作，在实际开发中可用于提供一次性的环境、微服务架构的搭建、统一环境的部署。

![Docker+Jenkins+Nginx+SpringBoot&#x81EA;&#x52A8;&#x5316;&#x90E8;&#x7F72;&#x9879;&#x76EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/fd04165c7fb341c8b58272be5162aac0-123507.jpeg)

## 前言

虽然Docker已经是风靡全球的容器技术了，统一环境避免环境问题上是Docker的主要吸引点之一，但使用时详细还是会遇到不少问题的，比如个人搭建时曾思考过这些问题：

* Jenkins官网既然有Docker上安装Jenkins的流程了，那我该怎么使用Jenkins容器呢？
* 如果使用Jenkins容器，我该怎么通过Jenkins容器部署SpringBoot项目？是通过Jenkins容器与SpringBoot容器中的文件交互进行项目部署吗？这能做到吗？又或是把SpringBoot项目放到Jenkins容器中管理，那Jenkins中又要安装git、maven等一堆东西，这一点都不方便。
* 使用IDEA Docker插件都可以直接本地连接到服务器的Docker创建镜像并运行容器了，为什么还需要Jenkins？

个人在实际搭建部署中也找到了与上相对应的答案：

* 如果使用Jenkins容器，这将使得部署更加麻烦，因Jenkins往往需要配置Maven、git等一系列变量，应另寻出路。Jenkins既然是一款脚本CI工具，而Docker也有自己的脚本，我应该将Docker脚本集成到Docker中这方面考虑。
* 在实际开发中，Jenkins可能不仅需要项目的部署，还需要进行开发人员的鉴权，如开发人员A只能查看部署指定项目，管理员可以查看部署所有项目，但Docker主要用于镜像构建与容器运行，无法像Jenkins一样获取github/gitlab代码，也无法进行开发人员的鉴权，所以Docker可以在Jenkins中只扮演简化部署过程的一个角色。
* 虽然IDEA插件可以直接把本地打包成功的项目部署服务器Dcoker并创建镜像运行容器，但为了安全还需要创建Docker CA认证下载到本地再进行服务器上的Docker连接，十分不便捷。

## 环境准备

当探索到自我提问的答案时，便确定了各组件的主要职责：

* Jenkins：接收项目更新信息并进行项目打包与Docker脚本的执行
* Docker：安装所需应用镜像与运行容器
* git：项目信息同步

搭建环境流程：

1. 安装JDK
2. 安装Maven
3. 安装git
4. 安装Jenkins\(该步骤之前的可参考[Jenkins安装并部署Java项目完整流程](https://www.toutiao.com/i6774653293302907395/?group_id=6774653293302907395)\)如有权限问题可将/etc/sysconfig/jenkins文件JENKINS\_USER修改为root或手动赋权
5. Centos安装Docker
6. 安装DockerCompose

```text
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

使用DockerCompose可省去容器增多时需多次执行docker run的麻烦

## 具体步骤

**配置文件**

### **1. SpringBoot项目Dockerfile**

```text
FROM java:8

MAINTAINER Wilson

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#这里的 /tmp 目录就会在运行时自动挂载为匿名卷，任何向 /tmp 中写入的信息都不会记录进容器存储层
VOLUME /ecs-application-docker
RUN mkdir /app
WORKDIR /app


#复制target/spring-boot-web-demo.jar到容器里WORKDIR下
COPY target/ecs-application.jar  ecs-application.jar
EXPOSE 9090

ENTRYPOINT ["java","-jar","ecs-application.jar"]
```

### **2. 配置docker-compose.yml**

```text
version: '3.7'
services:
  app:
    restart: always
    build: ./
    hostname: docker-spring-boot
    container_name: docker-spring-boot
    image: docker-spring-boot/latest
    #    不对外开放端口，只能通过容器访问
#    ports:
#      - 8080:8080
    volumes:
      - ./volumes/app:/app
  nginx:
    depends_on:
      - app
    container_name: docker-nginx
    hostname: docker-nginx
    image: nginx:1.17.6
    environment:
      TZ: Asia/Shanghai
    restart: always
    expose:
      - 80
    ports:
      - 80:80
    links:
      - app
    volumes:
      - ./volumes/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./volumes/nginx/conf.d:/etc/nginx/conf.d
      - ./volumes/nginx/logs:/var/log/nginx
```

### **3. Nginx**

* ./volumes/nginx/nginx.conf

```text
user nginx;
worker_processes 2; #设置值和CPU核心数一致
error_log /etc/nginx/error.log crit; #日志位置和日志级别
pid /etc/nginx/nginx.pid;

events
{
  use epoll;
  worker_connections 65535;
}
http{
    include mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" "$http_cookie"';

    access_log  /var/log/nginx/access.log main;
    #charset utf8;

    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;

    sendfile on;
    tcp_nopush on;
    keepalive_timeout 60;
    tcp_nodelay on;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    #limit_zone crawler $binary_remote_addr 10m;
    #server虚拟主机的配置
    include /etc/nginx/conf.d/*.conf;

}
```

* ./volumes/nginx/conf.d目录下的default.conf

```text
upstream application {
   server docker-spring-boot:8080;
}

server{
  listen 80;#监听端口
  server_name localhost;#域名
  access_log /var/log/nginx/nginx-spring-boot.log;
  location / {
      proxy_pass   http://application;
      proxy_set_header Host $host:$server_port;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

### **Jenkins部署执行流程**

maven打包Spring Boot项目为project.jar

根据是否以第一次项目部署执行以下不同的流程：

* 如当前挂载卷已含项目jar\(即非第一次运行\)，则运行以下步骤:拷贝project.jar覆盖挂载卷中的project.jar重新运行SpringBoot项目容器
* 如当前挂载卷不含项目jar\(即非第一次运行\)，则运行以下步骤:创建挂载卷目录拷贝project.jar到挂载卷中通过docker-compose读取docker-compose.yml配置创建镜像启动容器

Jenkins脚本\(如果Nginx配置更改较多也可添加Nginx容器重启指令\)：

```text
cd /var/lib/jenkins/workspace/docker-spring-boot/spring-boot-nginx-docker-demo
mvn clean package
if [ -e "./volumes/app/docker-spring-boot.jar" ]
  then rm -f ./volumes/app/docker-spring-boot.jar \
        && cp ./target/docker-spring-boot.jar ./volumes/app/docker-spring-boot.jar \
        && docker restart docker-spring-boot \
        && echo "update restart success"
  else mkdir volumes/app -p \
        && cp ./target/docker-spring-boot.jar ./volumes/app/docker-spring-boot.jar \
        && docker-compose -p docker-spring-boot up -d \
        && echo "first start"
fi
```

docker-compose up指令可以进行镜像的安装，所以也省去了只用docker指令时需要提前准备好镜像相关指令的麻烦。

## 结果测试

* 查看容器是否皆已启动:docker ps

![Docker+Jenkins+Nginx+SpringBoot&#x81EA;&#x52A8;&#x5316;&#x90E8;&#x7F72;&#x9879;&#x76EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/e0a361adbdab416e9b33f25d3c4a6dde-123507.jpeg)

* SpringBoot容器运行结果查看：如容器开放了8080端口则可通过[http://url:8080/swagger-ui.html测试，也可通过查看Jenkins工作空间下/volumes/app的SpringBoot日志校验结果\(SpringBoot日志的路径配置个人设置为app/logs目录下，前文已把容器中的app目录挂载到当前项目的volumes/app目录下](http://url:8080/swagger-ui.html测试，也可通过查看Jenkins工作空间下/volumes/app的SpringBoot日志校验结果%28SpringBoot日志的路径配置个人设置为app/logs目录下，前文已把容器中的app目录挂载到当前项目的volumes/app目录下)\)

![Docker+Jenkins+Nginx+SpringBoot&#x81EA;&#x52A8;&#x5316;&#x90E8;&#x7F72;&#x9879;&#x76EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/158b09abb84f44e28695d5261ec8ddc6-123508.jpeg)

* Nginx容器运行结果查看：访问[http://url/swagger-ui.html测试是否Nginx容器已成功连通SpringBoot容器并进行了反向代理，也可通过查看Jenkins工作空间下/volumes/nginx/logs的Nginx日志校验结果](http://url/swagger-ui.html测试是否Nginx容器已成功连通SpringBoot容器并进行了反向代理，也可通过查看Jenkins工作空间下/volumes/nginx/logs的Nginx日志校验结果)

![Docker+Jenkins+Nginx+SpringBoot&#x81EA;&#x52A8;&#x5316;&#x90E8;&#x7F72;&#x9879;&#x76EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/18a40b0e34e44dce9126c08329b509b3-123508.jpeg)

* 添加或删除controller接口再进行推到git，查看更改的接口是否可访问

## SpringBoot集群搭建

如需将SpringBoot通过容器集群搭建，只需进行以下更改：

* docker-compose.yml添加SpringBoot项目冗余，更改冗余容器名，区分日志挂载路径，冗余项目更改容器名

```text
version: '3.7'
services:
  app:
    restart: always
    build: ./
    hostname: docker-spring-boot
    container_name: docker-spring-boot
    image: docker-spring-boot/latest
    volumes:
      - ./volumes/app/docker-spring-boot.jar:/app/docker-spring-boot.jar
      - ./volumes/app/logs:/app/logs
  app-bak:
    restart: always
    build: ./
    hostname: docker-spring-boot
    container_name: docker-spring-boot-bak
    image: docker-spring-boot/latest
    volumes:
      - ./volumes/app/docker-spring-boot.jar:/app/docker-spring-boot.jar
      - ./volumes/app/logs-bak:/app/logs
  nginx:
    depends_on:
      - app
    container_name: docker-nginx
    hostname: docker-nginx
    image: nginx:1.17.6
    environment:
      TZ: Asia/Shanghai
    restart: always
    expose:
      - 80
    ports:
      - 80:80
    links:
      - app
      - app-bak
    volumes:
      - ./volumes/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./volumes/nginx/conf.d:/etc/nginx/conf.d
      - ./volumes/nginx/logs:/var/log/nginx
```

* nginx更改default.conf的upstream，添加冗余容器配置

```text
upstream application {
   server docker-spring-boot:8080 fail_timeout=2s max_fails=2 weight=1;
   server docker-spring-boot-bak:8080 fail_timeout=2s max_fails=2 weight=1;
}

server{
  listen 80;#监听端口
  server_name localhost;#域名
  access_log /var/log/nginx/nginx-spring-boot.log;
  location / {
      proxy_pass   http://application;
      proxy_connect_timeout 2s;
      proxy_set_header Host $host:$server_port;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

* Jenkins添加冗余容器重启脚本

```text
BUILD_ID=DONTKILLME
cd /var/lib/jenkins/workspace/docker-spring-boot/spring-boot-nginx-docker-demo
mvn clean package
if [ -e "./volumes/app/docker-spring-boot.jar" ]
  then rm -f ./volumes/app/docker-spring-boot.jar \
        && cp ./target/docker-spring-boot.jar ./volumes/app/docker-spring-boot.jar \
        && docker-compose -p docker-spring-boot up -d \
        && docker restart docker-spring-boot \
        && docker restart docker-spring-boot-bak \
        && docker restart docker-nginx \
        && echo "update restart success"
  else mkdir volumes/app -p \
        && cp ./target/docker-spring-boot.jar ./volumes/app/docker-spring-boot.jar \
        && docker-compose -p docker-spring-boot up -d \
        && echo "first start"
fi
```

测试集群效果：

* volumes/app放置了不同容器的日志，如该例子的logs、logs-bak
* 停止任一SpringBoot容器docker stop docker-spring-boot，仍可通过url/api通过Nginx访问

可以看出容器配置集群的以下优点：

* 安全性高，每一个应用都只属一个容器，通过特定配置才可与主机、其它容器交互
* 统一配置文件，简单粗暴的方式解决端口、路径、版本等配置问题，如该项目即使运行了2个8080端口的SpringBoot容器而不需担心端口的冲突、暴露问题，一切都在容器内解决
* 省略手动应用安装，易于迁移，由于版本、配置、环境等都已配置在Docker的配置文件中，所以不用担心更换机器后出现的各种配置、环境问题，且通过镜像拉取与容器运行可以省略如Nginx、Redis、Mysql等应用的安装与配置

![Docker+Jenkins+Nginx+SpringBoot&#x81EA;&#x52A8;&#x5316;&#x90E8;&#x7F72;&#x9879;&#x76EE;](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpeg/2020/07/22/2a287f6a56eb4edfb7aaf8cceef39de0-123509.jpeg)

> 该文章及相关资源可通过个人信息中的个人博客查看

