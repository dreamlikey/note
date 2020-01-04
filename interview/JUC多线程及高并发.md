1. #### 请谈谈你对volatile的理解

2. #### CAS你知道吗?

3. #### 原子类AtomicInteger的ABA问题谈谈?原子更新引用知道吗?

4. #### 我们知道ArrayList是线程不安全，请编码写-一个不安全的案例并给出解决方案。

   

   ##### 问题

   java.util.ConcurrentModificationException

   1. ```java
      public static void main(String[] args) {
          List<String> list = new ArrayList<>();
          for (int i = 0; i < 30; i++) {
              new Thread(() -> {
               list.add(UUID.randomUUID().toString().substring(0,8));
                  System.out.println(list);
              },String.valueOf(i)).start();
          }
      }
      ```

   ##### 解决方案

   1、Vector  方法加锁，synchronized

   2、Collections.synchronizedList

   ```java
   List<String> list = Collections.synchronizedList(new ArrayList<>());
   ```

   3、CpoyOnWriteArrayList

   写时复制，读写分离思想

   ```java
   List<String> list = new CopyOnWriteArrayList();
   ```





1. #### 公平锁/非公平锁/可重入锁/递归锁/自旋锁谈谈你的理解?请手写一个自旋锁

2. #### CountDownLatch/CyclicBarrier/Semaphore使用过吗?

3. #### 阻塞队列知道吗?

4. #### 线程池用过吗? ThreadPoolExecutor谈谈你的理解?

5. #### 线程池用过吗?生产上你如何设置合理参数

6. #### 死锁编码及定位分析