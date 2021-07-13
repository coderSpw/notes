# mysql 主从复制



## docker环境安装mysql

```shell
# 1.搜索mysql镜像 
docker search mysql 
# 2.拉取mysql镜像 或 从https://hub.docker.com/ 上查找对应版本下载
docker pull mysql
# 3.启动mysql  注: 密码不能设置root
docker run -di -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 --name 'mysql'  imageId
# 4.进入容器内
docker exec -ti containId bash
# 5.安装vim,方便操作文件
apt-get update
apt-get install vim
# 6.操作配置文件内容 文件路径 /etc/mysql/mysql.conf.d/mysqld.cnf
```

## linux 安装mysql

```shell
# 1. 拉取mysql安装包
wget http://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
# 2. 解压、移动到usr/local下并重命名为mysql
tar -zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
mv mysql-5.7.17-linux-glibc2.5-x86_64 /usr/local
mv mysql-5.7.17-linux-glibc2.5-x86_64 mysql
# 3. mysql下新建data文件夹
mkdir data
# 4. 查看是否有mysql组，若没有新建
cat /etc/group | grep mysql
groupadd mysql
# 5. 查看是否有mysql用户，没有新建
cat /etc/passwd | grep mysql
useradd -g mysql mysql
# 6. 设置访问权限
chown -R mysql:mysql /usr/local/mysql
# 7. 初始化mysql数据库 执行结束最后会出现初始密码（记得保存）
mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
# 8. 将 mysql 加入服务 
cp /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysql
# 9. 安装msyql依赖包
yum install libncurses*
# 10.设置开机自启mysql
chkconfig --add mysql
chkconfig mysql on
# 11. 关闭防火墙 或 对外暴露3306端口
systemctl stop firewalld.service  # 关闭防火墙
firewall-cmd --state  # 查看防火墙状态
firewall-cmd -add-port=3306/tcp --permanent #对外暴露3306端口
firewall-cmd --reload #刷新防火墙
# 12. 启动mysql服务
service mysql start
# 13. usr/local/mysql 下新增文件etc/my.cnf，并添加相关信息
```

my.cnf 文件具体内容

