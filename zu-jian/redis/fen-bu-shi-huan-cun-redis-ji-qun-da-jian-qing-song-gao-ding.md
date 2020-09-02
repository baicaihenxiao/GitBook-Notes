# 分布式缓存 Redis 集群搭建轻松搞定！

[https://mp.weixin.qq.com/s/G7who0MqYEPNmVOlcJgR\_A](https://mp.weixin.qq.com/s/G7who0MqYEPNmVOlcJgR_A)

作者 \| Esofar

来源 \| cnblogs.com/esofar/p/10486621.html

## Redis 集群简介

Redis Cluster 即 Redis 集群，是 Redis 官方在 3.0 版本推出的一套分布式存储方案。完全去中心化，由多个节点组成，所有节点彼此互联。Redis 客户端可以直接连接任何一节点获取集群中的键值对，不需要中间代理，如果该节点不存在用户所指定的键值，其内部会自动把客户端重定向到键值所在的节点。

Redis 集群是一个网状结构，每个节点都通过 TCP 连接跟其他每个节点连接。在一个有 N 个节点的集群中，每个节点都有 N-1 个流出的 TCP 连接，和 N-1 个流入的连接，这些 TCP 连接会永久保持。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/09/02/640-20200902221232587-221232.jpg)

Redis Cluster 同其他分布式存储系统一样，主要具备以下两个功能：

**数据分区**Redis 集群会将用户数据分散保存至各个节点中，突破单机 Redis 内存最大存储容量。集群引入了 哈希槽`slot`的概念，其搭建完成后会生 16384 个哈希槽`slot`，同时会根据节点的数量大致均等的将 16384 个哈希槽映射到不同的节点上。当用户存储`key-value`时，集群会先对`key`进行 CRC16 校验然后对 16384 取模来决定`key-value`放置哪个槽，从而实现自动分割数据到不同的节点上。

**数据冗余**Redis 集群支持主从复制和故障恢复。集群使用了主从复制模型，每个主节点`master`应至少有一个从节点`slave`。假设某个主节点故障，其所有子节点会广播一个数据包给集群里的其他主节点来请求选票，一旦某个从节点收到了大多数主节点的回应，那么它就赢得了选举，被推选为主节点，负责处理之前旧的主节点负责的哈希槽。

> 关于 Redis Cluster 详细介绍以及实现原理请参见 Redis Cluster 教程 和 Redis Cluster 规范，在此不再赘述。

## 下载 & 安装 Redis

> 实验环境信息 Linux 版本：CentOS Linux release 7.4.1708 Redis 版本：5.0.3

先在服务器或虚拟机中安装一个单机 Redis，如果已安装可以跳过本节，未安装过的正好学习下。

进入 Redis 待安装目录。

```text
Copycd /usr/local
```

下载、解压 Redis 源代码压缩包。

```text
Copywget http://download.redis.io/releases/redis-5.0.3.tar.gz
tar -zxvf redis-5.0.3.tar.gz
```

然后进入解压后的目录并使用 make 命令执行编译安装 Redis。

```text
Copycd redis-5.0.3
make && make install
```

不要高兴，因为你极有可能会遇到因为 GCC 编译器未安装导致编译失败的情况。不要着急，请顺序执行如下命令。

```text
Copyyum -y install gcc
make distclean
make && make install
```

> Redis 基于 C 语言开发，故编译源码需要 GCC（Linux下的一个编译器，这里需要用来编译`.c`文件）的支持。如机器上未安装需要先执行命令`yum -y install gcc`安装 GCC 编译工具，然后`make distclean`清除之前生成的文件，最后`make && make install`重新编译安装。

最终出现类似下文输出则表示 Redis 安装成功。

```text
Copy......
Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: 离开目录“/usr/local/redis-5.0.3/src”
```

如果源码编译无误且执行结果正确，`make install`命令会将程序安装至系统预设的可执行文件存放路径，一般是`/usr/local/bin`目录，可以通过如下终端输出确认。当然，也可以使用`make install PREFIX=`命令安装到指定路径。

```text
Copy[root@localhost bin]# cd /usr/local/bin
[root@localhost bin]# ls -l
总用量 32672
-rwxr-xr-x. 1 root root 4367328 3月   6 06:11 redis-benchmark
-rwxr-xr-x. 1 root root 8092024 3月   6 06:11 redis-check-aof
-rwxr-xr-x. 1 root root 8092024 3月   6 06:11 redis-check-rdb
-rwxr-xr-x. 1 root root 4802696 3月   6 06:11 redis-cli
lrwxrwxrwx. 1 root root      12 3月   6 06:11 redis-sentinel -> redis-server
-rwxr-xr-x. 1 root root 8092024 3月   6 06:11 redis-server
```

