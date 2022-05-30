# HashMap源码解析

## 1. 常量及变量

```java
  常量：static final
      1. DEFAULT_INITIAL_CAPACITY：默认初始容量  1 >>> 4 16
      2. MAXIMUM_CAPACITY: 最大容量 1 >>> 30  2^30
      3. DEFAULT_LOADER_FACTOR: 默认加载因子  0.75f
      4. TREEIFY_THRESHOLD: 链表转红黑树节点个数 8
      5. UNTREEIFY_THRESHOLD: 红黑树转链表节点个数 6
      6. MIN_TREEIFY_CAPACITY: 最小树化节点个数 64
  变量:
      1. threshold: 阈值
      2. Node<k,v>[] table: 数据列表
      3. size: 元素个数
      4. modCount: 修改次数
```

## 2. 构造方法

## 	2.1. 有参构造

​	**new HashMap<>(int capacity, float loadFactor);**

​	**new HashMap<>(int capacity)  => new HashMap<>(int capacity, DEFAULT_LOADER_FACTOR)**

1.  校验容量和加载因子；

2.  赋值容量和加载因子；

3. 设置阈值，调用tableSizeFor(capacity),   初始容量减一，逐个与向右位移2、4、8、16位逻辑或操作，最后阈值等于操作完的容量加一；

   ![image-20220501162800067](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220501162800067.png)
   
   > **为何初始容量需要先减一操作？**
   >
   > ​	初始容量如果没有减一，则操作后的阈值是减一操作阈值的两倍；					

## 	2.2. 无参构造

1.  将加载因子设置为默认加载因子（DEFAULT_LOADER_FACTOR）

## 3. put操作

1.  hash操作key的hashcode与高16位进行与操作（减少hash冲突几率）

1.   判断当前数组是否为空（是否首次存值）, 为空进行resize操作；

2.  查找当前节点存放位置（hash(key) & length - 1）为null，直接存放当前节点；

3.  如果当前节点存在值

   1.  如果当前节点的hash相同且key的值相等，直接替换；

   2.  如果存放元素的节点是树节点，进行红黑树节点存放操作；

   3.  最后就是链表操作，遍历链表查找，存在替换，

      ​	不存在，添加到当前节点后边，此时需要校验当前节点个数是否超过树化节点个数（TREEIFY_CAPACITY= 8-1），超过将链表转成红黑树

4.  操作数加一 modCount

5.  判断当前元素个数是否超过阈值，超过，进行resize扩容操作

   ![image-20220501162319231](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220501162319231.png)

## 4. resize操作

1.  赋值旧数组、容量以及阈值

2.  旧容量大于0，证明非首次put操作，容量未超过最大容量 ，则容量和阈值扩容两倍

3.  旧容量等于0且旧阈值大于0 , 有参构造首次put操作 

   ​	新容量 = 旧阈值

   ​	新阈值 = 新容量 * 默认加载因子(0.75)  = 旧阈值 * 默认加载因子 

4.  旧容量等于0且旧阈值为空， 无参构造首次put操作

   ​     新容量 = 默认初始容量(16)

   ​     新阈值 = 默认初始容量 *  默认加载因子 = 12

5.  table指向新数组

6.  非首次扩容，遍历元素重新存放

7.  如果只有一个节点  hash(key) & newCap -1 

8.  如果是树节点，树操作（中途会将树转变成链表）

9.  链表操作，遍历根据hash高低位放入指定数组中进行存放

​	![image-20220501162420250](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220501162420250.png)	

## 5. get操作

1.  当前table非空校验；

2.  根据当前节点的hash值与数组长度减一进行逻辑与操作，先寻找处于桶中的位置

3.  如果当前位置存在元素，取第一节点进行hash比较以及值比较，匹配直接返回

4.  如果存在多个节点，判断当前位置节点是否是树节点，是进行红黑树查找，采用二分查找方式

5.  最终就是遍历链表查找

   ![image-20220501162603700](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220501162603700.png)

## 6. remove操作

1.  当前数组非空校验；
2. 获取节点与get方法一致
3. 如果当前节点是树节点，进行树操作
4. 如果当前节点是第一个节点，后继节点替换第一个节点
5. 如果当前节点不是第一个节点，当前节点的后继节点挂在前驱节点后面

![image-20220501162654375](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220501162654375.png)