```shell
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set=utf8

[mysqld] 
#可以免密码登录
skip-grant-tables
###############################基础设置#####################################

#Mysql服务的唯一编号 每个mysql服务Id需唯一
server-id = 1

#服务端口号 默认3306
port = 3306

default_storage_engine=InnoDB

skip-name-resolve

#mysql安装根目录
basedir = /usr/local/mysql

#mysql数据文件所在位置,
datadir =/usr/local/mysql/data

#临时目录 比如load data infile会用到
tmpdir  = /tmp

#设置socke文件所在目录
socket  = /tmp/mysql.sock

#主要用于MyISAM存储引擎,如果多台服务器连接一个数据库则建议注释下面内容
skip-external-locking

#只能用IP地址检查客户端的登录，不用主机名
skip_name_resolve = 1

#事务隔离级别，默认为可重复读，mysql默认可重复读级别（此级别下可能参数很多间隙锁，影响性能）
transaction_isolation = READ-COMMITTED

#数据库默认字符集,主流字符集支持一些特殊表情符号（特殊表情符占用4个字节）
character-set-server = utf8mb4

#数据库字符集对应一些排序等规则，注意要和character-set-server对应
collation-server = utf8mb4_general_ci

#设置client连接mysql时的字符集,防止乱码
#init_connect=‘SET NAMES utf8mb4‘

#是否对sql语句大小写敏感，1表示不敏感
lower_case_table_names = 1

#最大连接数
max_connections = 400

#最大错误连接数
max_connect_errors = 1000

#TIMESTAMP如果没有显示声明NOT NULL，允许NULL值
explicit_defaults_for_timestamp = true

#SQL数据包发送的大小，如果有BLOB对象建议修改成1G
max_allowed_packet = 128M

#MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭
#MySQL默认的wait_timeout  值为8个小时, interactive_timeout参数需要同时配置才能生效
interactive_timeout = 2880000
wait_timeout = 2880000

#内部内存临时表的最大值 ，设置成128M。
#比如大数据量的group by ,order by时可能用到临时表，
#超过了这个值将写入磁盘，系统IO压力增大
tmp_table_size = 134217728
max_heap_table_size = 134217728

#禁用mysql的缓存查询结果集功能
#后期根据业务情况测试决定是否开启
#大部分情况下关闭下面两项
query_cache_size = 0
query_cache_type = 0

#MySQL5.7 group by新特性，报错1055,具体需要根据本身情况进行设定,没有固定值
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

#####################用户进程分配到的内存设置BEGIN#############################

##每个session将会分配参数设置的内存大小
#用于表的顺序扫描，读出的数据暂存于read_buffer_size中，当buff满时或读完，将数据返回上层调用者
#一般在128kb ~ 256kb,用于MyISAM
#read_buffer_size = 131072
#用于表的随机读取，当按照一个非索引字段排序读取时会用到，
#一般在128kb ~ 256kb,用于MyISAM
#read_rnd_buffer_size = 262144
#order by或group by时用到

#建议先调整为2M，后期观察调整
sort_buffer_size = 2097152

#一般数据库中没什么大的事务，设成1~2M，默认32kb
binlog_cache_size = 524288

########################用户进程分配到的内存设置END############################

#在MySQL暂时停止响应新请求之前的短时间内多少个请求可以被存在堆栈中
#官方建议back_log = 50 + (max_connections / 5),封顶数为900
back_log = 130

############################日志设置##########################################

#数据库错误日志文件
log_error = error.log

#慢查询sql日志设置
slow_query_log = 1
slow_query_log_file = slow.log

#检查未使用到索引的sql
log_queries_not_using_indexes = 1

#针对log_queries_not_using_indexes开启后，记录慢sql的频次、每分钟记录的条数
log_throttle_queries_not_using_indexes = 5

#作为从库时生效,从库复制中如何有慢sql也将被记录
log_slow_slave_statements = 1

#慢查询执行的秒数，必须达到此值可被记录
long_query_time = 8

#检索的行数必须达到此值才可被记为慢查询
min_examined_row_limit = 100

#mysql binlog日志文件保存的过期时间，过期后自动删除
expire_logs_days = 5

############################主从复制设置#####################################

#开启mysql binlog功能
log-bin=mysql-bin

#binlog记录内容的方式，记录被操作的每一行
binlog_format = ROW

#对于binlog_format = ROW模式时，减少记录日志的内容，只记录受影响的列
binlog_row_image = minimal

#master status and connection information输出到表mysql.slave_master_info中
master_info_repository = TABLE

#the slave‘s position in the relay logs输出到表mysql.slave_relay_log_info中
relay_log_info_repository = TABLE

#作为从库时生效,想进行级联复制，则需要此参数
log_slave_updates

#作为从库时生效,中继日志relay-log可以自我修复
relay_log_recovery = 1

#作为从库时生效,主从复制时忽略的错误
slave_skip_errors = ddl_exist_errors

#relay_log配置中继日志
relay_log=edu-mysql-relay-bin

#####################redo log和binlog的关系设置BEGIN#########################

#(步骤1) prepare dml相关的SQL操作，然后将redo log buff中的缓存持久化到磁盘
#(步骤2)如果前面prepare成功，那么再继续将事务日志持久化到binlog
#(步骤3)如果前面成功，那么在redo log里面写上一个commit记录
#当innodb_flush_log_at_trx_commit和sync_binlog都为1时是最安全的，
#在mysqld服务崩溃或者服务器主机crash的情况下，binary log只有可能丢失最多一个语句或者一个事务。
#但是都设置为1时会导致频繁的io操作，因此该模式也是最慢的一种方式。
#当innodb_flush_log_at_trx_commit设置为0，mysqld进程的崩溃会导致上一秒钟所有事务数据的丢失。
#当innodb_flush_log_at_trx_commit设置为2，只有在操作系统崩溃或者系统掉电的情况下，上一秒钟所有事务数据才可能丢失。

#commit事务时,控制redo log buff持久化磁盘的模式 默认为1
innodb_flush_log_at_trx_commit = 2

#commit事务时,控制写入mysql binlog日志的模式 默认为0
#innodb_flush_log_at_trx_commit和sync_binlog都为1时，mysql最为安全但性能上压力也是最大
sync_binlog = 1

####################redo log和binlog的关系设置END############################

############################Innodb设置#####################################

#数据块的单位8k，默认是16k，16kCPU压力稍小，8k对select的吞吐量大
#innodb_page_size的参数值也影响最大索引长度，8k比16k的最大索引长度小
#innodb_page_size = 8192

#一般设置物理存储的60% ~ 70%
innodb_buffer_pool_size = 1G

#5.7.6之后默认16M
#innodb_log_buffer_size = 16777216

#该参数针对unix、linux，window上直接注释该参数.默认值为NULL
#O_DIRECT减少操作系统级别VFS的缓存和Innodb本身的buffer缓存之间的冲突
innodb_flush_method = O_DIRECT

#此格式支持压缩, 5.7.7之后为默认值
innodb_file_format = Barracuda

#CPU多核处理能力设置，假设CPU是2颗4核的，设置如下
#读多，写少可以设成2:6的比例
innodb_write_io_threads = 4
innodb_read_io_threads = 4

#提高刷新脏页数量和合并插入数量，改善磁盘I/O处理能力
#默认值200（单位：页）
#可根据磁盘近期的IOPS确定该值
innodb_io_capacity = 500

#为了获取被锁定的资源最大等待时间，默认50秒，超过该时间会报如下错误:
# ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
innodb_lock_wait_timeout = 30

#调整buffer pool中最近使用的页读取并dump的百分比,通过设置该参数可以减少转储的page数
innodb_buffer_pool_dump_pct = 40

#设置redoLog文件所在目录, redoLog记录事务具体操作内容
innodb_log_group_home_dir = /data/mysql/data/

#设置undoLog文件所在目录, undoLog用于事务回滚操作
innodb_undo_directory = /data/mysql/undo/

#在innodb_log_group_home_dir中的redoLog文件数, redoLog文件内容是循环覆盖写入。
innodb_log_files_in_group = 3

#MySql5.7官方建议尽量设置的大些，可以接近innodb_buffer_pool_size的大小
#之前设置该值较大时可能导致mysql宕机恢复时间过长，现在恢复已经加快很多了
#该值减少脏数据刷新到磁盘的频次
#最大值innodb_log_file_size * innodb_log_files_in_group <= 512GB,单文件<=256GB
innodb_log_file_size = 1024M

#设置undoLog文件所占空间可以回收
#5.7之前的MySql的undoLog文件一直增大无法回收
innodb_undo_log_truncate = 1
#innodb_undo_tablespaces = 3
innodb_undo_logs = 128

#5.7.7默认开启该参数 控制单列索引长度最大达到3072
#innodb_large_prefix = 1

#5.7.8默认为4个, Inodb后台清理工作的线程数
#innodb_purge_threads = 4

#通过设置配置参数innodb_thread_concurrency来限制并发线程的数量，
#一旦执行线程的数量达到这个限制，额外的线程在被放置到对队列中之前，会睡眠数微秒，
#可以通过设定参数innodb_thread_sleep_delay来配置睡眠时间
#该值默认为0,在官方doc上，对于innodb_thread_concurrency的使用，也给出了一些建议:
#(1)如果一个工作负载中，并发用户线程的数量小于64，建议设置innodb_thread_concurrency=0；
#(2)如果工作负载一直较为严重甚至偶尔达到顶峰，建议先设置innodb_thread_concurrency=128,
###并通过不断的降低这个参数，96, 80, 64等等，直到发现能够提供最佳性能的线程数
#innodb_thread_concurrency = 0

#强所有发生的死锁错误信息记录到error.log中，之前通过命令行只能查看最近一次死锁信息
innodb_print_all_deadlocks = 1

############################其他设置########################################

[mysqldump]
quick
max_allowed_packet = 128M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 256k
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout

[mysqld_safe]
#增加每个进程的可打开文件数量
open-files-limit = 28192
```



