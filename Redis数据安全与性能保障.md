# Redis数据安全与性能保障 #

## 1.Redis数据持久化 ##

###1.快照持久化###
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

###2. AOF持久化###

1）AOF持久化会将被执行的写命令写到AOF文件的末尾

2）AOF的写入频率配置 appendfsync



选项| 同步频率| 优缺点

：---：|