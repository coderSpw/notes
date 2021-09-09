redis笔记



## 一、介绍

> Redis是一个开源（BSD许可）的内存数据结构存储，用作数据库、缓存和消息代理。Redis提供数据结构，如字符串、哈希、列表、集合、带范围查询的排序集合、位图、hyperloglogs、地理空间索引和流。Redis具有内置复制、Lua脚本、LRU逐出、事务和不同级别的磁盘持久性，并通过Redis Sentinel和Redis Cluster的自动分区提供高可用性



## 二、Linux环境集群搭建

### 1. 集群架构图

![image-20210909213356774](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909213356774.png)



### 2.主从搭建

#### 1. 官网下载redis压缩包

> https://redis.io/download

#### 2. 安装redis

```shell
#1.解压
 tar -zxvf redis-6.2.5.tar.gz
#2.重命名
mv redis-6.2.5 redis
#3.安装
make & make install 
```

#### 3. 配置主从修改redis.conf配置文件

```shell
# cp 6379（主） 6380（从） 配置文件
cp redis.conf redis-6379.conf
cp redis.conf redis-6380.conf
```

|              参数              |              6379              |              6380              |
| :----------------------------: | :----------------------------: | :----------------------------: |
|         port（端口号）         |              6379              |              6379              |
|        bind（外网访问）        |            0.0.0.0             |            0.0.0.0             |
|   protected-mode（保护模式）   |               no               |               no               |
|     daemonize（后台运行）      |              yes               |              yes               |
|      pidfile（启动文件）       |    /var/run/redis_6379.pid     |    /var/run/redis_6380.pid     |
|      logfile（日志文件）       | /home/admin/redis/log/6379.log | /home/admin/redis/log/6380.log |
|    replicaof（主库ip+port）    |             不设置             |            ip  port            |
| replica-read-only （从库只读） |                                |              yes               |

#### 4. 启动文件 验证结果

```shell
#启动文件
cd src
./redis-server.sh ../redis-6379.conf
./redis-server.sh ../redis-6380.conf
#客户端进入redis
./redis-cli.sh -p 6379
./redis-cli.sh -p 6379
# redis 命令 查看主从信息
info replication 
```

- 主：

  ![image-20210909162506591](C:/Users/s5536/AppData/Roaming/Typora/typora-user-images/image-20210909162506591.png)

- 从：

  ![image-20210909162738539](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909162738539.png)



### 3. 集群搭建

#### 1. 安装依赖包

``` shell
yum install ruby
yum install rubygems
```

#### 2. 修改配置文件

``` shell
dir /home/admin/redis/6379/data/
cluster-enabled yes
cluster-config-file redis-6379.conf
cluster-node-timeout 15000
cluster-replica-validity-factor 1
cluster-require-full-coverage yes
cluster-replica-no-failover no
```

#### 3. 重启redis服务

#### 4. 使用redis-cli启动集群

``` shell
./redis-cli --cluster create 123.57.86.182:6379 47.98.184.103:6379 39.106.1.193:6379  47.98.184.103:6380 39.106.1.193:6380 123.57.86.182:6380 --cluster-replicas 1
```

#### 5.验证

``` sh
#使用集群方式打开客户端
./redis-cli -c -p 6379
```

![image-20210909211748556](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909211748556.png)

![image-20210909211826405](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909211826405.png)

![image-20210909211852014](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909211852014.png)

![image-20210909211931786](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210909211931786.png)



### 4. 搭建哨兵

#### 1. 修改sentinel.conf配置文件

```shell
port 6381
daemonize yes
pidfile "/var/run/redis-sentinel.pid"
logfile "/home/admin/redis/log/sentinel-103.log"
dir "/home/admin/redis/sentinel"
sentinel monitor mymaster 47.98.184.103 6379 1
```

#### 2. 启动

```shell
./redis-sentinel.sh ../sentinel.conf
```

#### 3. 验证结果