至此，单机 Redis 安装完成。

## 搭建 Redis 集群

进入正题。

依据 Redis Cluster 内部故障转移实现原理，Redis 集群至少需要 3 个主节点，而每个主节点至少有 1 从节点，因此搭建一个集群至少包含 6 个节点，三主三从，并且分别部署在不同机器上。

条件有限，测试环境下我们只能在一台机器上创建一个伪集群，通过不同的 TCP 端口启动多个 Redis 实例，组成集群。

目前 Redis Cluster 的搭建有两种方式：

1. 手动方式搭建，即手动执行 cluster 命令，一步步完成搭建流程。
2. 自动方式搭建，即使用官方提供的集群管理工具快速搭建。

两种方式原理一样，自动搭建方式只是将手动搭建方式中需要执行的 Redis 命令封装到了可执行程序。生产环境下推荐使用第二种方式，简单快捷，不易出错。不过本文实战演示两种方式都会提及。

### 手动方式搭建

#### 启动节点

搭建集群的第一步就是要先把参与搭建集群的每个节点启动起来。

由于我们这是在一台机器上模拟多个节点，可以预先规划下各个节点的属性：

| 节点编号 | IP 地址 | TCP 端口 | 节点类型 | 从节点 | 启动配置 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| A | 127.0.0.1 | 7001 | 主 | D | /usr/local/redis-cluster/7001/redis.conf |
| B | 127.0.0.1 | 7002 | 主 | E | /usr/local/redis-cluster/7002/redis.conf |
| C | 127.0.0.1 | 7003 | 主 | F | /usr/local/redis-cluster/7003/redis.conf |
| D | 127.0.0.1 | 8001 | 从 | / | /usr/local/redis-cluster/8001/redis.conf |
| E | 127.0.0.1 | 8002 | 从 | / | /usr/local/redis-cluster/8002/redis.conf |
| F | 127.0.0.1 | 8003 | 从 | / | /usr/local/redis-cluster/8003/redis.conf |

根据上述规划，可以先通过如下命令创建各个节点启动配置文件的存放目录。

```text
Copymkdir /usr/local/redis-cluster
cd redis-cluster
mkdir -p 7001 7002 7003 8001 8002 8003
```

顺序执行如下行命令，进入 Redis 源码包目录并将默认配置文件`redis.conf`分别复制到六个节点配置存放目录中，作为各自节点启动配置文件。

```text
Copycd /usr/local/redis-5.0.3
cp redis.conf /usr/local/redis-cluster/7001
cp redis.conf /usr/local/redis-cluster/7002
cp redis.conf /usr/local/redis-cluster/7003
cp redis.conf /usr/local/redis-cluster/8001
cp redis.conf /usr/local/redis-cluster/8002
cp redis.conf /usr/local/redis-cluster/8003
```

接下来需要分别修改每个节点的配置文件。下面贴的是节点 A 的配置文件`/usr/local/redis-cluster/7001/redis.conf`中启用或修改的一些必要参数。其他节点 B、C、D、E、F 参照修改，注意把涉及端口的地方修改成各自节点预先规划的即可。

```text
Copybind 192.168.83.128                    # 设置当前节点主机地址
port 7001                              # 设置客户端连接监听端口
pidfile /var/run/redis_7001.pid        # 设置 Redis 实例 pid 文件
daemonize yes                          # 以守护进程运行 Redis 实例
cluster-enabled yes                    # 启用集群模式
cluster-node-timeout 15000             # 设置当前节点连接超时毫秒数
cluster-config-file nodes-7001.conf    # 设置当前节点集群配置文件路径
```

完成上述工作就可以通过如下几组命令启动待搭建集群中的 6 个节点了。

```text
Copy/usr/local/bin/redis-server /usr/local/redis-cluster/7001/redis.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/7002/redis.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/7003/redis.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/8001/redis.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/8002/redis.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/8003/redis.conf
```

最后通过`ps -ef|grep redis`命令确认各个节点服务是否已经正常运行。

