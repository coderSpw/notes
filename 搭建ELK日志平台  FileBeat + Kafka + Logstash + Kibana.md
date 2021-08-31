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
     # 文件读取
     #file {
      #path => "/usr/local/cnsaep/log/cnsaep.log"
      #start_position => beiginning
     #}
   }
   ```

3. output.conf

   ```sh
   output {
     if[type] == "elk_kafka" {
       stdout {
         codec => "json"
       }
       elasticsearch {
        host => "47.98.184.103:9200"
        index => "logstash-cnsaep-%{+YYYY-MM-dd}"
      }
     }
   }
   ```

#### 4.3 启动logstash

```sh
nohup ./logstash -f ../etc/conf.d/ --config.reload.automatic &
```

#### 

### 5. docker环境安装elasticsearch

#### 5.1 dockerHub拉取对应版本镜像

```sh
docker pull elasticsearch:7.13.3
```

#### 5.2 调整jvm.options大小 

``` sh
-Xms1g
-Xmx1g
```

#### 5.3 启动es容器

```sh
docker run -di --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  imgId
```

#### 5.4 安装head插件并启动

```sh
docker pull mobz/elasticsearch-head:5-alpine
#启动
docker run -di --name es-head -p 9100:9100 imgId
```



### 6. docker环境安装kibana

#### 6.1.拉取镜像

```sh
docker pull kibana:7.13.3
```

#### 6.2 修改kibana.yml配置文件

```yml
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://47.98.184.103:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
```

#### 6.3 启动容器 默认端口5601

```sh
docker run -di --name kibana -p 5601:5601 imgId
```



#### 6.4 启动成功界面展示

![image-20210715125232638](C:/Users/s5536/AppData/Roaming/Typora/typora-user-images/image-20210715125232638.png)



### 7.  出现问题及解决方案

#### 7.1. 内存不足

> Error occurred during initialization of VM Initial heap size set to a larger value than the maximum heap size
>
> 1. 原因： 内存不足，修改 kafka-server-start.sh  heap内存大小
>
> 2. 解决方案： 找到指定的jvm.options 修改

```sh
find / -name jvm.options
```



#### 7.2 使用elasticsearch使用head插件跨域

> 使用head插件连接es出现跨域    
>
> 在elasticsearch.yml 文件中新增跨域参数
>
> ==**http.cors.enabled: true**==
> ==**http.cors.allow-origin: "*"**==



#### 7.3 使用head插件操作es时出现406 Not Acceptable

![image-20210715113635278](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210715113635278.png)

操作方法：

1. 查找vendor.js   相对路径：/app/_site/vendor.js

   ```sh
   find / -name vendor.js
   ```

 2. 修改文件内容

    ```text
    ①. 6886行   contentType: "application/x-www-form-urlencoded
    
    改成
    
    contentType: "application/json;charset=UTF-8"
    
    ②. 7574行 var inspectData = s.contentType === "application/x-www-form-urlencoded" &&
    
    改成
    
    var inspectData = s.contentType === "application/json;charset=UTF-8" &&
    
    # 若是docker重启
    docker restart -t=60 containId
    ```

    