## master 配置

```shell
# 1.修改my.cnf文件内容
	#服务id 唯一
	server-id = 128
	#日志文件名称
	log-bin = mysql-bin
# 2.登录mysql
 mysql -uroot -p 
# 3.新增用户slave用于主从同步
 create user 'slave'@'%' identified by '123456';
# 4.设置用户的权限
grant replication slave, replication client on *.* to 'slave'@'%';
# 5.查询日志文件信息
show master status;
```

## slave配置 

```shell
# 1. 登录mysql
mysql -uroot -p
# 2. 设置密码
set password = password('root');
# 3. 设置root用户的访问权限
grant all privileges on *.* to 'root'@'%';
flush privileges;
# 4. 新增只有查询权限的用户
create user 'viewer'@'localhost' identified by '123456';
grant select on *.* to 'viewer'@'%';
flush privileges;
# 5. 配置主库信息 （关键）
change master to master_host="192.168.26.128",
			    master_user="slave",
			    master_password="123456",
			    master_port=3306,
			    master_log_file="mysql-bin.000001",
			    master_log_pos=617,
			    master_connect_retry=30;
# 6. 修改my.cnf文件内容（主从服务相关信息）
	#服务id
	server-id=129
	############################主从复制设置#####################################

    #开启mysql binlog功能
    log-bin=mysql-slave-bin

    #binlog记录内容的方式，记录被操作的每一行
    binlog_format = ROW

    #对于binlog_format = ROW模式时，减少记录日志的内容，只记录受影响的列
    binlog_row_image = minimal

    #master status and connection information输出到表mysql.slave_master_info中
    master_info_repository = TABLE

    #the slave‘s position in the relay logs输出到表mysql.slave_relay_log_info中
    relay_log_info_repository = TABLE

    #作为从库时生效,想进行级联复制，则需要此参数
    log_slave_updates

    #作为从库时生效,中继日志relay-log可以自我修复
    relay_log_recovery = 1

    #作为从库时生效,主从复制时忽略的错误
    slave_skip_errors = ddl_exist_errors

    #relay_log配置中继日志
    relay_log=edu-mysql-relay-bin
 # 7.退出客户端并重启mysql服务
 service mysql restart
```

