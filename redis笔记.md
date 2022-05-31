redis笔记



## 一、介绍

> Redis是一个开源（BSD许可）的内存数据结构存储，用作数据库、缓存和消息代理。Redis提供数据结构，如字符串、哈希、列表、集合、带范围查询的排序集合、位图、hyperloglogs、地理空间索引和流。Redis具有内置复制、Lua脚本、LRU逐出、事务和不同级别的磁盘持久性，并通过Redis Sentinel和Redis Cluster的自动分区提供高可用性



## 二、Linux环境集群搭建

### 1. 集群架构图

![image-20210910113742365](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210910113742365.png)



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

  ![image-20210910113835311](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210910113835311.png)

- 从：

  ![image-20210909162738539](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210909162738539.png)



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

![image-20210909211748556](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210909211748556.png)

![image-20210909211826405](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210909211826405.png)

![image-20210909211852014](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210909211852014.png)

![image-20210909211931786](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210909211931786.png)



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

![image-20210910113924066](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210910113924066-16498401412591.png)

![image-20210910114008808](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20210910114008808.png)





## 三、redis客户端使用

> #### 基**本数据**类型： Strings、Lists、Sets、SortedSets、Hashes
>
> #### 高级数据类型： hyperLogLog、Geo、bitMaps
>
> ### 集群模式下不支持切换database，且无法批量添加mset...

#### 1. String类型基本使用

```shell
#添加元素
set num 1
#根据key获取value
get num
#自增1
incr num
#自增指定长度 
incr num 10
#自减1
decr num
#自减指定长度
decr num 10
#获取元素长度
strlen name
#截取元素
getrange name 0 2
#替换/拼接元素
setrange name 0 haha  
#删除key
del name
#获取并删除
getdel name 
#添加元素 分布式锁
setnx name haha
#自定义添加元素   NX:存在添加失败， XX：不存在添加失败
set name haha EX seconds PX millseconds  NX|XX
```

#### 2. List类型基本使用

```shell
#创建集合添加元素 入栈操作
rpush list a b c d e f 1 2 3 4
lpush list a b c d e f 1 2 3 4
#从集合中移除元素 出栈操作  lpop key  count
lpop list 2 
rpop list 2
#查看集合元素个数
llen list 
#截取数组元素
lrange list 0 -1 
#指定位置新增元素 linsert key before|after  target element 
linsert list before 1 h 
#集合去重
ltrim list 0 -1 
#从集合中移除元素  lrem list count element 
lrem list 2 2 
#从集合中根据索引获取元素
lindex list 2 
#指定索引出设置元素
lset list 2 e 
```

#### 3. set类型基本使用

```shell

```

