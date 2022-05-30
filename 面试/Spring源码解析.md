# Spring源码解析

## 1. Bean声明周期源码解析

1. 首先调用AbstractApplicationContext类的refresh方法；

2. 执行preRefresh方法，加载并检查环境变量，添加监听器；

3. 执行obtainFreshBeanFacotry方法；

   1. 获取bean工厂(ConfigurationListableBeanFctory) ；
   2. 执行loadBeanDefination方法，初始化BeanDefination对象，并将其加载到IOC容器中；

4. 执行prepareBeanFactory方法，bean工厂预处理操作；

5. 执行postProcessBeanFactory方法，加载实现BeanPostProcessor接口的类；

6. 执行invokeBeanFactoryPostProcessors方法；

   1. 如果当前工厂类是BeanDefinationRegistry的话，执行实现BeanDefinationRegistry的

      postProcessBeanDefinationRegistry方法；

   2. 实例化并执行实现BeanPostProcessror接口的实现类的postProcessBeanFactory方法；

7. 执行registerBeanPostProcessor方法，注册实现BeanProcessor接口的类；

8. 执行initMessageResource方法，初始化消息资源，国际化使用；

9. 执行initApplicationMulticaster方法，初始化事件分派器；

10. 执行onRefresh方法，刷新过程中的业务逻辑处理；

    比如springBoot中的内嵌应用启动服务器，就在这里进行创建；

11. 执行registerListeners方法，注册应用监听器，并将监听器添加到时间分派器中；

12. 执行finishBeanFactoryInitilization方法，工厂初始化完成后对bean进行操作；

13. 执行preInstantiateSingletons方法，判断如果当前bean是FactoryBean的话，前面拼接&符号；

14. 执行getBean方法 =====> doGetBean方法；

15. 调用getSingleton方法从缓存中获取bean；

    1. 先从singletonObjects单例池中获取；
    2. 不存在且当前bean正在创建中，从earlySingletonObjects二级缓存中获取；
    3. 不存在且当前允许循环依赖，对单例池上锁；
    4. 再次从单例池中获取，不存在再从二级缓存中获取；
    5. 不存在从singletonFactories三级缓存中获取；
    6. 存在，将当前bean从三级缓存中移除，添加到二级缓存中；

16.  如果缓存中不存在且当前bean是单例，调用getSingleton方法，

     存在两个参数：1）beanName 2）ObjectFactory 函数式接口；

    1. getSingleton方法首先会对当前单例池上锁；
    2. 然后从单例池中获取bean；
    3. 不存在通过ObjectFactory调用getObject方法获取；

17. 执行createBean方法；

18. 调用实现InstantiationAwareBeanPostProcessor接口类的postProcessBeforeInstantiation方法；

19. 执行doCreateBean方法，创建bean对象主要分为三大步： 1）实例化  2）属性赋值  3）初始化；

20. 从factoryBeanInstanceCache缓存中获取beanWrapper，不存在调用进行实例化；

21. 调用createBeanInstance方法，内部执行determineConstructorsFromPostProcessors方法，获取构造方法；

    执行实现SmartInstantiationAwareBeanPostProcessor接口类的determineCandidateConstructors方法, 生成beanWrapper对象；

21. 执行applyMergedBeanDefinationPostProcessors方法，解析内部添加@Autowired和@Value注解的属性，生成InjectionMeta存放；
22. 如果当前bean是单例且允许循环依赖且正在创建，调用addSingleFactory方法；如果当前bean在单例池中不存在，从二级缓存中移除并放入到三级缓存中；（解决循环依赖以及代理bean 的重复创建问题）；
23. 调用getEarlyReferences方法，实现SmartInstantiationAwareBeanPostProcessor接口类的getEarlyReference方法；
24. 实例化完成，调用populateBean方法，属性填充；
25. 调用实现InstantiationAwareBeanPostProcessor接口类的postPorcessAfterInstantiation方法，停止实例化；
26. 根据类型byType/byName方法进行属性注入，调用实现InstantiationAwareBeanPostProcessor接口类的postProcessProperties方法；
27. 属性填充完成，调用initilizeBean方法，初始化操作；
28. 执行invokeAwareMethods方法，调用实现BeanAware、ClassLoaderAware、BeanFactoryAware接口的setBeanName、setClassLoaderName、setBeanFactoryName方法；
29. 执行applyBeanPostProcessorsBeforeInitilization方法，调用实现BeanPostProcessor接口类的postProcessBeforeInitilication方法；
30. 执行invokeInitMethods方法；
    1. 调用实现InitilizingBean接口类的afterPropertiesSet方法；
    2. 如果声明init-Method方法或者添加@PostConstruct注解的方法，执行invokeCustomInitMethod方法；
31. 执行applyBeanPostProcessorAfterInitilization方法，调用实现BeanPostProcessor接口类的postProcessorAfterInitilication方法；
32. 初始化完成，将对象添加到单例池中；
33. 当应用停止或者重启，进行销毁操作；
34. 调用InitilizingBean 的destoryBean方法；
35. 执行destroy-method方法或添加@PreDestroy注解的方法； 

## 2. 代理Bean生成过程源码解析

1. 在AbstractAutowireBeanFactory类中的doCreateBean方法中；

2. getEarlyReferences方法；

   会调用AbstractAutoProxyCreator类的getEarlyReferences方法，将当前bean的Class作为key,存放到

   earlyProxyReferences代理bean的缓存中；

3.  applyPostProcessorsAfterInitilization方法；

   调用AbstractAutoProxyCreator类的postProcessAfterInitilization方法；

4. 先从earlyProxyReferences缓存中获取，不存在调用wrapIfNecessary方法获取代理对象；

5. 调用getAdvicesAndAdvisorForBean方法，获取当前bean的增强拦截器；

6. 调用createProxy方法创建代理对象；

7. 判断targetClass属性是否为true，如果是调用cglib动态代理生成代理对象；

8. 如果实现了接口的话，调用jdk动态代理生成代理对象；否则，调用cjlib生成代理对象；

## 3. @Transactional源码解析

1. 声明@EnableTransactionManagement注解；
2. 注解内部导入了TransactionMangementConfigurationSelector类；
3. 调用selectImports方法，adviceMode为PROXY;
4. 创建ProxyTransactionMangementConfiguraion类；
   1. 创建并加入IOC容器中TransactionAttributeSource事务属性源；
   2. 创建并加入IOC容器中TransactionInterceptor事务拦截器，注入TransactionAttributeSource；
   3. 创建并加入IOC容器中transactionAdvisor事务增强器，注入TransactionAttributeSource，TransactionInterceptor；
5. 通过TransactionInterceptor拦截标注@Transactional注解；
6. 执行invoke方法，获取目标对象；
7. 执行invokeWithinTransaction方法来进行拦截；
   1. processWithInvocation 环绕通知;
   2. complateTransactionAfterThrowing 异常通知，调用事务管理器的rollback方法回滚事务；
   3. commitTransactionAfterReturning  正常通知, 调用事务管理器的commit方法提交事务；