```text
Copy[root@localhost bin]# ps -ef|grep redis
root       5613      1  0 04:25 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:7001 [cluster]
root       5650      1  0 04:26 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:7002 [cluster]
root       5661      1  0 04:26 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:7003 [cluster]
root       5672      1  0 04:27 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:8001 [cluster]
root       5681      1  0 04:27 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:8002 [cluster]
root       5690      1  0 04:27 ?        00:00:00 /usr/local/bin/redis-server 127.0.0.1:8003 [cluster]
root       5731   1311  0 04:28 pts/0    00:00:00 grep --color=auto redis
```

如上输出可以看出上面规划的 6 个节点都成功启动。

#### 节点握手

虽然上面 6 个节点都启用了群集支持，但默认情况下它们是不相互信任或者说没有联系的。节点握手就是在各个节点之间创建链接（每个节点与其他节点相连），形成一个完整的网格，即集群。

节点握手的命令如下：

```text
Copycluster meet ip port
```

但为了创建群集，不需要发送形成完整网格所需的所有 cluster meet 命令。只要能发送足够的`cluster meet`消息，可以让每个节点都可以通过一系列已知节点到达每个其他节点，缺失的链接将被自动创建。

例如，如果我们通过`cluster meet`将节点 A 与节点 B 连接起来，并将 B 与 C 连接起来，则 A 和 C 会自己找到握手方式并创建链接。

我们的创建的 6 个节点可以通过 redis-cli 连接到 A 节点执行如下五组命令完成握手，生产环境需要将 IP `127.0.0.1`替换成外网 IP。

```text
Copycluster meet 127.0.0.1 7002
cluster meet 127.0.0.1 7003
cluster meet 127.0.0.1 8001
cluster meet 127.0.0.1 8002
cluster meet 127.0.0.1 8003
```

如上述命令正常执行输出结果如下。

```text
Copy[root@localhost bin]# /usr/local/bin/redis-cli -p 7001
127.0.0.1:7001> cluster meet 127.0.0.1 7002
OK
127.0.0.1:7001> cluster meet 127.0.0.1 7003
OK
127.0.0.1:7001> cluster meet 127.0.0.1 8001
OK
127.0.0.1:7001> cluster meet 127.0.0.1 8002
OK
127.0.0.1:7001> cluster meet 127.0.0.1 8003
OK
```

接下来可以通过 cluster nodes 命令查看节点之间 的链接状态。我随机找了两个节点 B 和 F 测试，输出结果如下所示。

```text
Copy[root@localhost /]# /usr/local/bin/redis-cli -p 7002 cluster nodes
61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12 127.0.0.1:7001@17001 master - 0 1552220691885 4 connected
a8a41694f22977fda78863bdfb3fc03dd1fab1bd 127.0.0.1:8002@18002 master - 0 1552220691000 5 connected
51987c4b5530c81f2845bb9d521daf6d3dce3659 127.0.0.1:8001@18001 master - 0 1552220690878 3 connected
1b4b3741945d7fed472a1324aaaa6acaa1843ccb 127.0.0.1:7002@17002 myself,master - 0 1552220690000 1 connected
19147f56e679767bcebb8653262ff7f56ca072a8 127.0.0.1:7003@17003 master - 0 1552220691000 2 connected
ed6fd72e61b747af3705b210c7164bc68739303e 127.0.0.1:8003@18003 master - 0 1552220690000 0 connected
[root@localhost /]# /usr/local/bin/redis-cli -p 8002 cluster nodes
1b4b3741945d7fed472a1324aaaa6acaa1843ccb 127.0.0.1:7002@17002 master - 0 1552220700255 1 connected
ed6fd72e61b747af3705b210c7164bc68739303e 127.0.0.1:8003@18003 master - 0 1552220703281 0 connected
19147f56e679767bcebb8653262ff7f56ca072a8 127.0.0.1:7003@17003 master - 0 1552220700000 2 connected
a8a41694f22977fda78863bdfb3fc03dd1fab1bd 127.0.0.1:8002@18002 myself,master - 0 1552220701000 5 connected
61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12 127.0.0.1:7001@17001 master - 0 1552220702275 4 connected
51987c4b5530c81f2845bb9d521daf6d3dce3659 127.0.0.1:8001@18001 master - 0 1552220701265 3 connected
```

可以看到，节点 B 和节点 F 都已经分别和其他 5 个节点建立链接。

至此，节点握手完成。

#### 分配槽位

