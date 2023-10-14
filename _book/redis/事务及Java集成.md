### 1. 什么是Redis事务

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

### 2. Redis事务相关命令和使用

MULTI ：开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。

EXEC：执行事务中的所有操作命令。

DISCARD：取消事务，放弃执行事务块中的所有命令。

WATCH：监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。

UNWATCH：取消WATCH对所有key的监视。

### 3. 如何理解Redis与事务的ACID？

- **原子性atomicity**

**Redis事务在编译错误时回滚，运行时错误跳过。**很多文章由此说Redis事务违背原子性的；而官方文档认为是遵从原子性的。

Redis官方文档给的理解是，**Redis的事务是原子性的：所有的命令，要么全部执行，要么全部不执行**。而不是完全成功。

- **一致性consistency**

redis事务可以保证命令失败的情况下得以回滚，数据能恢复到没有执行之前的样子，是保证一致性的，除非redis进程意外终结。

- **隔离性Isolation**

redis事务是严格遵守隔离性的，原因是redis是单进程单线程模式(v6.0之前），可以保证命令执行过程中不会被其他客户端命令打断。

但是，Redis不像其它结构化数据库有隔离级别这种设计。

- **持久性Durability**

**redis事务是不保证持久性的**，这是因为redis持久化策略中不管是RDB还是AOF都是异步执行的，不保证持久性是出于对性能的考虑。

### 4. CAS操作实现乐观锁

 check-and-set (CAS)：被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回nil-reply来表示事务已经失败。

```sql
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```

### 5. Redis事务执行步骤

通过上文命令执行，很显然Redis事务执行是三个阶段：

- **开启**：以MULTI开始一个事务
- **入队**：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
- **执行**：由EXEC命令触发事务

当一个客户端切换到事务状态之后， 服务器会根据这个客户端发来的不同命令执行不同的操作：

- 如果客户端发送的命令为 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令的其中一个， 那么服务器立即执行这个命令。
- 与此相反， 如果客户端发送的命令是 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令以外的其他命令， 那么服务器并不立即执行这个命令， 而是将这个命令放入一个事务队列里面， 然后向客户端返回 QUEUED 回复。

<center><img src="redis.png" style="zoom:80%"></center>

### 6. Jedis

**Jedis是Redis官方提供的Java客户端**，用于在 Java 应用程序中连接、操作 Redis，它提供了与 Redis 通信的 API，简化了 Java 开发者与 Redis 的交互流程。

Jedis Github Readme：https://github.com/redis/jedis#getting-started

### 7. SpringBoot

在 SpringBoot2.x 之后，原来使用的 jedis 被替换成了 lettcuce，原因：

Jedis：采用直连，多个线程操作不安全，需要使用 Jedis pool 来避免，会有线程阻塞的情况（BIO模式）

Lettuce：采用 netty，实例可以在多个线程共享，不存在线程不安全的情况，可以减少线程数量（NIO模式）

Lettuce Doc：https://lettuce.io/core/release/reference/#_for_gradle_users

### 8. Redis.conf 详解

查找 redis.conf 位置

```bash
redis-cli config get dir
find / -name redis.conf
```

Linux系统可以

```bash
systemctl status redis-server
```

- unit 单位对大小写不敏感
- bind 127.0.0.1 绑定IP
- protected-mode yes 保护模式，docker启动需要关闭
- port 6379 端口设置
- daemonize yes 以守护进程的方式运行
- pidfile /var/run/redis_6379.pid 如果以后台方式运行，需要指定pid进程文件
- loglevel notice
- logfile "" 日志的文件位置名
- database 16 默认的数据库数量
- always-show-logo yes 是否总是显示logo

快照：持久化，在规定的时间内，执行了多少次操作，则会持久化到文件`.rdb`,`.aof`

redis是内存数据库，如果没有持久化，那么断电即丢失！

```in
save 900 1 #如果900秒，如果至少有一个key进行了修改，我们即进行持久化操作
save 300 10
save 60 10000
```

- stop-writes-on-bgsave-error yes 如果持久化出错，是否还需要继续工作

- rdbcompression yes 是否压缩rdb持久化文件，需要消耗cpu资源
- rdbchecksum yes 保存时是否校验rdb文件
- dir ./ 持久化rdb文件保存的目录

```bash
config get requirepass # 获取requirepass项配置
config set requirepass "123456" # 设置requirepass项配置
auth 123456 # 登录才能查看及修改以上配置内容
```

- maxclients 10000 默认可连接的最大客户端数目

- maxmemory <bytes> redis配置最大内存

- LRU least recently used

- LFU least frequently used

- maxmemory-policy noeviction 内存到达上限之后的处理策略：

  ```ini
  volatile-lru：只对设置了过期时间的key进行LRU算法进行删除
  allkeys-lru：对所有key执行LRU算法进行删除
  volatile-lfu：只对设置了过期时间的key进行LFU算法进行删除
  allkeys-lfu：对所有key执行LFU算法进行删除
  volatile-random：随机删除设置有过期时间的key
  allkeys-random：随机删除
  volatile-ttl：删除即将过期的
  noeviction：永不过期，返回错误
  ```

- appendonly no 默认不开启aof模式，默认是使用rdb方式持久化的，大部分情况下，rdb完全够用

- appendfilename "appendonly.aof" 持久化的文件名字

- appendfsync everysec 每秒执行一次同步，always 每次修改都写入，速度较慢，no 让操作系统决定同步，速度最快