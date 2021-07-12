## 1、定时任务多副本下只执行一次

	### Spring Schedule 搭配 shedlock 实现

1.  引入shedlock相关依赖

   ```java
   <!-- shedLock -->
   		<dependency>
   			<groupId>net.javacrumbs.shedlock</groupId>
   			<artifactId>shedlock-core</artifactId>
   			<version>4.5.0</version>
   		</dependency>
   		<dependency>
   			<groupId>net.javacrumbs.shedlock</groupId>
   			<artifactId>shedlock-spring</artifactId>
   			<version>4.5.0</version>
   		</dependency>
   		<dependency>
   			<groupId>net.javacrumbs.shedlock</groupId>
   			<artifactId>shedlock-provider-jdbc-template</artifactId>
   			<version>4.5.0</version>
   		</dependency>
   ```

2.   定义表

   ```mysql
   CREATE TABLE tb_shedlock(
   		`NAME` varchar(64) PRIMARY KEY NOT NULL DEFAULT '' COMMENT '任务名',
   		`lock_until` timestamp(3) NULL DEFAULT NULL COMMENT '释放时间',
   		`locked_at` timestamp(3) NULL DEFAULT NULL COMMENT '锁定时间',
   		`locked_by` varchar(255) DEFAULT NULL COMMENT '锁定实例'
   ) comment '定时任务表';
   ```

3. 配置类

   ```java
   /**
    * 定时任务分布式锁 (避免多服务下多次执行)
    * @author spw
    * @date 2021/6/28
    */
   @Configuration
   public class LockProviderConfig {
   
       @Bean
       public LockProvider lockProvider(DataSource dataSource) {
           return new JdbcTemplateLockProvider(
               JdbcTemplateLockProvider.Configuration.builder()
                   .withJdbcTemplate(new JdbcTemplate(dataSource))
                   .withTableName("tb_shedlock")
                   .withTimeZone(TimeZone.getTimeZone("GMT+8"))
                   .build()
           );
       }
   }
   ```

4. 添加相应注解

   ```java
   /**
    * 定时任务
    * @author spw
    * @date 2021/6/28
    */
   @Slf4j
   @Component
   @EnableScheduling
   @EnableSchedulerLock(defaultLockAtMostFor = "30s")
   public class ScheduleJob {
       /**
        *  秒 分 时 日 月 周 年(可选)
        *
        *  *：每个   -：范围   /: 指定增量  ？：不指定值(只被用于日和周)    L: 月中最后一天 / 年中最后一周
        *
        *  每天凌晨2点整执行
        */
       @Scheduled(cron = "0 0 2 * * ?")
       @SchedulerLock(name = "testScheduled")
       public void testScheduled() {
           log.info("test scheduled =====> {}", new Date());
       }
   }
   
   ```




## 2、log4j2日志文件中输入sql执行语句

1.  mybatisPlus配置：

   ```yaml
   mybatis-plus:
     configuration:
       log-impl: org.apache.ibatis.logging.log4j2.Log4j2Impl
       map-underscore-to-camel-case: true
     mapper-locations: ["classpath:mapper/*.xml"]
     global-config:
       db-config:
         logic-delete-value: 1
         logic-not-delete-value: 0
   ```

2. log4j2日志文件配置：

   ```xml
    <!--Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。-->
       <!--然后定义loggers，只有定义了logger并引入的appender，appender才会生效-->
       <loggers>
           <!--过滤掉spring和mybatis的一些无用的DEBUG信息-->
           <logger name="cn.cnsa.exam.mapper" level="DEBUG" additivity="false">
               <AppenderRef ref="RollingFile"/>
           </logger>
           <logger name="org.mybatis" level="DEBUG" additivity="false">
               <AppenderRef ref="RollingFile"/>
           </logger>
           <!--监控系统信息-->
           <!--若是additivity设为false，则 子Logger 只会在自己的appender里输出，而不会在 父Logger 的appender里输出。-->
           <Logger name="org.springframework" level="info" additivity="false">
               <AppenderRef ref="RollingFile"/>
           </Logger>
   
           <root level="info">
               <appender-ref ref="RollingFile"/>
           </root>
       </loggers>
   ```