## 操作

1. 同步成功信息

   ```
   Name                         |Value                                                 |
   -----------------------------|------------------------------------------------------|
   Slave_IO_State               |Waiting for master to send event                      |
   Master_Host                  |192.168.26.128                                        |
   Master_User                  |slave                                                 |
   Master_Port                  |3306                                                  |
   Connect_Retry                |30                                                    |
   Master_Log_File              |mysql-bin.000001                                      |
   Read_Master_Log_Pos          |1332                                                  |
   Relay_Log_File               |edu-mysql-relay-bin.000005                            |
   Relay_Log_Pos                |876                                                   |
   Relay_Master_Log_File        |mysql-bin.000001                                      |
   Slave_IO_Running             |Yes                                                   |
   Slave_SQL_Running            |Yes                                                   |
   Replicate_Do_DB              |                                                      |
   Replicate_Ignore_DB          |                                                      |
   Replicate_Do_Table           |                                                      |
   Replicate_Ignore_Table       |                                                      |
   Replicate_Wild_Do_Table      |                                                      |
   Replicate_Wild_Ignore_Table  |                                                      |
   Last_Errno                   |0                                                     |
   Last_Error                   |                                                      |
   Skip_Counter                 |0                                                     |
   Exec_Master_Log_Pos          |1332                                                  |
   Relay_Log_Space              |1087                                                  |
   Until_Condition              |None                                                  |
   Until_Log_File               |                                                      |
   Until_Log_Pos                |0                                                     |
   Master_SSL_Allowed           |No                                                    |
   Master_SSL_CA_File           |                                                      |
   Master_SSL_CA_Path           |                                                      |
   Master_SSL_Cert              |                                                      |
   Master_SSL_Cipher            |                                                      |
   Master_SSL_Key               |                                                      |
   Seconds_Behind_Master        |0                                                     |
   Master_SSL_Verify_Server_Cert|No                                                    |
   Last_IO_Errno                |0                                                     |
   Last_IO_Error                |                                                      |
   Last_SQL_Errno               |0                                                     |
   Last_SQL_Error               |                                                      |
   Replicate_Ignore_Server_Ids  |                                                      |
   Master_Server_Id             |128                                                   |
   Master_UUID                  |a34a8969-7bed-11eb-9bcb-0242ac110002                  |
   Master_Info_File             |mysql.slave_master_info                               |
   SQL_Delay                    |0                                                     |
   SQL_Remaining_Delay          |                                                      |
   Slave_SQL_Running_State      |Slave has read all relay log; waiting for more updates|
   Master_Retry_Count           |86400                                                 |
   Master_Bind                  |                                                      |
   Last_IO_Error_Timestamp      |                                                      |
   Last_SQL_Error_Timestamp     |                                                      |
   Master_SSL_Crl               |                                                      |
   Master_SSL_Crlpath           |                                                      |
   Retrieved_Gtid_Set           |                                                      |
   Executed_Gtid_Set            |                                                      |
   Auto_Position                |0                                                     |
   Replicate_Rewrite_DB         |                                                      |
   Channel_Name                 |                                                      |
   Master_TLS_Version           |                                                      |
   ```

   

