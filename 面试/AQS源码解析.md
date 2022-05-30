# AQS（AbstractQueuedSychnorized）源码解析

## 1. 核心

- 状态

- 队列

  - 同步队列（sync queue） ：双向链表

    ![image-20220505174813774](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505174813774.png)

  - 条件队列（condition queue） ：单向链表

    ![image-20220505195958682](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195958682.png)

- cas

  **compareAndSetState**：设置状态

  **compareAndSetHead：**设置头节点

  **compareAndSetTail**：设置尾节点

## 2. 条件队列与同步队列之间的联系

针对于Condition操作，当调用await方法会将当前节点放入条件队列中，调用signal/signalAll一个一个的唤醒条件队列中的线程来竞争锁，

获取锁失败，放入同步队列中排队等待。

![image-20220505175204132](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505175204132.png)

## 3. 独占锁/共享锁/公平锁/非公平锁解释

### 3.1. 独占锁/共享锁

1. 独占锁：当前锁只能被一个线程占有，其他线程只能等当前线程释放锁后被唤醒来竞争锁资源；

   例如：ReentrantLock 、ReentrantReadWriteLock.WriteLock、CyclicBarrier

2. 共享锁：当前锁可以被多个线程同时持有

   例如： ReentrantReadWriteLock.ReadLock、CountDownLatch、Semaphore

### 3.2. 公平锁/非公平锁

1. 公平锁：先进先出，先进队列中的线程先获取锁；
2. 非公平锁：同时唤醒多个线程来竞争，优先级高的先获取锁；

## 4. 独占锁获取锁源码解析

![image-20220505181821477](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505181821477.png)

1.  首先尝试获取锁 tryAcquire(子类方法)，获取当前线程以及状态；

   1. 如果state等于0，证明当前锁未被线程占有，将当前线程设置为独占线程，cas方式修改状态；

   2. 如果state大于0且当前线程是持有锁的线程，证明当前操作为重入锁操作，将当前线程状态加一；

   3. 否则获取锁失败；

      > ```
      > 公平锁相对于非公平锁多一个hasQueuedPredecessors条件判断
      > 公平锁：当前线程是同步队列第一个节点且设置状态成功才能获取锁成功；
      > ```

      ![image-20220505181840198](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505181840198.png)

2. 将当前节点添加到同步队列中, addWaiter

   1. 如果队列尾节点不为空，将当前节点设置为尾节点并连接到前尾节点后面；

   2. 如果队列不存在，enq方法，自旋操作，新new一个节点作为头节点，然后将当前节点设置为尾节点并连接到前尾节点后面；

      ![image-20220505181900560](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505181900560.png)

      ![image-20220505181918925](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505181918925.png)

3. 通过自旋处理同步队列中节点 ，acquireQueued

   1. 如果当前节点的前一个节点是头节点且尝试获取锁成功，将前节点设置为null，当前节点设置为队列的头节点,中断跳出自旋；

   2.  处理等待状态大于0（CANCEL）的节点，shouldParkAfterFailedAcquire，

      将前节点状态设置为SIGNAL，用来通知下个节点获取锁

      ![image-20220505182030589](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505182030589.png)

      ![image-20220505182110987](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505182110987.png)

## 5. 独占锁释放锁源码解析

![image-20220505183705186](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505183705186.png)

1. 首先尝试释放锁操作tryRelease

   1. 如果当前线程不是持有锁的线程，抛异常IllegalMonitorStateException

   2. 如果当前线程状态为0，释放锁成功，将当前独占线程置为null

      ![image-20220505183735788](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505183735788.png)

 2. 释放锁成功后，判断当前头节点状态是否为SIGNAL且不是哑节点, 唤醒后继线程； unparkSuccessor

    1. 将当前节点状态设置为0；

    2. 如果当前节点的下一个节点waiStatus大于0（CANCEL），从后向前遍历清除CANCEL状态节点，获取waiStatus小于0的节点，唤醒节点；

       ![image-20220505183748640](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505183748640.png)

## 6. 共享锁获取锁源码分析

![image-20220505191610791](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505191610791.png)

1. 调用tryAcquireShared尝试获取锁；

   ![image-20220505191626767](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505191626767.png)

2. 如果小于0，获取锁失败，将当前线程添加到同步队列中；

   ![image-20220505191655478](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505191655478.png)

3. 自旋，获取当前节点的前驱节点，如果是头节点的话，再次尝试获取锁；

4. 如果大于等于0，证明获取锁成功，唤醒下个节点获取锁；  setHeadAndPropagate

   ![image-20220505191732684](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505191732684.png)

## 7. 共享锁释放锁源码分析

1. 尝试释放锁tryReleaseShared；

   ![](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505192652283.png)

2. 释放锁，唤醒下个节点获取锁；

   ![](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505192710858.png)

## 8. Condition中await源码分析

![image-20220505195050544](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195050544.png)

1. 将当前线程添加到条件队列中，如果尾节点状态不是CONDITION（CANCEL），遍历清除无效节点    **addConditionWaiter**

   ![image-20220505195106839](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195106839.png)

2. 释放当前线程占用的锁   **fullyRelease**

   ![image-20220505195122568](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195122568.png)

3. 如果当前线程在同步队列中，证明未执行signal方法，将当前线程挂起，当调用signal方法或线程中断跳出循环  **isOnSyncQueue**

   ![image-20220505195137146](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195137146.png)

4. 将当前节点添加到同步队列中   **acquireQueued**

   ![image-20220505195227128](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195227128.png)

5. 如果当前节点下一个节点不为空，再次遍历清除无效节点（CANCEL）   **unlinkCancelledWaiters**

   ![image-20220505195214818](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195214818.png)

## 9. Condition中signal/signalAll源码解析

![image-20220505195719174](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195719174.png)

1. 校验当前锁占有的线程是否是当前线程，不是抛异常 IllegalMonitorStateException

2. 循环遍历条件队列一个一个唤醒线程，如果是signal只唤醒第一个节点； **doSignalAll**  ==>  **transferForSignal**

   ![](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195734075.png)

3. 修改当前节点等待状态，然后将唤醒的线程添加到同步队列中；  **enq**

   ![image-20220505195753660](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195753660.png)

   ![image-20220505195808319](https://raw.staticdn.net/coderSpw/notes/master/note_img/image-20220505195808319.png)