# JVM

# 1. 类加载流程及各步解释

![image-20220506104906118](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506104906118.png)

### 1.1. 加载

将类的字节码加载到方法区中，在堆中给.class分配一个内存空间

### 1.2. 链接

#### 1.2.1. 验证

校验类代码是否符合JVM规范以及安全性检查

#### 1.2.2. 准备

给静态变量分配内存空间并赋默认值

> 如果是被static final修饰的基本数据类型/字符串，直接赋初始值
>
> 如果是被static final修饰的引用数据类型 , 再初始化时再进行初始化赋值

#### 1.2.3. 解析

将符号引用转变为直接引用

### 1.3. 初始化

执行类构造器的clinit()方法，类变量赋值和静态代码块合并

> **类初始化的情况**
>
> - main方法所在的类，会首先被初始化
> - 首次访问类的静态变量或静态方法时，会被初始化
> - 子类初始化但父类还没初始化会进行初始化
> - 子类访问父类静态变量，会触发父类初始化
> - new会进行初始化
> - class.forName会初始化

## 2. 内存结构

![image-20220506111350717](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506111350717.png)

### 2.1. 本地方法栈

使用native关键字修饰的方法用于通过本地方法接口调C或C++方法

### 2.2. 程序计数器

保存jvm下一条指令执行的地址

### 2.3. 虚拟机栈

线程私有，由一个个栈帧组成，每个栈帧包括局部变量表、操作数栈、动态链接以及方法出口组成

![image-20220506112249071](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506112249071.png)

局部变量用于存储基本数据类型的数据以及引用数据类型堆位置的指针；

操作数栈用于参数计算位置

> StackOverFlowError：栈溢出
>
> 通过设置xss参数大小来调控栈深度

### 2.4. 堆

所有线程共享区域，存放new创建的对象，存在垃圾回收机制

内存划分

- 年轻代 yong    	1   Minor GC
  - eden区         8
  - survivor0区    1
  - survivor1区    1
- 老年代 old     2    Full GC

年轻代与老年代默认比例 1：2  ，年轻代中默认比例  8：1：1

> **什么情况下会直接跳过年轻代进入老年代？**
>
> 1. 大对象 ,  只对Serial和ParNew两款收集器有效 **-XX:PretenureSizeThreshold=1M**
> 2. 对象经历Minor GC 次数超过设置的阈值 ，默认 **-XX:MaxTenuringThreshold = 15**
> 3. Survivor区中同龄对象大小超过空间的50%，大于此年龄的对象进入老年代

### 2.5. 方法区

存放类的基本信息、常量池、字符串常量池

- 1.8以前为永久代，jvm空间内
- 1.8及以后为元空间，jvm内存外开辟一个空间存放类信息以及常量池，将字符串常量池放入堆中

### 2.6. 解释器

将字节码解析为平台通用的机器码，下次遇到相同的字节码也需要解析

### 2.7. 即时编译器

将字节码编译为平台特定的机器码，并存放到code cache中，下次遇到直接执行无需编译

### 2.8. 垃圾回收器

用于回收堆内存中无效数据，通过可达性分析方法来清除GC ROOT链外的节点

## 3. 垃圾回收算法

### 3.1. 标记-清除算法

先标记再清除

缺点：会存在内存碎片

![img](https://raw.staticdn.net/coderSpw/notes/master/note_img/211a3bc9503beb5bebff30ab80ed6248.png)

### 3.2. 标记-整理算法

先标记再清除最后整理

缺点：效率低

![img](https://raw.staticdn.net/coderSpw/notes/master/note_img/85592dde987fa92751d8a1e5c5ba55ef.png)

### 3.3. 复制算法

将内存空间分为两个相等大小的空间，将有效数据挪到新复制的内存空间中

缺点： 内存资源占用多，多用于存活率低的情况

![img](https://raw.staticdn.net/coderSpw/notes/master/note_img/f7abf09c61ddd49e45b87f4bbf735cbd.png)

## 4. GC垃圾回收器

1. 分代回收

   - 年轻代： 

     - serial     单线程 **复制算法**
     - parallel  强调吞吐量（运行代码时间/（运行代码时间+GC时间）） **复制算法**
     - parNew  多线程 **复制算法**

   - 老年代：

     - serial Old  单线程 **标记整理算法**

     - Parallel Old   多线程 **标记整理算法**

     - CMS     **标记清除算法** 

       初始标记（STW）  =》并发标记  =》 重新标记 （STW）  =》并发清理

     ![image-20220506142222872](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506142222872.png)

     ![image-20220506142231763](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506142231763.png)

     ![image-20220506142721780](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506142721780.png)

2. 分区回收

   - G1 :

     将堆划分为大小相等的区域

     整体采用标记整理算法，区域内使用复制算法

     初始标记 =》并发标记 =》 最终标记 =》筛选回收

##### 			![image-20220506143318953](C:/Users/s5536/AppData/Roaming/Typora/typora-user-images/image-20220506143318953.png)

## 5. 类加载器

### 5.1.  分类描述

| 名称                                      | 加载的类              | 说明                            |
| ----------------------------------------- | --------------------- | ------------------------------- |
| Bootstrap ClassLoader（启动类加载器）     | JAVA_HOME/jre/lib     | 无法直接访问                    |
| Extension ClassLoader(拓展类加载器)       | JAVA_HOME/jre/lib/ext | 上级为Bootstrap，**显示为null** |
| Application ClassLoader(应用程序类加载器) | classpath             | 上级为Extension                 |
| 自定义类加载器                            | 自定义                | 上级为Application               |

### 5.2. 双亲委派机制

类加载时，会自下而上委派给父加载器加载，父类找不到在向下查找，如果找不到则抛异常

![image-20220506152653705](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506152653705.png)

通过ClassLoader中的loadClass方法操作：

1. 给当前类名上锁

2. 如果parent（父加载器）存在的话，递归查找 , 如果parent为null，则证明是启动类加载器，调用启动类加载器查找方法（native）

3. 如果查找的类不存在，调用findClass方法，首先将类名拼接.class，通过类路径来查找类的字节码文件加载；

   ![image-20220506154103406](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506154103406.png)

   ![image-20220506154119464](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220506154119464.png)

### 5.3. 破坏双亲委派机制案例

### 5.3.1. JDBC通过加载SPI接口实现

DriverManager --- loadInitialDrivers ----  ServiceLoader.load(Driver.class)   ----  Thread.currentThread().getContextClassLoader()

加载驱动会首先获取线程上下文加载器进行加载，不存在再走默认加载器

### 5.3.2 Tomcat

tomcat采用隔离机制，给每个web容器提供一个单独的WebAppClassLoader加载器，优先加载本身的class文件，加载不到再交给CommonClassLoader加载



![img](https://raw.staticdn.net/coderSpw/notes/master/note_img/v2-371ed70e75e64b7d6fed16a05db59e4e_720w.jpg)