```mysql
-- master 新建库和表
create database mydb;
-- master 新建表
create table test(
	`id` int primary key auto_increment,
    `name` varchar(20)
);
-- 新增数据
insert into test(`name`) values ('tom'),('jerry');

-- 查看slave数据展示
```



## 出现问题

> MySQL8.0 配置时主从复制出现   Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.
>
> 在MySQL8.0之前，身份验证的插件是mysql_native_password，在MySQL 8.0中，caching_sha2_password 是默认的身份验证插件
>
> 修改用户密码
>
> alter user xxxx   identified with mysql_native_password  by  'xxx';





> Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work
>
> mysql 5.6的复制引入了uuid的概念，各个复制结构中的server_uuid得保证不一样，但是查看到直接copy  data文件夹后server_uuid是相同的，show variables like '%server_uuid%';
>
> 找到data文件夹下的auto.cnf文件，修改里面的uuid值，保证各个db的uuid不一样，重启db即可

# 基于mybatis 多数据源配置

## 步骤

1. 新建datasource注解
2. 配置类将多个数据源注入到spring容器中
3. 新建ContextHodler类，使用ThreadLocal保证线程之间相互独立
4. 自定义dynamicDatasource继承AbstractRoutingDatasource，重写determineCurrentLookUpKey和afterPropertiesSet方法
   - determineCurrentLookUpKey：切换数据源，从threadLocal中获取数据源key，再通过此key从resolvedDataSources中获取对应Datasource
   - afterPropertiesSet： 初始化数据源，设置默认数据源以及将需要使用的多个数据源注入到targetDataSources中，最后super父类此方法将targetDataSources中数据源加载到resolvedDataSources中;
5. 自定义aop切面根据方法名前缀及注解参数规则拦截请求，向threadLocal设置指定数据源；
6. 配置sqlsessionFatory

## 代码实现

### 	@DataSource

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(value = {ElementType.TYPE, ElementType.METHOD})
public @interface DataSource {
    String value() default "master";
}

```

### 	DruidDataSourceProperties

```java
@Data
@ConfigurationProperties(prefix = "spring.datasource.druid")
public class DruidDataSourceProperties {
	private int initialSize;
	private int maxActive = 30;
	private int minIdle;
	private long maxWait;
	private long timeBetweenEvictionRunsMillis;
	private long minEvictableIdleTimeMillis;
	private String validationQuery;
}
```



### 	DatasourceConfig

```java
@Slf4j
@Configuration
@EnableConfigurationProperties(DruidDataSourceProperties.class)
public class DatasourceConfig {
    @Autowired
    private DruidDataSourceProperties dataSourceProperties;

    /**
     * 主库 （写库）
     * @return
     */
    @Bean
    @Primary
    @Qualifier("master")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource master() {
        return this.config();
    }

    /**
     * 从库 （读库）
     * @return
     */
    @Bean
    @Qualifier("slave")
    @ConfigurationProperties(prefix = "spring.datasource.second")
    public DataSource slave() {
        return this.config();
    }

    /**
     * mybatis配置
     * @param datasource
     * @return
     */
    @Bean
    @Qualifier("customSqlSessionFactory")
    public SqlSessionFactory customSqlSessionFactory(DynamicDatasource datasource) {
        log.info("mybatis config begin...");
        try {
            //配置数据源
            SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
            factoryBean.setDataSource(datasource);
            //配置model路径
            factoryBean.setTypeAliasesPackage("com.spw.authmg.pojo");
            //扫描mapper文件路径
            ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            factoryBean.setMapperLocations(resolver.getResources("classpath:mapper/*.xml"));
            //下划线 -> 驼峰  & sql语句打印
            org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
            configuration.setMapUnderscoreToCamelCase(true);
            configuration.setLogImpl(org.apache.ibatis.logging.stdout.StdOutImpl.class);
            factoryBean.setConfiguration(configuration);
            return factoryBean.getObject();
        } catch (Exception e) {
            log.error("mybatis config error: {}", e);
        }
        return null;
    }

