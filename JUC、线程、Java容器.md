## Collection

	List
		ArrayList
		LinkedList
	Set
	Queue
## Map

### HashMap

##### 		put操作的执行过程

##### 		HashMap怎么解决Hash碰撞

##### 		为什么使用红黑树而不是直接扩容

##### 如何线程安全的使用HashMap

HashMap为什么线程不安全

​	1、两个线程同时put操作，刚好这两个key的hash值相同发生了hash碰撞，那么其中一个key就会被另一个覆盖，而数据丢失

​	2、如果多个线程同时检查到元素个数超过数组大小*loadfactor，进行扩容操作，最终只有一个线程将数组赋给table，其它线程的的扩容操作就会丢失，put新增的数据也会丢失。

​	3、多线程同时扩容，可能造成链表死循环

​		http://www.importnew.com/22011.html



[]: http://www.importnew.com/21396.html	"线程安全的使用HashMap"

- Collections.synchronizedMap(new HashMap());

			返回SychronizedMap对象，SychronizedMap使用了synchronized关键字保证线程安全

- HashTable

			synchronized关键字保证线程安全

- ConcurrentHashMap

			数据结构数组+链表+红黑树
		CAS算法和synchronized锁实现线程安全，在并发处理中使用的是乐观锁，遇见冲突的时候才进行并发处理
				

###### 三种方式的性能比较

​			ConcurrentHashMap使用，性能上比HashTable、synchronizedMap使用synchronized方式性能	高几倍2.5倍
​	

### HashTable



### TreeMap

[^Map]: 

------



## List

### ArrayList

### LinkedList



[^List]: 

------

​	

## JUC

### 锁分类

#### 	悲观锁（独占锁）

​			每次请求都加锁，其它线程请求数据会阻塞直到它拿到锁。

​			synchronized、ReentrantLock就是一种独占锁，它会让竞争这个锁的线程挂起，等待锁释放再参与竞争。

#### 	乐观锁

​		每次请求不加锁，更新数据时判断是否有别人更新过数据。版本好或CAS算法实现

#### 	两种锁的使用场景	

​		乐观锁适用于写比较少的情况（多读场景），冲突发生的情况少，因为如果多写场景，冲突概率大请求失败的概率大，系统吞吐量降低，上层应用不断重试又增加了系统压力。

​		多写场景下悲观锁比较合适。

### CAS置换算法compare and swap

https://blog.csdn.net/tiandao321/article/details/80811103
	1、它是一种非阻塞方式保证线程安全的算法，当内存中的值与期望值不相等的时候表示被其它线程修改过，返回失败；相等时内存更新为新值并返回成功。

​	2、CAS有几个重要参数，期望值expect、更新值update、操作变量所在内存地址valueOffset

​	3、（不重要）CAS是硬件同步源语，有CPU硬件实现，速度快。
​	
​	CAS缺点
​		1、ABA问题
​		2、循环时间长开销大
​			对于自旋CAS操作，更新失败后进行轮询重试操作，如果长时间循环会对系统造成一定负担。
​		3、只能保证一个变量的原子操作
​			

### AQS	

### ConcurrentHashMap

​	数据结构与HashMap一样，使用CAS算法和Synchronized关键字实现线程安全，在并发处理中使用的是乐观锁，遇见冲突的时候才进行同步处理。
​	HashTable、SynchronizedMap使用Synchronized锁实现线程安全，处理并发使用悲观锁，不管什么情况在访问对象上加锁。


​		
### Lock

​	ReentranLock底层使用CAS算法但它不是乐观锁，ReentrantLock同样会让线程等待，与synchornized一样线程顺序执行。ReentrantLock使用CAS算法来判断当前线程是否持有锁，如果不是就放入等待队列中，synchornized是作用于JVM内存屏障来判断线程是否持有锁。

#### 	ReentrantLock 可重入锁

#### 	ReentrantReadWriteLock 可重入读写锁


​		
### Atomic原子操作包

​	1、AtomicInteger、AtomicLong、AtomicBoolean，为什么没有浮点数和String
​	
​	2、基于CAS置换算法实现线程安全，内存中的值与期望值相等，更新并返回成功；内存中的值与期望值不等，轮询。
​	3、AtomicReference、AtomicStampedReference的作用
​		AtomicReference保证多个变量的原子操作
​		AtomicStampedReference 解决ABA问题


​		
### 线程池

​	Executor
​	ExecutorService
​	AbstracExecutorService
​	ThreadPoolExecutor
​		线程池参数
​		submit 
​			有返回值
​		execute
​			无返回值
​	Future
​	FutureTask

​	


​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		
​		