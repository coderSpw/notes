# Mybatis源码解析

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/1717343a66d9566c~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

![在这里插入图片描述](https://raw.staticdn.net/coderSpw/notes/master/note_img/1717343a5a426a4d~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

## 1.  SqlSessionFactoryBuilder

通过SqlSessionFactoryBuilder的build方法创建SqlSessionFactory工厂。

build存在多个重载方法，

1. 首先会通过inputStream流来加载并解析配置文件中的属性生成Configuration对象；

   mapper存放在mapperStatements一个map对象，key：namespace+id, value: MappedStatement对象

2. 然后调用Configuration作为参数的重载方法生成SqlSessionFactory。

3. ![image-20220515124950310](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515124950310.png)

## 2. SqlSessionFactory

![image-20220515125055643](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125055643.png)

1. 通过SqlSessionFactory调用openSession方法获取SqlSession对象。

2. 通过Configuration中的environment获取TransactionFactory，然后通过事务工厂创建事务Transaction；

3. 通过tx和execType执行方式来获取执行器，同时会调用插件拦截器链对Executor进行增强生成代理对象；执行器的声明周期为sqlSession; 

   执行器分为：

   - BatchExecutor   批处理执行器

     **执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同；**

   - ReuseExecutor    重复使用Statement对象的执行器

     **执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，就是重复使用Statement对象。**

   - SimpleExecutor   简单执行器

     **每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。**

   - CachingExecutor    二级缓存执行器

     **每次执行都会先从二级缓存里面查询**

> ​		**interceptorChain.pluginAll   ===> Intercept.plugin  ===>  Plugin.wrap**
>
> ​		**采用jdk动态代理的方式生成代理对象；**

![image-20220515125121217](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125121217.png)

![image-20220515125711579](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125711579.png)

![image-20220515130011000](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515130011000.png)

4. 最后创建DefaultSqlSession对象；

## 3. SqlSession（selectList流程）

### 3.1. 基于statementID

1. 通过namespace+id从Configuration的mappedStatements中获取MappedStatement对象；

2. 使用executor执行器执行query方法；

   ![image-20220515125158966](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125158966.png)

3. 请求参数与sql绑定生成boundSql对象；

4. 创建缓存key，CacheKey；

5. 执行重载query方法；

   ![image-20220515125228339](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125228339.png)

6. 通过第四步创建的key从一级缓存中获取，不存在再查数据库；

7. 调用queryFromDatabase从数据库中查询；

   ![image-20220515125301413](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125301413.png)

8. 调用doQuery方法查询结果；

   ![image-20220515125325221](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125325221.png)

9. 创建StatementHandler对象，内部也会调用inteceptorChain插件链生成代理对象；

   此时会生成paramterHandler和ResultSetHandler代理对象；

   ![image-20220515125540357](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125540357.png)

   ![image-20220515125552461](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125552461.png)

   ![image-20220515125619837](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125619837.png)

   ![image-20220515125642280](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125642280.png)

10. 执行getConnection方法从连接池中获取连接；

11. 执行prepare方法生成Statement对象；

12. 从boundSql中获取sql语句，然后执行statement的execute方法执行sql;

13. 调用resultHandler的hanleResultSets方法对返回数据与对象进行参数映射；

14. 将查询出来的结果放入一级缓存中；

    ![image-20220515125423538](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515125423538.png)

### 3.2. 基于Mapper接口

1. 通过接口的class字节码作为参数调用；

2. 通过接口的字节码来从map获取MapperProxyFactory代理工厂对象；

3. 调用newInstance方法，通过mapper的代理工厂进行操作

   内部采用的JDK动态代理；

   ![image-20220515130126864](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515130126864.png)

4. 通过MapperMethod的execute方法根据类型来执行相对应的sql操作；

5. 内部同样使用的是sqlSession来进行操作；

   ![image-20220515130334966](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515130334966.png)

   ![image-20220515130408122](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515130408122.png)

![image-20220515130424590](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220515130424590.png)