    /**
     * 注入数据源事务
     * @param datasource
     * @return
     */
    @Bean("customTransactionManger")
    public DataSourceTransactionManager customTransactionManger(DynamicDatasource datasource) {
        return new DataSourceTransactionManager(datasource);
    }


    /**
     * druid 参数配置
     * @return
     */
    private DruidDataSource config() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setInitialSize(dataSourceProperties.getInitialSize());
        dataSource.setMaxActive(dataSourceProperties.getMaxActive());
        dataSource.setMinIdle(dataSourceProperties.getMinIdle());
        dataSource.setMaxWait(dataSourceProperties.getMaxWait());
        dataSource.setTimeBetweenEvictionRunsMillis(dataSourceProperties.getTimeBetweenEvictionRunsMillis());
        dataSource.setMinEvictableIdleTimeMillis(dataSourceProperties.getMinEvictableIdleTimeMillis());
        dataSource.setValidationQuery(dataSourceProperties.getValidationQuery());
        return dataSource;
    }
}
```

### 	DatasourceContextHolder

```java
public final class DatasourceContextHolder {
    /**
     * 设置线程局部变量，保证线程之间互相独立
     */
    private static final  ThreadLocal<String> holder = new ThreadLocal<>();

    static {
        //首次加载设置master为默认数据源
        holder.set(Constants.DatasourceType.MASTER);
    }
    /**
     * 获取数据源
     * @return
     */
    public static String getDatasource() {
        return holder.get();
    }

    /**
     * 设置数据源
     * @param name
     */
    public static void setDatasource(String name) {
        holder.set(name);
    }
}
```

### 	DynamicDatasource

```java
@Slf4j
@Component
public class DynamicDatasource extends AbstractRoutingDataSource {
    @Autowired
    @Qualifier("master")
    private DataSource master;
    @Autowired
    @Qualifier("slave")
    private DataSource slave;

    @Override
    protected Object determineCurrentLookupKey() {
        log.info("数据源切换为：{}", DatasourceContextHolder.getDatasource());
        return DatasourceContextHolder.getDatasource();
    }

    @Override
    public void afterPropertiesSet() {
        //将主库设置为默认数据源
        this.setDefaultTargetDataSource(master);
        //添加多数据源，后期路由key使用
        Map<Object, Object> targetDataSource = new HashMap<>(3);
        targetDataSource.put(Constants.DatasourceType.MASTER, master);
        targetDataSource.put(Constants.DatasourceType.SLAVE, slave);
        this.setTargetDataSources(targetDataSource);
        //将targetDataSources 加载到 resolvedDataSources
        super.afterPropertiesSet();
    }
}
```

### 	Constants

```
public interface Constants {
    /**
     * 数据源常量
     */
    final class DatasourceType {
        public static final String MASTER = "master";
        public static final String SLAVE = "slave";
    }

    /**
     * 方法名称前缀匹配规则
     */
    final class MethodRule {
        public static final String INSERT = "insert";
        public static final String SAVE = "save";
        public static final String UPDATE = "update";
        public static final String DELETE = "delete";
        public static final String FIND = "find";
        public static final String QUERY = "query";
        public static final String COUNT = "count";
        public static final String SELECT = "select";
    }

}
```

## 问题

> @Transactional下使用多数据源失效
>
> 原因： spring TransactionInterceptor会通过aop拦截带@Transactional的方法,然后使用DatasourceTransactionManager事务管理器创建事务，并将事务的信息存放到ThreadLocal中；
>
> 因事务拦截会在自定义切面拦截之前执行，来获取数据源并将其缓存，在调用操作数据库方法之前事务已经获取到connection连接，此时并为获取到切换到的数据源；
>
> 解决方法：将切换数据源方法作用于事务上一层，使其在操作事务之前将其切换成指定数据源；