此时 Redis 集群还并没有处于上线状态，可以在任意一节点上执行 cluster info 命令来查看目前集群的运行状态。

```text
Copy[root@localhost ~]# /usr/local/bin/redis-cli -p 7001 cluster info
cluster_state:fail
......
```

上面输出`cluster_state:fail`表示当前集群处于下线状态。因为只有给集群中所有**主节点**分配好槽位（即哈希槽`slot`，本文第一小节有提及）集群才能上线。

分配槽位的命令如下：

```text
Copycluster addslots slot [slot ...]
```

根据预先规划，这一步需要使用 cluster addslots 命令手动将 16384 个哈希槽大致均等分配给**主节点** A、B、C。

```text
Copy/usr/local/bin/redis-cli -p 7001 cluster addslots {0..5461}
/usr/local/bin/redis-cli -p 7002 cluster addslots {5462..10922}
/usr/local/bin/redis-cli -p 7003 cluster addslots {10923..16383}
```

上面三组命令执行完毕，可以再次查看目前集群的一些运行参数。

```text
Copy[root@localhost ~]# /usr/local/bin/redis-cli -p 7001 cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:5
cluster_my_epoch:4
cluster_stats_messages_ping_sent:11413
cluster_stats_messages_pong_sent:10509
cluster_stats_messages_meet_sent:11
cluster_stats_messages_sent:21933
cluster_stats_messages_ping_received:10509
cluster_stats_messages_pong_received:10535
cluster_stats_messages_received:21044
```

如上输出`cluster_state:ok`证明 Redis 集群成功上线。

#### 主从复制

Redis 集群成功上线，不过还没有给主节点指定从节点，此时如果有一个节点故障，那么整个集群也就挂了，也就无法实现高可用。

集群中需要使用 cluster replicate 命令手动给从节点配置主节点。

集群复制命令如下：

```text
Copycluster replicate node-id
```

集群中各个节点的`node-id`可以用`cluster nodes`命令查看，如下输出`1b4b3741945d7fed472a1324aaaa6acaa1843ccb`即是主节点 B 的`node-id`。

```text
Copy[root@localhost /]# /usr/local/bin/redis-cli -p 8002 cluster nodes
1b4b3741945d7fed472a1324aaaa6acaa1843ccb 127.0.0.1:7002@17002 master - 0 1552220700255 1 connected
ed6fd72e61b747af3705b210c7164bc68739303e 127.0.0.1:8003@18003 master - 0 1552220703281 0 connected
19147f56e679767bcebb8653262ff7f56ca072a8 127.0.0.1:7003@17003 master - 0 1552220700000 2 connected
a8a41694f22977fda78863bdfb3fc03dd1fab1bd 127.0.0.1:8002@18002 myself,master - 0 1552220701000 5 connected
61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12 127.0.0.1:7001@17001 master - 0 1552220702275 4 connected
51987c4b5530c81f2845bb9d521daf6d3dce3659 127.0.0.1:8001@18001 master - 0 1552220701265 3 connected
```

根据预先规划，A主D从；B主E从；C主F从。执行如下三组命令分别为从节点 D、E、F 指定其主节点，使群集可以自动完成主从复制。

```text
Copy/usr/local/bin/redis-cli -p 8001 cluster replicate 61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12
/usr/local/bin/redis-cli -p 8002 cluster replicate 1b4b3741945d7fed472a1324aaaa6acaa1843ccb
/usr/local/bin/redis-cli -p 8003 cluster replicate 19147f56e679767bcebb8653262ff7f56ca072a8
```

命令执行成功后，我们便算以手动方式成功搭建了一个 Redis 集群。

最后，再来查看一下集群中的节点信息。

```text
Copy[root@localhost ~]# /usr/local/bin/redis-cli -p 8002 cluster nodes
1b4b3741945d7fed472a1324aaaa6acaa1843ccb 127.0.0.1:7002@17002 master - 0 1552233328337 1 connected 5462-10922
ed6fd72e61b747af3705b210c7164bc68739303e 127.0.0.1:8003@18003 slave 19147f56e679767bcebb8653262ff7f56ca072a8 0 1552233327000 2 connected
19147f56e679767bcebb8653262ff7f56ca072a8 127.0.0.1:7003@17003 master - 0 1552233325000 2 connected 10923-16383
a8a41694f22977fda78863bdfb3fc03dd1fab1bd 127.0.0.1:8002@18002 myself,slave 1b4b3741945d7fed472a1324aaaa6acaa1843ccb 0 1552233327000 5 connected
61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12 127.0.0.1:7001@17001 master - 0 1552233327327 4 connected 0-5461
51987c4b5530c81f2845bb9d521daf6d3dce3659 127.0.0.1:8001@18001 slave 61e8c4ed8d1ff2a765a4dd2c3d300d8121d26e12 0 1552233326320 4 connected
```

