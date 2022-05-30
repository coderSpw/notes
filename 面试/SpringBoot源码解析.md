# SpringBoot源码解析

## 1. @SpringBootApplication注解

#### 1.1. @SpringBootConfiguration

内部引入@Configuration注解，标注当前类是个配置类

#### 1.2. @ComponentScan

自动扫描符合条件的bean组件，并将其添加到IOC容器中；

#### 1.3. @EnableAutoConfiguration

开启自动配置

##### 1.3.1. @AutoConfigurationPackage

内部import导入Registar.class，主要是注册同级及子级的包中的bean；

##### 1.3.2. @Import({AutoConfigurationImportSelector.class})

1. 执行selectImports方法；
2. 执行getAutoConfigurationEntry方法；
3. 执行SpringFactoriesLoader.loadFactoryNames方法，使用反射方式实例化从META-INF/spring.factories中key为EnableAutoConfiguration下的标注了@Configuration类，并加载到IOC容器中；
4. 去除重复配置类；
5. 去除exclude属性标注的类；
6. 存放导入的配置类；

## 2. SpringBoot的执行流程源码解析

#### 2.1. 执行SpringApplication.run(XXX.class, args)；

主要分为两步：1）实例化  2）执行run方法；

#### 2.2. 实例化

1. 判断是否为web容器，使用webEnvironment标记；
2. 从META-INF/spring.factories中查找并加载可用的ApplicationContextInitilizer;
3. 从META-INF/spring.factories中查找并加载可用的ApplicationContextListener;
4. 推断并设置main方法的定义类；

#### 2.3. 执行run方法

1. 从META-INF/spring.factories中查找并加载可用的SpringApplicationRunListener；

2. 遍历监听器调用start方法；

3. 将参数资源加载到环境中，并遍历监听器执行environmentPrepared方法；

4. 打印banner;

5. 创建应用上下文，根据webApplicationType选择对应的应用上下文，

   分为：reactive、servlet、default

6. 执行prepareContext，对应用上下文进行后置处理，对@EnableAutoConfiguration导入的配置类加载到准备好的应用上下文中；

7. 执行refresh方法，实际上执行的是AbstractApplicationContext类中的refresh方法，完成bean的创建；

8. 执行afterRefresh方法，容器加载完成后的自定义操作；

9. 执行callRunners方法，执行实现CommandLineRunner和ApplicationRunner接口类的run方法，进行容器启动后的操作；

## 3. 内嵌容器启动过程

1. 通过spring-boot-autoconfigure中META-INF/spring.factories文件里面EnableAutoconfiguration键下的

   EmbededWebServerFactoryCustomizerAutoConfiguration类；

2. 通过EmbededWebServerFactoryCustomizerAutoConfiguration类里面@conditionalOnClass判断注入的容器配置文件来获取启动容器，默认Tomcat.class，TomcatWebServerFactoryCustomizerConfiguration -- TomcatWebServerFactoryCustomizer

3. 通过@enableAutoConfiguration注解加载完配置bean;

4. 调用run方法，内部执行refresh方法，实际调用AbstractApplicationContext的refresh方法；

5. 执行onRefresh方法，执行ServletWebServerApplicationContext子类的onRefresh方法，此处会创建启动容器 createWebServer;

6. 调用getWebServerFactory工厂，通过工厂调用getWebServer来创建启动容器并启动；
