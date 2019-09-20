Redis
	缓存穿透
		什么是缓存穿透：
			查询一个不存在的数据，由于缓存是不命中时被动写的，数据库查询不到数据不会写入缓存，导致每次查询都要走数据库，
			流量大时，数据库就要承受很大的压力。还可以利用不存在的热点key频繁攻击应用。
		解决方法：
			1、数据库查询不到数据将""写入缓存，但是过期时间要设置很短				
		
	缓存雪崩
		多个key使用了同样的过期时间，导致多个缓存在同时失效，请求全部转到数据库，数据库瞬时压力过大导致崩溃
		解决方法：
			不使用固定的过期时间，加上一定时间的随机值。
	缓存击穿


​			
	分布式锁(可重入)
		http://redis.cn/topics/distlock.html
		获取锁：
			正确示范：SET resource_name my_random_value NX PX 30000
			错误示范：Long result = jedis.setnx(lockKey, requestId);
						if (result == 1) {
							// 若在这里程序突然崩溃，则无法设置过期时间，将发生死锁
							jedis.expire(lockKey, expireTime);
						}
						使用setnx命令加锁设置锁的过期时间，并不是一个原子操作而是两步操作，
						如果在设置过期时间前程序突然崩溃，那么就将导致死锁
						
		删除锁(通过lua脚本实现)：
			if redis.call("get",KEYS[1]) == ARGV[1] then
				return redis.call("del",KEYS[1])
			else
				return 0
			end
			使用这种方式释放锁可以避免删除别的客户端获取成功的锁
			比如客户端A获取了锁执行了业务之后释放锁时，由于锁已经过期被自动释放B客户获取到了锁，
			如果使用del命令删除key，就会把B客户端的锁删除掉。


​			
	持久化方式
		rdb
			指定的时间间隔对数据进行快照
		aof
			记录没次的写操作命令，使用这些命令来恢复数据，AOF将写操作命令追加到AOF文件末尾，
			还能对AOF文件进行重写优化，使AOF文件不至于过大。
	淘汰策略
	
	数据结构
		String	  字符串
		List	  列表
		set		  无序集合
		SortedSet 有序集合
		Hash	  散列表


Zookeeper
	事务的ACID
	CAP理论
		CAP理论指在分布式系统中一致性、可用性、分区容错性三者不能同时满足
		
		一致性C
		可用性A
		分区容错性P
	BASE理论
		Base理论是CAP理论演化而来，核心思想是即使无法满足强一致性，但每个业务可以根据自身的特点，采用适当的方式使系统达到最终一致性。
	zookeeper能做什么
	zookeeper的分布式一致性算法


​	

### Zookeeper与Eureka

zk保证的是一致性和分区容错性

​	zk master宕机，需要一定的时间进行master选举，在此期间不能进行服务的注册发现

eureka保证的是可用性和分区容错性

​	eureka是server-client形式，server之间相互独立，不存在leader，其中一个server宕机了只要还有一个server活着，client就会把信息注册到server上，等到宕机的server活过来将注册信息同步过去。



结论：我们能够忍受注册中心返回的是几分钟前的数据，但是不能接受服务不可用获取不到注册信息，也就是说服务的注册与发现可用性的要求高于一致性，如果用来做服务的注册与发现eureka是更好的选择。