### 自动方式搭建

Redis 3.0 版本之后官方发布了一个集群管理工具 redis-trib.rb，集成在 Redis 源码包的`src`目录下。其封装了 Redis 提供的集群命令，使用简单、便捷。

不过 redis-trib.rb 是 Redis 作者使用 Ruby 语言开发的，故使用该工具之前还需要先在机器上安装 Ruby 环境。后面作者可能意识到这个问题，Redis 5.0 版本开始便把这个工具集成到 redis-cli 中，以`--cluster`参数提供使用，其中`create`命令可以用来创建集群。

#### 启动节点

使用集群管理工具搭建集群之前，也是需要先把各个节点启动起来的。节点的启动方式请参见本文「手动方式创建」-「启动节点」一节，此处不再赘述。

#### 集群管理工具搭建

如果您安装的 Redis 是 3.x 和 4.x 的版本可以使用 redis-trib.rb 搭建，不过之前需要安装 Ruby 环境。

先使用 yum 安装 Ruby 环境以及其他依赖项。

```text
Copyyum -y install ruby ruby-devel rubygems rpm-build
```

确认安装版本。

```text
Copy[root@localhost redis-cluster]# ruby -v
ruby 2.0.0p648 (2015-12-16) [x86_64-linux]
```

再使用 redis-trib.rb 脚本搭建集群，具体命令如下所示。

```text
Copy/usr/local/redis-5.0.3/src/redis-trib.rb create --replicas 1 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003
```

不过，本文实验环境使用的 Redis 版本是 5.0.3，所以我可以直接使用`redis-cli --cluster create`命令搭建，具体命令如下所示。

```text
Copy/usr/local/bin/redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 --cluster-replicas 1
```

主节点在前，从节点在后。其中`--cluster-replicas`参数用来指定一个主节点带有的从节点个数，如上`--cluster-replicas 1`即表示 1 个主节点有 1 个从节点。

命令执行成功会有类似如下输出。

```text
Copy[root@localhost bin]# redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:8001 127.0.0.1:8002 127.0.0.1:8003 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:8001 to 127.0.0.1:7001
Adding replica 127.0.0.1:8002 to 127.0.0.1:7002
Adding replica 127.0.0.1:8003 to 127.0.0.1:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 32f9819fc7d561bfa2b7189182200e86d9901b8a 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
M: cca0fbfa374bc175d481e68ee9ed13b65453e967 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
M: 964cfa1c2dcfe36b6d3c63637f0d57ccb568354e 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
S: 1b47b9e6e7a79523579b8d2ddcd5e708583ed317 127.0.0.1:8001
   replicates 32f9819fc7d561bfa2b7189182200e86d9901b8a
S: aba9330f3e70f26a8af4ced1b672fbcc7bc62d78 127.0.0.1:8002
   replicates cca0fbfa374bc175d481e68ee9ed13b65453e967
S: 254db0830cd764e075aa793144572d5fa3a398f0 127.0.0.1:8003
   replicates 964cfa1c2dcfe36b6d3c63637f0d57ccb568354e
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: 32f9819fc7d561bfa2b7189182200e86d9901b8a 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: aba9330f3e70f26a8af4ced1b672fbcc7bc62d78 127.0.0.1:8002
   slots: (0 slots) slave
   replicates cca0fbfa374bc175d481e68ee9ed13b65453e967
S: 1b47b9e6e7a79523579b8d2ddcd5e708583ed317 127.0.0.1:8001
   slots: (0 slots) slave
   replicates 32f9819fc7d561bfa2b7189182200e86d9901b8a
S: 254db0830cd764e075aa793144572d5fa3a398f0 127.0.0.1:8003
   slots: (0 slots) slave
   replicates 964cfa1c2dcfe36b6d3c63637f0d57ccb568354e
M: cca0fbfa374bc175d481e68ee9ed13b65453e967 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 964cfa1c2dcfe36b6d3c63637f0d57ccb568354e 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

OK，搭建完成！一条命令搞定。

