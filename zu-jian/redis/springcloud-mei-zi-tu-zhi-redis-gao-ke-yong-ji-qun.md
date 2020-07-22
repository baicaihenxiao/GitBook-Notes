# SpringCloud 妹子图之 Redis 高可用集群

[https://mp.weixin.qq.com/s/H6DxNxyZXTpQwWDNF8UjtA](https://mp.weixin.qq.com/s/H6DxNxyZXTpQwWDNF8UjtA)

### **前言**

一般的小项目，比如几百人左右访问的项目，访问量几万的项目，如果想用缓存，单机实例完全够用。小黄图就是用的阿里云`256MB`配置的`Redis`缓存，日几千的访问量是妥妥够用的了。

`Redis`号称可以支撑`10w+qps`，当然这也给机器配置有一定的关系，如果单实例满足不了需求，想追求更高的性能和稳定性，可以选择主从、哨兵已经更好的解决方案`Redis-Cluster` 集群。

### **架构**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721000238979-000239.jpg)

### **集群部署**

如题，我们这里直接使用 `Redis-Cluster` 高可用集群。

`Redis-Cluster`集群采用无中心结构,它的特点如下：

* 所有的redis节点彼此互联\(PING-PONG机制\)，内部使用二进制协议优化传输速度和带宽。
* 节点的fail是通过集群中超过半数的节点检测失效时才生效。
* 客户端与redis节点直连，不需要中间代理层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。

生产环境`redis`集群至少需要一个备份节点，才能更好的保证集群的高可用。一个集群里面有 M1-S1、M2-S2、M3-S3 六个主从节点，其中节点 M1 包含 0-5500号哈希槽，节点 M2 包含5501-11000 号哈希槽，节点 M3 包含11001-16384号哈希槽。如果是 M1 宕掉，集群便会选举S1 为新节点继续服务，整个集群还会正常运行。当 M1、S1 都宕掉了，这时候集群就不可用了。

建议使用至少六台单独的服务器进行部署，这里为了搭建方便，撸主在一台服务器上进行实操。

下载最新版本：

```text
wget http://download.redis.io/releases/redis-6.0.5.tar.gz
```

升级基础组件，否则编译安装会报错：

```text
# 升级到gcc 9.3：
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
# 需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。
# 如果要长期使用gcc 9.3的话：
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

编译安装：

```text
make && make install
```

创建集群配置：

```text
mkdir redis-cluster
```

创建集群文件夹，然后在该文件夹下创建6个文件，同`redis.conf`配置一致，`7001.conf、7002.conf、7003.conf、7004.conf、7005.conf、7006.conf`

然后修改各个配置：

```text
port  700x                             // 端口700X
bind 本机ip                             // 需要改为其他节点机器可访问的ip 否则创建集群时无法访问对应的端口，无法创建集群
daemonize    yes                       // redis后台运行
pidfile  /var/run/redis_7001.pid       // pidfile文件对应7001,7002
cluster-enabled  yes                   // 开启集群  把注释#去掉
cluster-config-file  nodes_7001.conf   // 集群的配置  配置文件首次启动自动生成 7001,7002
cluster-node-timeout  15000            // 请求超时  默认15秒，可自行设置
appendonly  yes    // # AOF 持久化，按需要选择是否开启
```

启动服务：

```text
redis-server ../redis-cluster/7001.conf
redis-server ../redis-cluster/7002.conf
redis-server ../redis-cluster/7003.conf
redis-server ../redis-cluster/7004.conf
redis-server ../redis-cluster/7005.conf
redis-server ../redis-cluster/7006.conf
```

创建集群：

```text
--replicas参数指定集群中每个主节点配备几个从节点，这里设置为1。
redis-cli --cluster create 172.17.1.218:7001 172.17.1.218:7002 172.17.1.218:7003 172.17.1.218:7004 172.17.1.218:7005 172.17.1.218:7006 --cluster-replicas 1
```

查看集群：

```text
redis-cli --cluster check 172.17.1.218:7001
172.17.1.218:7002 (406a1d6e...) -> 0 keys | 5461 slots | 1 slaves.
172.17.1.218:7003 (85df8dc9...) -> 0 keys | 5462 slots | 1 slaves.
172.17.1.218:7004 (54f13720...) -> 2 keys | 5461 slots | 1 slaves.
[OK] 2 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 172.17.1.218:7001)
S: 2d589d0077771ca9a7b2909b351e350aa69bea45 172.17.1.218:7001
   slots: (0 slots) slave
   replicates 54f13720da944d3dcc75351c4042fcaf3b2300e0
M: 406a1d6e9bcc760501c6b8f71f9151e020b247bd 172.17.1.218:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: e52eda05e4df779e15437f7f42522459a3e26b27 172.17.1.218:7005
   slots: (0 slots) slave
   replicates 85df8dc9b649e02740fa0f932aa1df135d44953e
M: 85df8dc9b649e02740fa0f932aa1df135d44953e 172.17.1.218:7003
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 54f13720da944d3dcc75351c4042fcaf3b2300e0 172.17.1.218:7004
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 60f49255085936fbcf682bafc81319f9393150c9 172.17.1.218:7006
   slots: (0 slots) slave
   replicates 406a1d6e9bcc760501c6b8f71f9151e020b247bd
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

从集群信息，我们可以看出有三组主从关系：

* 7004\(主\)-7001\(从\)
* 7003\(主\)-7005\(从\)
* 7002\(主\)-7006\(从\)

其实一开始7001是主，为了测试主从切换，把7001杀掉，由7004接管，然后重启7001变为从服务。

### **整合**

`pom.xml` 引入：

```text
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置文件 `bootstrap.yml`：

```text
spring:
  redis:
    timeout: 30000
    password: 123456
    cluster:
      nodes:
        - 172.17.1.218:7001
        - 172.17.1.218:7002
        - 172.17.1.218:7003
        - 172.17.1.218:7004
        - 172.17.1.218:7005
        - 172.17.1.218:7006
      max-redirects: 3
    lettuce:
      max-active: 1000
      max-idle: 10
      max-wait: -1
      min-idle: 5
```

开发者可以对 `RedisTemplate` 进行一个简单的封装成 `RedisUtil`，可参考妹子图缓存工具类。

### 小结

这就是微服务版本的 `Redis` 高可用集群，是不是有点简单。不过，生产环境建议配置内网地址，开启防火墙，配置必要的鉴权访问机制，这玩意一旦被嗅探到可以会被老板扣工资的。还有一点，一定要搭建有效的监控预警系统，这样可以针对性的对其进行扩容、修复。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721000239032-000239.jpg)

▲扫一扫回复【妹子图】获取源码

