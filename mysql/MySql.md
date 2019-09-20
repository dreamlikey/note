数据库引擎
	innodb
	myisam
	
索引
	索引类型
		B+Tree平衡二叉树
		聚簇索引
		非聚簇索引

事务隔离级别
	第一类丢失更新、脏读、不可重复读、幻读
	
性能优化
	1、性能优化法则
	2、索引优化
		频繁更新的字段不适合做为索引
		数据重复率高的字段不适合做为索引
		
​		
	3、表结构设计
		a、适当冗余
		
		b、字段类型选取
			数字类型与字符类型区别


​			
			数值类型
				手机号	
				ip地址  
				年龄 	tinyint
				状态 	tinyint
				
			字符类型
				char
				varchar
			时间类型
				datetime
				timestamp

数据类型
	
	数字类型
		数字类型字段所占磁盘空间是固定的，tinyint 1byte、smallint 2byte、int 4byte、bigint 8byte，
		声明一个字段int(5)，括号中的数字不是字段长度，而是字段的显示宽度（int类型占4个字节），存入数字小于宽度左侧用0补满，不限制超过指定宽度的显示。
			例如：存入1显示为00001，123456显示为123456
		
		tinyint		1字节
		smallint 	2字节
		int			4字节
		bigint		8字节
	字符串类型
		MySql规定一行数据的字符类型数据总长度不能超过65535字节
		字符串类型字段所占存储空间与编码有关，不同的编码下每个字符所占空间如下
			latin 1byte、gbk 2byte、utf8 3byte
		char 	定长字符型
			
		varchar	变长字符型 1-2个字段存储字符长度


​		
​		
​		
​		
​		
​		