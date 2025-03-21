### 1、深度分页

mysql server调用inndb接口依次查询数据0-1000000，直到server判断出1000000后的数据。前面100W条数据的扫描都是做的无效操作，很消耗性能。

简单说就是全表扫描，页越深扫描数据越多

```mysql
-- 全表扫描
select * from order_info limit 1000000,100;
```

```mysql
-- 主键索引
select * from order_info where id > 1000000 limit 100;
```

#### 优化

深度分页的优化方法，减少回表、记录上一页最大id然后where筛选

##### 较少回表

1、覆盖索引

2、筛选出id然后通过id再回表，避免筛选过程中扫描的数据回表。

##### 记录上一页ID

记录上一页ID作为数据过滤的起始位置，避免前面逐条扫描。

走了主键索引，定位数据更快



### 2、mysql优化器生成的执行计划选择了错误索引

```mysql
## 耗时4.31秒 mysql使用modified索引
EXPLAIN
  select * from order_info  where period = 202205 order by desc modified limit 0, 1000;
  
 
```

![](.\image\sql优化-1.png)



执行计划选择了idx_modifyed索引，innodb从后向前遍历idx_modifyed对应id回表主键索引筛选出period=202205的数据。如果where条件中period=202205是顺序靠后的数据，需要遍历的数据不会太多性能会好，如果数据靠前那么需要遍历的数据很多查询效率会很低。

#### 初级优化

指定查询计划索引为idx_period【force INDEX(idx_period)】

```mysql
--  优化后
-- 耗时0.295秒 
EXPLAIN
  select * from order_info force INDEX(idx_period) where period = 202205 order by modified desc limit 0, 1000;
```

![](.\image\sql优化-2.png)



sql执行过程：

idx_period索引上筛选出period=202205的数据，拿到数据的主键id到主键索引查询数据，到内存或者文件通过modified字段排序，取出分页数据。

回表查询数据然后排序是比较慢的；只需要idx_period索引上增加modified且有序；**idx_peroid_modified联合索引**就是这样一个数据结构。

##### 问题

回表然后排序



#### 中级优化

mysql查询计划选择了联合索引，并且不需要额外排序，查询耗时0.1秒

```mysql
-- 联合索引
ALTER TABLE order_info ADD INDEX idx_period_modified (period, modified);

-- 耗时108毫秒
select * from order_info  where period = 202205 order by modified desc limit 0, 1000;
```

![](.\image\sql优化-3.png)



##### 问题

深度分页问题，靠前页性能很快，但是靠后页性能慢很多。



### 最终优化

新增联合索引后查询效率提升很多，但是深度分页后性能又会很慢。

#### 深度分页

```mysql
-- 查询99000-100000的数据
-- 耗时 3秒
select * from order_info  where period = 202205 order by modified desc limit 99000, 1000;
```

分页查询，筛选数据起始位置很大。

深度分页性能很差，因为要依次扫描索引+然后回表【id查询主键索引】查询出完整的数据交给mysql server判断数据是否符合要求，直达扫描到分页起始位置的数据。前面扫描取的数据会被丢弃，是无效操作。



#### 深度分页的问题

无效查询，而且还要回表

#### 优化

1、不回表；子查询，条件查出1000个主键，关联主键索引回表查询这1000个主键的数据

查询耗时210毫秒

```mysql
select * from order_info  where period = 202205 order by modified desc limit 99000, 1000;

-- 子查询，条件查出1000个主键，关联主键索引回表查询这1000个主键的数据
-- 耗时210毫秒 
select * from (select id form order_info where period = 202205 order by modified desc limit 99000, 1000) temp join order_info oi on (temp.id = oi.id);
```



![](.\image\sql优化-4.png)