# Redis数据安全与性能保障 #

## 1.Redis数据持久化 ##

### 1.快照持久化 ###
1）创建快照的方式：
	
- 客户端向Redis发送BGSAVE命令。
- 客户端向Redis发送SAVE命令。
- Redis的save配置，如：save 60 10000,则从Redis最近一次创建快照之后开始计算，当“60s之内有10000次写入”这个条件被满足时，Redis会自动触发BGSAVE命令，如果用户设置了多个触发条件，则在任意一个选项被满足时都会出发BGSAVE命令。
- 当Redis通过shutdown命令接收到关闭服务器请求，或接收到标准term信号时，会执行一次SAVE命令。
- Redis服务器A连接另一个Redis服务器B，当A向B发送SYNC命令开始一次复制操作的时候，如果B没有在执行BGSAVE命令，或B距离上一次BGSAVE指令时间较长，则会执行BGSAVE

2）BGSAVE与SAVE命令的区别
- BGSAVE命令 Redis会调用fork创建一个子进程来负责将快照写入硬盘，父进程则继续处理命令请求
- SAVE命令 Redis会阻塞所有客户端，在创建快照完成之前不再响应任何其他命令。
- SAVE命令并不常用，通常只在内存不足，无法执行BGSAVE，或者等待持久化操作结束，阻断一段时间也无所谓的情况下才会选择SAVE

### 2. AOF持久化 ###

1）AOF持久化会将被执行的写命令写到AOF文件的末尾

2）AOF的写入频率配置 appendfsync

选项|同步频率|优缺点
:---:|:---:|:---:
always|每个Redis写命令都要同步写入硬盘|系统奔溃时丢失数据最少，但是会严重降低Redis的速度
everysec|每秒执行一次同步|可以兼顾数据安全与性能，但是系统崩溃时会丢失最多1s内的写数据
no|让操作系统来决定何时进行同步|不会对Redis性能造成影响，但是系统崩溃时会丢失不定量的数据

3）随着Redis不断运行，AOF文件的体积也会不断增长，占据的硬盘空间也会越来越大，同时，执行还原操作的时间也会特别长

4）可以通过BGWRITEAOF命令来重写AOF文件，其原理是移除AOF文件中的冗余命令，使AOF文件尽量的小，其工作原理也是Redis fork出一个子进程对AOF重写，但是子进程也会占用系统资源和内存空间，所以在AOF文件过大时，该命令可能会导致Redis宕机数秒

## 2.Redis复制 ##

### 1.Redis复制过程 ###

步骤|主服务器操作|从服务器操作
:---:|:---:|:---:
1|(等待命令进入)|连接(或重连接）主服务器，发送SYNC命令
2|开始执行BGSAVE,并使用缓冲区记录BGSAVE之后执行的所有写命令|根据配置决定是否继续使用现有数据来处理客户端的请求，如果不处理则返回错误信息
3|BGSAVE执行完毕，向从客户端发送快照文件，并在发送期间继续使用缓冲区记录被执行的写命令|丢弃所有旧数据，开始载入主服务器发来的快照文件
4|快照文件发送完毕，开始向从服务器发送储存在缓冲区中的写命令|完成快照的解释操作，并断开自己的从客户端的连接，正常接收客户端请求
5|缓冲区写命令发送完毕，之后所有执行的写命令也都会发送给从服务器|执行主服务器发来的所有缓冲区写命令，并在之后持续接收并执行主服务器发来的写命令

### 2.设置从服务器的方法 ###
1. 配置选项SLAVEOF host port将一个Redis服务器设置为从服务器
2. 向运行中的Redis服务器发送SLAVEOF命令将其设置为从服务器

注：Redis不支持主主复制

### 3.主从链 ###
1）因为从服务器也可以拥有自己的从服务器，所以可能会形成一条主从链
2）主从链如下图：
![](https://github.com/coderdl/redis_knowledge_summary/blob/master/%E5%BC%95%E7%94%A8%E5%9B%BE%E7%89%87/%E4%B8%BB%E4%BB%8E%E9%93%BE.png)

## 3.Redis事务 ##

### 1.Redis事务的基本定义 ###
1）Redis的事务以MULTI未开始，之后跟着用户传入的多个命令，最后以EXEC为结束，但是这种方式在EXEC被调用之前不会执行任何实质操作，所以用户没办法根据读到的数据类型来做决定。

2） Redis在执行事务的过程中，会延迟执行已入队的命令，直到收到客户端发送的EXEC命令。这种“一次性发送多个命令，然后等待所有回复出现”的做法通常被称作“流水线”。

### 2.Redis事务的常见命令 ###

1. MULTI开始一个事务 
2. WATCH WATCH指令对键进行监视，在WATCH到EXEC这段时间里，如果有其他客户端抢先对被监听的键进行了替换，更新或删除操作，那在EXEC时，事务将执行失败，并返回一个WATCHError.
3. UNWATCH 不再对某个键进行监视
4. DISCARD 
5. EXEC 执行一个事务

### 3.Redis事务实例 ###
    


	def list_item(conn, itemid, sellerid, price):
		inventory = 'inventory:%s'%sellerid
		item = '%s.%s'%(itemid, sellerid)
		end = time.time() + 5
		pipe = conn.pipe()
		while time.time() < end:
			try:
				pipe.watch(inventory)
				if not pipe.sismeber(inventory, itemid):
					pipe.unwatch()
					return None
		
				pipe.multi()
				pipe.zadd("market": item, pirce")
				pipe.srem("inventory", itemid)
				pipe.exec()
				return True
			except redis.exceptions.WATCHError:
				pass
		return False
    


### 4.非事务型流水线 ###
    

	pipe = conn.pipeline()
    
1）如果用户在执行pipeline()时传入True作为参数，或者不传入参数，那么客户端将使用MULTI和EXEC包裹起来用户要执行的所有命令。

2）如果用户在执行pipeline()时传入False作为参数，那么客户端会像执行事务那样收集起用户要执行的所有命令，但是不适用mutli和exec包裹起来。

3) 如果用户需要向Redis发送多个命令，并且对于这些命令来说，一个命令的执行结果并不会影响到另一个命令的输入，而且这些命令也不需要以事物的方式执行，那使用非事务型流水线可以提升Redis的性能。