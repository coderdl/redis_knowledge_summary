# Redis 基础命令 #

## 1. Redis数据类型 ##
1. 字符串
2. 列表
3. 集合
4. 散列
5. 有序集合

### 1.Redis字符串 ###
Redis字符串可以支持储存：1.字符串 2.整数 3.浮点数
#### 1.Redis字符串常用指令 ####
自增/自减命令

1. INCR key-name
2. DECR key-name
3. INCRBY key-name amount
4. DECRBY key-name amount
5. INCRBYFLOAT key-name amount

注1：对一个不存在的键或value为空串的键执行上述指令，系统会将其当作0处理，如果对一个value无法解析为浮点数或整数的键执行上述指令，系统将报错

注2：INCR底层使用了INCRBY方法实，INCRBY如果不指定参数会默认为1

处理子串和二进制位命令

1. APPEND key-name value
2. GETRANGE key-name start end
3. SETRANGE key-name offset value
3. GETBIT key-name offset
4. SETBIT key-name offset value
5. BITCOUNT key-name [start end]	//[]表示可选参数
6. BITOP operation dest-key key-name [key-name ...]

注1：在使用SETRANGE和SETBIT命令时，如果当前字符串长度不能满足写入需求，系统会先自动使用空字节(null)将字符串扩展到所需长度，再执行写入或更新操作

注2：使用GETRANGE时，超过字符串末尾的值会被当做空串

注3：使用GETBIT时，超过字符串末尾的二进制位会被当作0

### 2.列表 ###

#### 1.列表常用命令 ####
1. RPUSH key-name value [value ...]
2. LPUSH key-name value [value ...]
3. RPOP key-name
4. LPOP key-name
5. LINDEX key-name offset
6. LRANGE key-name start end
7. LTRIM key-name start end
8. BLPOP key-name [key-name ...] timeout
9. BRPOP key-name [key-name ...] timeout
10. RPOPLPUSH source-key dest-key
11. BRPOPLPUSH source-key dest-key timeout

### 3.集合 ####

#### 1.集合常用命令 ####
1. SADD key-name item
2. SREM key-name item
3. SISMEMBER key-name item
4. SCARD key-name 返回集合包含的元素数量
5. SMEMBERS key-name 返回集合包含的所有元素
6. SRANDMEMBER key-name [count] 随即从集合中取1-n个元素，若count小于0，则取出的值可能重复
7. SPOP key-name
8. SMOVE source-key dest-key item
9. SDIFF key-name [key-name ...]
10. SDIFF dest-key key-name [key-name ...]
11. SINTER key-name [key-name ...]
12. SINTERSTORE dest-key key-name [key-name ...]
13. SUNION key-name [key-name ...]
14. SUNIONSTORE dest-key key-name [key-name ...]

### 4.散列 ###
####1.散列常用命令 ####
1. HMGET key-name key [key ...]
2. HMSET key-name key value [key value...]
3. HDEL key-name key [key ...]
4. HLEN key-name
5. HEXISTS key-name key 
6. HKEYS key
7. HVALS key-name
8. HGETALL key-name
9. HINCRBY key-name key increment
10. HINCRBYFLOAT key-name key increment


### 5.有序集合 ###
####1.有序集合常用命令 ####
1. ZADD key-name score member [score member ...]
2. ZREM key-name member [member ...]
3. ZCARD key-name
4. ZINCRBY key-name increment member
5. ZCOUNT key-name min max
6. ZRANK key-name member 返回member在有序集合中的排名
7. ZSCORE key-name member
8. ZRANGE key-name start stop [WITHSCORES]
9. ZREVRANK key-name member
10. ZREVRANGE key-name start stop [WITHSCORES]
11. ZRANGEBYSCORE key-name min max [WITHSCORES]
12. ZREVRANGEBYSCORE key-name min max [WITHSCORES]
13. ZREMRANGEBYRANK key-name start stop
14. ZREMRANGEBYSCORE key-name min max
15. ZINTERSTORE dest-key key-count key [key ...] [WEIGHTS weight [weight ..]] [AGGREGATE SUM|MIN|MAX]
16. ZUNIONSTORE dest-key key-count key [key ...] [WEIGHTS weight [weight ..]] [AGGREGATE SUM|MIN|MAX]

注：ZREV*指令及逆序操作


### 6.发布与订阅 ###
####1.发布与订阅常用指令####
1. SUBSCRIBE channel [channel ...]
2. UNSUBSCRIBE [channel ...]
3. PUBLISH channel message
4. PSUBSCRIBE pattern [pattern ...] 订阅与给定模式相匹配的所有频道
5. PUNSUBSCRIBE [pattern ...] 退订给定模式的所有频道，如果没有给定，则退订所有频道

注：
Redis自带的发布与订阅模式的缺点：
1.Redis系统自身的稳定性：假设Redis订阅了某频道，但是Redis自己读取速度不够快，导致了消息挤压，进而进一步导致Redis输出缓冲区体积越来越大，这可能会导致Redis速度变慢，甚至崩溃，也可能直接导致Redis被系统杀死。（新版redis可以解决这个问题，redis会自动断开不符合配置要求的订阅客户端）
2.数据传输的可靠性：如果客户端在执行订阅的过程中断线，在其重新连接后，会丢失短线期间发送的所有消息

###7.其他Redis命令###
1. 排序命令：SORT source-key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC|DESC] [ALPHA] [STORE dest-key]
2. 事务命令： WATCH, MULTI, EXEC, UNWATCH, DISCARD
3. 过期时间命令：
	1. PERSIST key-name  移除建的过期时间
	2. TTL key-name 查看键距离过期还有多少秒
	3. EXPIRE key-name seconds 让给定键在多少秒之后过期
	4. EXPIREAT key-name timestamp 将给定键的过期时间设置为指定的UNIX时间戳
	5. PTTL key-name 查看键距离过期时间还有多少毫秒
	6. PEXPIRE key-name milliseconds 让给定的键在指定毫秒之后过期
	7. PEXPIREAT key-name milliseconds 将一个毫秒级的UNIX时间戳设置为键的过期时间