## 3、MultipartFile转File

```java
/**
     * MultipartFile 转 File
     * @param multipartFile 上传文件
     * @return
     */
    private static File multipartFile2File(MultipartFile multipartFile) throws IOException {
        // 获取文件名
        String fileName = multipartFile.getOriginalFilename();
        // 获取文件后缀
        String prefix=fileName.substring(fileName.lastIndexOf("."));
        // 用文件名+当系统时间毫秒值，防止生成的临时文件重复
        final File file = File.createTempFile(prefix + System.currentTimeMillis(), prefix);
        // MultipartFile to File
        multipartFile.transferTo(file);
        return file;
    }
```



## 4、Mysql 获取当前日期所在的周一和周日

```mysql
#获取当前日期所在的周一
select subdate(curdate(),date_format(curdate(),'%w')-1);
#获取当前日期所在的周日
select subdate(curdate(),date_format(curdate(),'%w')-7);

```



## 5. spring boot项目 jar包无法读取文件

> #### Spring Boot 读取 resources下的文件报错 cannot be resolved to absolute file path because it does not reside in the file system: jar

1. 通过classPathRescoure读取文件
2. 创建临时文件，将inputStream流拷贝到临时文件中
3. 通过fileInputStream读取文件内容

```java
public void download(HttpServletRequest request, HttpServletResponse response) throws IOException {
        //生成临时文件
        File file = File.createTempFile("UserTemplate", ".xlsx");
        try (OutputStream out = response.getOutputStream();
             InputStream inputStream = new ClassPathResource("template/UserTemplate.xlsx")
             	.getInputStream();
             FileInputStream fileInputStream = new FileInputStream(file)) {
            //生成目标文件
            FileUtils.copyInputStreamToFile(inputStream, file);
            response.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
            response.setHeader("Access-Control-Expose-Headers", "Content-Disposition");
            response.setHeader("Content-Disposition", "attachment;filename=" + file.getName());
            response.addHeader("Pargam", "no-cache");
            response.addHeader("Cache-Control", "no-cache");
            int b;
            while ((b = fileInputStream.read()) != -1) {
                out.write(b);
            }
        }
    }
```



## 6. Spring Boot + Druid密码加解密

 1. 采用druid JAR  的ConfigTools类 加密生成 公钥、私钥、加密后的密码

    **java -cp druid-1.1.10.jar com.alibaba.druid.filter.config.ConfigTools 123456**

    ![image-20210707110247524](https://gitee.com/coderSpw/notes_img/raw/master/image-20210707110247524.png)

2. 配置文件配置，根据公钥解密   

   **config.decrypt=true;config.decrypt.key=${spring.datasource.druid.publicKey}**

   ``` java
   /**
        * druid 参数配置
        * @return
        */
       private DruidDataSource config() {
           DruidDataSource druidDataSource = new DruidDataSource();
           druidDataSource.setMinIdle(druidProperties.getMinIdle());
           druidDataSource.setMaxActive(druidProperties.getMaxActive());
           druidDataSource.setMaxWait(druidProperties.getMaxWait());
           druidDataSource.setTimeBetweenEvictionRunsMillis(druidProperties.getTimeBetweenEvictionRunsMillis());
           druidDataSource.setMinEvictableIdleTimeMillis(druidProperties.getMinEvictableIdleTimeMillis());
           druidDataSource.setValidationQuery(druidProperties.getValidationQuery());
           //设置数据库密码解密 config.decrypt=true;config.decrypt.key=${spring.datasource.druid.publicKey}
           Properties properties = new Properties();
           properties.setProperty("config.decrypt", "true");
           properties.setProperty("config.decrypt.key", druidProperties.getPublicKey());
           druidDataSource.setConnectProperties(properties);
           try {
               druidDataSource.setFilters(druidProperties.getFilters());
               // druidDataSource.init();
           } catch (SQLException e) {
               log.error("配置druid数据源异常", e);
           }
   
           return druidDataSource;
       }
   ```

   
