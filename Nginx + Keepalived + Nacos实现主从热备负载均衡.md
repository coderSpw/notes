#  Nginx + Keepalived + Nacos实现主从热备负载均衡



## 一、Nginx+ Keepalived部署

### 1. 维基百科介绍

> **Nginx**（发音同“engine X”）是异步框架的[网页服务器](https://zh.wikipedia.org/wiki/網頁伺服器)，也可以用作[反向代理](https://zh.wikipedia.org/wiki/反向代理)、[负载平衡器](https://zh.wikipedia.org/wiki/负载均衡)和[HTTP缓存](https://zh.wikipedia.org/wiki/HTTP缓存)。
>
> 一大部分Web服务器使用Nginx[[9\]](https://zh.wikipedia.org/wiki/Nginx#cite_note-9)，通常作为[负载均衡器](https://zh.wikipedia.org/wiki/负载均衡)。
>
> Nginx是一款面向性能设计的HTTP服务器，相较于[Apache](https://zh.wikipedia.org/wiki/Apache_HTTP_Server)、[lighttpd](https://zh.wikipedia.org/wiki/Lighttpd)具有占有[内存](https://zh.wikipedia.org/wiki/内存)少，稳定性高等优势

> Keepalived是基于vrrp协议的一款高可用软件。Keepailived有一台主服务器和多台备份服务器，在主服务器和备份服务器上面部署相同的服务配置，使用一个虚拟IP地址对外提供服务，当主服务器出现故障时，**虚拟IP地址会自动漂移**到备份服务器。
>
> VRRP（Virtual Router Redundancy Protocol，虚拟路由器冗余协议），VRRP是为了解决静态路由的高可用。
>
> VRRP的基本架构
> 虚拟路由器由多个路由器组成，每个路由器都有各自的IP和共同的VRID(0-255)，其中一个VRRP路由器通过竞选成为MASTER，占有VIP，对外提供路由服务，其他成为BACKUP，MASTER以IP组播（组播地址：224.0.0.18）形式发送VRRP协议包，与BACKUP保持心跳连接，若MASTER不可用（或BACKUP接收不到VRRP协议包），则BACKUP通过竞选产生新的MASTER并继续对外提供路由服务，从而实现高可用。

### 2. 基于Linux环境部署Nginx

1. 官网下载tar.gz文件

   >  官网下载地址： http://nginx.org/en/download.html

2.  解压

   ```sh
   tar -zxvf nginx-1.13.7.tar.gz
   ```

3. 下载安装相关组件

   ```sh
   #编译依赖gcc环境
   yum install gcc-c++
   #nginx的http模块使用pcre来解析正则表达式
   yum install -y pcre pcre-devel
   #zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip
   yum install -y zlib zlib-devel
   #nginx不仅支持http协议，还支持https（即在ssl协议上传输http）
   yum install -y openssl openssl-devel
   ```

4. 创建makefile文件

   ```sh
   #注意：上边将临时文件目录指定为/var/temp/nginx，需要在/var下创建temp及nginx目录
   ./configure \--prefix=/home/work/nginx \--pid-path=/var/run/nginx/nginx.pid \--lock-path=/var/lock/nginx.lock\--error-log-path=/var/log/nginx/error.log \--http-log-path=/var/log/nginx/access.log \--with-http_gzip_static_module \--http-client-body-temp-path=/var/temp/nginx/client \--http-proxy-temp-path=/var/temp/nginx/proxy \--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \--http-scgi-temp-path=/var/temp/nginx/scgi
   ```

5. 安装

   ```sh
   make & make install
   ```

6. 启动

   ```sh
   ./sbin/nginx
   ```

7.  修改配置文件重新加载

   ```sh
   ./sbin/nginx -s reload
   ```




### 3. 部署Keepalived 实现Nginx集群高可用

1. ​	安装keepalived

   ```sh
   yum install -y keepalived
   ```

2.  编写nginx监控脚本    /etc/keepalived/check_nginx.sh

   ```sh
   #!/bin/bash
   counter=$(ps -C nginx --no-heading|wc -l)
   if [ "${counter}" = "0" ]; then
       /usr/local/bin/nginx
       sleep 2
       counter=$(ps -C nginx --no-heading|wc -l)
       if [ "${counter}" = "0" ]; then
           /etc/init.d/keepalived stop
       fi
   fi
   ```

3. 修改keepalived.conf文件

   - MASTER

     ```sh
     ! Configuration File for keepalived
     global_defs {
         notification_email {
             s553648727@163.com
         }
         notification_email_from coderspw@163.com
         smtp_server  smtp.163.com
         smtp_connect_timeout 30
         router_id LVS_DEVEL
     }
     
     vrrp_script chk_nginx {
         script "/etc/keepalived/check_nginx.sh" 
         interval 2 
         weight -5 
         fall 3  
         rise 2 
     }
     
     vrrp_instance VI_1 {
         state MASTER
         interface eth0
         mcast_src_ip 123.57.86.182
         virtual_router_id 51
         priority 101
         advert_int 2
         authentication {
             auth_type PASS
             auth_pass 123456
         }
         virtual_ipaddress {
             123.57.86.182
             47.98.184.103
             39.106.1.193
         }
         track_script {
            chk_nginx 
         }
     }
     ```

   - BACKUP

     ```sh
     ! Configuration File for keepalived
     global_defs {
         notification_email {
             s553648727@163.com
         }
         notification_email_from coderspw@163.com
         smtp_server  smtp.163.com
         smtp_connect_timeout 30
         router_id LVS_DEVEL
     }
     
     vrrp_script chk_nginx {
         script "/etc/keepalived/check_nginx.sh" 
         interval 2 
         weight -5 
         fall 3  
         rise 2 
     }
     
     vrrp_instance VI_1 {
         state BACKUP
         interface eth0
         mcast_src_ip 47.98.184.103
         virtual_router_id 51
         priority 100
         advert_int 2
         authentication {
             auth_type PASS
             auth_pass 123456
         }
         virtual_ipaddress {
             123.57.86.182
             47.98.184.103
             39.106.1.193
         }
         track_script {
            chk_nginx 
         }
     }
     ```

4. 重启keepalived

   ```sh
   service keepalived restart
   ```

5.  运行成功结果

   ![image-20210831232442276](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831232442276.png)

> **当前测试： 阿里云服务器外网目前不支持keepalived**



### 4. 出现问题

#### 1. 安装时出现：  centOS8 与nginx低版本存在兼容问题 

![image-20210831173241546](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831173241546.png)

解决方法： 注释掉 26行代码

![image-20210831173732123](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831173732123.png)







## 二、 Nacos集群部署

### 1. 介绍 & 特性

- 介绍：

  > Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
  >
  > Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

- 特性：

  - **服务发现和服务健康监测**

  - **动态配置服务**

  - **动态 DNS 服务**

  - **服务及其元数据管理**

    

### 2. 官网下载tar.gz文件

> 官网：https://nacos.io/zh-cn/
>
> 下载地址： https://github.com/alibaba/nacos/releases



### 3. 部署Nacos

1. 解压

```sh
tar -zxvf nacos-server-2.0.3.tar.gz
```

2. 修改application.properties文件 , 持久化方式采用mysql数据库

   ![image-20210831202955240](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831202955240.png)

3. mysql新建nacos数据库，导入conf文件夹下的nacos-mysql.sql

   ![image-20210831203153651](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831203153651.png)

4.  copy cluster.conf 配置集群节点IP+port

   ![image-20210831203252536](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831203252536.png)

5.  修改startup.sh启动文件中内存使用大小 ，根据自己服务器配置设置大小   目前三台服务器都是2c2g  暂时使用512M

   ![image-20210831203439764](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831203439764.png)

6.  分别启动三个节点

   ```sh
   ./startup.sh
   ```

7.  启动成功集群展示界面  登录账号/密码 都为nacos

   ![image-20210831203707498](https://raw.githubusercontent.com/coderSpw/notes/master/note_img/image-20210831203707498.png)

### 4. 集成Nginx

​	   修改nginx.conf 文件 

> **注：文件中命名不能使用下划线**

![image-20210831211426472](C:/Users/s5536/AppData/Roaming/Typora/typora-user-images/image-20210831211426472.png)

