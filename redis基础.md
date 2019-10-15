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

###2.列表 ###

####1.列表常用命令 ####
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

###3.集合 ####
####1.集合常用命令 ####

