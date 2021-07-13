# 搭建ELK日志平台  FileBeat + Kafka + Logstash + Kibana



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

# Each - is an input. Most options can be set at the input level, so
# you can use different inputs for various configurations.
# Below are the input specific configurations.

- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /usr/local/cnsaep/log/*.log
    
#================================ Outputs =====================================

# Configure what output to use when sending the data collected by the beat.

#-------------------------- kafka output ------------------------------
output.kafka:
  hosts: ["172.17.25.157:9092"]
  topic: cnsaep-server-log
  required_acks: 1

```

#### 1.3 开启使用模块

```sh
./filebeat modules enable system nginx mysql kafka
```



#### 1.4  启动Filebeat

``` sh
./filebeat -c -e filebeat.yml
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
./kafka-server-start.sh -daemon ../config/server.properties
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

### 4. docker环境安装logstash

#### 4.1 安装logstash

```sh
#拉取镜像
docker pull logstash:7.13.3
```

#### 4.2 启动logstash  端口号 5044

```sh

```

#### 4.3 修改配置文件

```yml

```

#### 4.4 重新启动logstash



### 6.  出现问题及解决方案

#### 1. 内存不足

> Error occurred during initialization of VM Initial heap size set to a larger value than the maximum heap size

1. 原因： 内存不足，修改 kafka-server-start.sh  heap内存大小

2. 解决方案： 找到指定的jvm.options 修改

   ```sh
   find / -name jvm.options
   ```









