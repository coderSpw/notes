# 搭建ELK日志平台  FileBeat + Kafka + Logstash + Kibana



> 下载elk文件官网：
>
> https://www.elastic.co/cn/downloads/past-releases#elasticsearch



### 1. linux环境下安装fileBeat

#### 1.1. 安装Filebeat

```sh
#下载filebeat
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.3.0-linux-x86_64.tar.gz
#解压
tar -zxvf filebeat-7.3.0-linux-x86_64.tar.gz
```

#### 1.2 修改filebeat.yml 配置文件  **输出 -> kafka**

```yml
#=========================== Filebeat inputs =============================
filebeat.inputs:
- type: log
  paths:
    - /usr/local/cnsaep/log/cnsaep.log
  json.keys_under_root: true
  json.add_error_key: true
  json.message_key: log
  
#================================ Outputs =====================================
# ---------------------------- console  Output ----------------------------
#output.console:
  #pretty: true

# ---------------------------- kafka Output ----------------------------
output.kafka:
  hosts: ["47.98.184.103:9092"]
  topic: "elk_log"
```

#### 1.3  启动Filebeat

``` sh
nohup ./filebeat -c -e filebeat.yml &
```



### 2. docker环境安装zookeeper

#### 2.1 拉取zookeeper镜像

```sh
#搜索zk镜像 或从dockerhub中搜索
docker search zookeeper
#拉取镜像
docker pull zookeeper
```

#### 2.2 启动zk容器

```sh
#启动容器
docker run -di --name zookeeper -p 2181:2181  imgId
```

#### 2.3 验证启动是否成功

```sh
#查看运行中的容器
docker ps 
#若没启动成功 查看所有容器 找到zk容器并查看日志
docker ps -a
docker logs containId
```



### 3. linux环境下安装kafka

> 因服务器内存太小，docker安装要修改内存大小 ，暂时linux环境本地安装使用

#### 3.1  安装kafka

```sh
# 官网下载 tgz
```

#### 3.2 启动kafka  端口号 9092

```sh
#后台启动启动kafka
nohup ./bin/kafka-server-start.sh ./config/server.properties &
```

#### 3.3 验证安装是否成功

```sh
#新建topic 
 ./kafka-topics.sh --create --bootstrap-server 172.17.25.157:9092 --topic cnsaep-server-log --partitions 1 --replication-factor 1
#zk查看topic是否存在
ls /brokers/topics
#启动生产者
./kafka-console-producer.sh --bootstrap-server 172.17.25.157:9092 --topic cnsaep-server-log
#启动消费者
 ./kafka-console-consumer.sh --bootstrap-server 172.17.25.157:9092 --topic cnsaep-server-log --from-beginning
```

#### 

### 4. linux环境安装logstash

#### 4.1 官网下载

#### 4.2 修改配置文件

1. 新建 etc/conf.d目录

2. input.conf

   ```sh
   input{
     kafka {
       type => "elk_kafka"
       codec => "json"
       topics => "elk_log"
       decorate_events => true
       bootstrap_servers => "47.98.184.103:9092"
     }
   }
   ```

3. output.conf

   ```sh
   output {
     if[type] == "elk_kafka" {
       stdout {
         codec => "json"
       }
     }
   }
   ```

#### 4.3 启动logstash

```sh
nohup ./logstash -f ../etc/conf.d/ --config.reload.automatic &
```

#### 

### 5. Linux环境安装elasticsearch

#### 5.1 官网下载

#### 5.2 调整jvm.options大小 

``` sh

```







### 6.  出现问题及解决方案

#### 1. 内存不足

> Error occurred during initialization of VM Initial heap size set to a larger value than the maximum heap size

1. 原因： 内存不足，修改 kafka-server-start.sh  heap内存大小

2. 解决方案： 找到指定的jvm.options 修改

   ```sh
   find / -name jvm.options
   ```









