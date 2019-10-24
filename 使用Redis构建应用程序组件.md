# 使用Redis构建应用程序组件 #

## 1.分布式锁 ##

1）对于可以被多个线程访问的共享内存数据结构，程序在对数据进行加锁时，首先需要通过获取锁得到对数据进行排他性访问的能力，然后才能对数据进行操作，操作结束之后需要再释放锁。

2）Redis中使用WATCH命令对数据进行加锁，因为WATCH只会在数据被其它用户修改的情况下通知执行该命令的客户端，而不会组织其他客户端对数据进行访问，所以WATCH命令是一个乐观锁。

3）分布式系统中也需要类似的锁机制。

4）锁发生故障后可能出现的情况：

	- 持有锁的进程因为操作锁的时间过长而导致锁自动释放，进程自己并不知晓这一点，甚至可能会在执行完毕后错误的释放其他锁
	- 一个锁过期后，多个进行尝试获取锁，并且都获得的锁
	- 一个持有锁的进程崩溃后，锁并没有被释放，其他试图获取锁的进程只能阻塞

### 1.简易锁 ###

Redis的SETNX命令只会在键（key）不存在的情况下为键设置值，很适合用来锁的获取功能。

接下来设计的锁只是基本上正确的，并不是完全可靠的。

1）获取锁的实例
    

	def acquire_lock(conn, lockname, acquire_timeout=10):
		identifier = str(uuid.uuid4())
		
		end = time.time() + acquire_timeout
		while time.time() < end:
			if conn.setnx('lock' + lockname, identifier):
				return identifier
			time.sleep(0.01)

		return False

    
2) 释放锁的实例
    

	def release_lock(conn, lockname, identifier):
		pipe = conn.pipeline(True)
		lockname = 'lockname:' + lockname
		
		while True:
			try:
				pipe.watch(lockname)
				if pipe.get(lockname) == identifier # 检查进程是否仍然持有锁
					pipe.multi()
					pipe.delete(lockname)
					pipe.excute()
					return True
				pipe.unwatch()
				break

			except redis.exceptions.WatchError:
				pass
		
		return False

注：加锁时时需要注意锁的粒度，一般来说，细粒度的锁往往可以因为减少竞争出现的概率而提升程序的性能，但是，多个细粒度锁也使得死锁等问题出现的概率增加。

### 2.带有超时时间的锁 ###

为了保证在持有锁的进程崩溃的情况下，锁也可以正常释放，所以需要为锁加上超时功能。

- 程序在获得锁后，调用EXPIRE命令为锁设置过期时间，使得Redis可以自动删除过期的锁
- 客户端在尝试获取锁时，会检查锁的超时时间，并为未设置超时间的锁设置超时时间（可能存在持有锁的进程在介于SETNX与EXPIRE之间崩溃的情况），保证锁可以自动释放。

代码示例：
    
	def acquire_lock_with_timeout(conn, lockname, acquire_timeout=10, lock_timeout=10):
		identifier = str(uuid.uuid4())
		lockname = 'lock:' + lockname
		lock_timeout = int(math.ceil(lock_timeout)) # 保证传给expire的参数为整数
	
		end = time.time() + acquire_timeout
		while time.time() < end:
			if conn.setnx(lockname, identifier):
				conn.expire(lockname, lock_timeout)
				return identifier
			elif not conn.ttl(lockname):
				conn.expire(lockname, lock_timeout)

			time.sleep(0.01)

		return False

