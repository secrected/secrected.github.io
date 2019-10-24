---
title: redis基础
date: 2019-10-24 14:10:03
tags: redis
categories: [Java,stuNotes,dba]
---

> —REmote DIctionary Server

#### 分布式数据库中CAP原理：

C:Consistency（强一致性）    A:Availability（可用性） P:Partition tolerance（分区容错性）
一个分布式系统不可能同时很好的`满足一致性`，`可用性`和`分区容错性`这三个需求，最多只能同时较好的满足两个。
  <!--more-->
因此，根据 CAP 原理将 NoSQL 分成三大类：

* CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。

* CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。 Redis、Mongodb

* AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。大多数web应用
  一致性和可用性之间取一个平衡。多余大多数web应用，其实并不需要强一致性。因此牺牲C换取P，这是目前分布式数据库产品的方向
  

  
  #### NoSQL四大分类：

* KV键值---memcache+redis

* 文档型数据库(bson格式)

* 列存储数据库–--HBase

* 图关系数据库–--社交网络，推荐系统等。专注于构建关系图谱
  
  安装包：
  redis-benchmark:  性能测试工具，官方数据：写 8w 读 11w
  redis-server：Redis服务器启动命令
  redis-sentinel：redis集群使用
  redis-cli： 客户端，操作入口
  redis-check-dump：修复有问题的dump.rdb文件
  单进程模型来处理客户端的请求，Redis的实际处理速度完全依靠主进程的执行效率
  dbsize 查看当前数据库的key的数量
  flushdb：清空当前库 Flushall；通杀全部库

#### Redis五大数据类型：

string： 一个redis中字符串value最多可以是512M
hash： 键值对集合
list： 链表
set、：string类型的无序集合。它是通过HashTable实现实现
zset(sorted set：有序集合)： 每个元素都会关联一个double类型的分数，成员唯一,但分数(score)却可以重复。

**$键(key)$**
exists key，key是否存在
move key db ，当前库就没有了，被移除了
expire key 秒钟：为给定的key设置过期时间
ttl key ，查看还有多少秒过期，-1永不过期，-2已过期
type key ，查看key类型
**String**
set/get/del/append/strlen
Incr/decr/incrby/decrby    ,一定要是数字才能进行加减
getrange/setrange     :获取/设置指定区间范围内的值
setex(set with expire)键秒值    /setnx(set if not exist) 设置带过期时间的key，动态设置
mset/mget/msetnx     同时设置一个或多个
getset    (先get再set)
**List**
lpush/rpush/lrange
lpop/rpop
lindex，    按照索引下标获得元素(从上到下) lindex list 0
llen
lrem key     删N个value
ltrim key     开始index 结束index，截取指定范围的值后再赋值给key
rpoplpush     源列表 目的列表 lpoprpush list 1 list 2
lset key index value
linsert key before/after 值1 值2
**Set**
sadd/smembers/sismember
srem key value     删除集合中元素
srandmember key     某个整数(随机出几个数)
spop key     随机出栈
smove key1 key2     在key1里某个值 作用是将key1里的某个值赋给key2
差集：sdiff 交集：sinter 并集：sunion
**Hash**
hset/hget/hmset/hmget/hgetall/hdel/ hlen     —hset h na 33 ----hget h na
hexists key     在key里面的某个值的key
hkeys/hvals
hincrby/hincrbyfloat /
hsetnx     不存在赋值，存在了无效。
**Zset**
zadd/zrange zrangebyscore key     开始score 结束score
--------ZRANGE s0 0 -1 withscores
------- ZRANGEBYSCORE s0 1 (2 ZRANGEBYSCORE s0 (1 (2
--------ZRANGEBYSCORE s0 1 3 limit 2 2
zrem key     某score下对应的value值，删除元素
zcard ：获取集合中元素个数
zcount key score区间 ：获取分数区间内元素个数
zrank key values值： 获取value在zset中的下标位置
zscore key 值, 按照值获得对应的分数
zrevrank key values值，作用是逆序获得下标值 zrevrange
zrevrangebyscore key 结束score 开始score
zrevrangebyscore zset1 90 60 withscores 分数是反着来的

#### redis.conf

**tcp-backlog**
设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。
在高并发环境下需要一个高backlog值来避免慢客户端连接问题。Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog两个值来达到想要的效果
**SNAPSHOTTING快照**
RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，默认：
*是1分钟内改了1万次，*
*或5分钟内改了10次，*
*或15分钟内改了1次。*
save 秒钟 写操作次数
**LIMITS限制**
（1）volatile-lru：使用LRU算法移除key，只对设置了过期时间的键
（2）allkeys-lru：使用LRU算法移除key
（3）volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键
（4）allkeys-random：移除随机的key
（5）volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key
（6）noeviction：不进行移除。针对写操作，只是返回错误信息
**APPEND ONLY MODE追加**
appendfsync
always：同步持久化 每次发生数据变更立即记录到磁盘 性能差但数据完整性好
everysec：默认，异步操作，每秒记录 如果一秒内宕机，有数据丢失

#### redis的持久化

是否启用：ps -ef|grep redis  、lsof -i :6379
config get dir 取目录
df -h free 磁盘空间与内存
**RDB（Redis DataBase）**
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到
一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能
如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方
式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。
在指定的时间间隔内将内存中的数据集快照写入磁盘，-----Snapshot快照
***触发RDB快照***
save/ BGSAVE：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。可以通过lastsave命令获取最后一次成功执行快照的时间
flushall命令，也会产生dump.rdb文件，但里面是空的，无意义
优势-------适合大规模的数据恢复;对数据完整性和一致性要求不高
劣势-------在一定间隔时间做一次备份，如果redis意外down掉就会丢失最后一次快照的修改；fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性
动态所有停止RDB保存规则的方法：redis-cli config set save “”

**AOF（Append Only File）**
以日志的形式来记录每个写操作，将Redis执行过的所有写指令记录下来(读操作不记录)，只许追加文件但不可以改写文件，redis启动之初会读取该文件重新构建数据，换言之，redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作
***AOF启动/修复/恢复***
默认的appendonly no，改为yes
写坏的AOF文件：redis-check-aof --fix进行修复—重启redis然后重新加载
***rewrite***
AOF文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制,
当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩，
只保留可以恢复数据的最小指令集.可以使用命令`bgrewriteaof`
***重写原理：***
AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，遍历新进程的内存中数据，每条记录有一条的Set语句。重写aof文件的操作，并没有读取旧的aof文件，而是将整个内存中的数据库内容用命令的方式重写了一个新的aof文件，这点和快照有点类似
*触发机制：*
Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

优势：`appendfsync always` 每次发生数据变更会被立即记录到磁盘 性能较差但数据完整性比较好；每秒同步：`appendfsync everysec` 异步操作，每秒记录 如果一秒内宕机，有数据丢失；不同步：appendfsync no 从不同步
劣势：相同数据集的数据而言aof文件要远大于rdb文件，恢复速度慢于rdb;aof运行效率要慢于rdb,每秒同步策略效率较好，不同步效率和rdb相同

#### Redis的事务

> 部分支持事务
> 一个队列中，一次性、顺序性、排他性的执行一系列命令

`悲观锁(Pessimistic Lock)`，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁。关系型数据库用到了很多锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁
`乐观锁(Optimistic Lock)`，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量

MULTI    开始一个事务》多个命令入队到事务中》EXEC    命令触发事务
**特性**
单独的隔离操作：执行的过程中，不会被其他客户端发送来的命令请求所打断。
没有隔离级别的概念。
不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚。
***watch监控：***
通过WATCH命令在事务执行之前监控了多个Keys，倘若在WATCH之后有任何Key的值发生了变化，EXEC命令执行的事务都将被放弃，同时返回Nullmulti-bulk应答以通知调用者事务执行失败

#### Redis的发布订阅

进程间的一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。
先订阅后发布后才能收到消息，
1 可以一次性订阅多个，SUBSCRIBE c1 c2 c3
2 消息发布，PUBLISH c2 hello-redis
3 订阅多个，通配符*， PSUBSCRIBE new*
4 收取消息， PUBLISH new1 redis2015

#### Redis的复制(Master/Slave)

主从复制：自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主
作用：读写分离；容灾恢复
从库配置：slaveof 主库IP 主库端口
修改配置文件细节操作：

* 拷贝多个redis.conf文件

* 开启daemonize yes

* pid文件名字，log文件名字，dump.rdb名字

* 指定端口
  中途变更转向:会清除之前的数据，重新建立拷贝最新的
  slaveof 新主库IP 新主库端口
  使当前数据库停止与其他数据库的同步，转成主数据库》SLAVEOF no one
  **复制原理**  

* slave启动成功连接到master后会发送一个sync命令

* Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，
  在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步

* 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

* 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
  只要是重新连接master,一次完全同步（全量复制)将被自动执行
  **哨兵模式(sentinel)**
  能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
  `*使用流程：*`
  新建sentinel.conf文件》配置哨兵,填写内容： sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1 最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机》启动哨兵：redis-sentinel /myredis/sentinel.conf
  info replication查看，一组sentinel能同时监控多个Master
  *复制的缺点：*
  
  复制延时：由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统繁忙，延迟问题会加严重，Slave机器数量的增加也会更加严重。

#### Redis的Java客户端Jedis

事务提交

     public class TestTransaction {
         public boolean transMethod() {
             Jedis jedis = new Jedis("127.0.0.1", 6379);
             int balance;// 可用余额
             int debt;// 欠额
             int amtToSubtract = 10;// 实刷额度
             jedis.watch("balance");
             //jedis.set("balance","5");
             balance = Integer.parseInt(jedis.get("balance"));
             if (balance < amtToSubtract) {
               jedis.unwatch();
               System.out.println("modify");
               return false;
             } else {
               System.out.println("***********transaction");
               Transaction transaction = jedis.multi();
               transaction.decrBy("balance", amtToSubtract);
               transaction.incrBy("debt", amtToSubtract);
               transaction.exec();
               balance = Integer.parseInt(jedis.get("balance"));
               debt = Integer.parseInt(jedis.get("debt"));
    
               System.out.println("*******" + balance);
               System.out.println("*******" + debt);
               return true;
         }
       /**
        * watch命令就是标记一个键，如果标记了一个键， 在提交事务前如果该键被别人修改过，那事务就会失败，这种情况通常可以在程序中
        */
       public static void main(String[] args) {
          TestTransaction test = new TestTransaction();
          boolean retValue = test.transMethod();
          System.out.println("main retValue-------: " + retValue);
       }
     }

* 主从复制

```
 public static void main(String[] args) throws InterruptedException 
  {
     Jedis jedis_M = new Jedis("127.0.0.1",6379);
     Jedis jedis_S = new Jedis("127.0.0.1",6380);
     jedis_S.slaveof("127.0.0.1",6379);
     jedis_M.set("k6","v6");
     Thread.sleep(500);
     System.out.println(jedis_S.get("k6"));
  }

```

### 常见配置redis.conf

参数说明

1. Redis默认不是以守护进程的方式运行，可以配置项修改，使用yes启用守护进程
   daemonize no
2. 以守护进程方式运行时，默认会把pid写入/var/run/redis.pid文件，可以pidfile指定
   pidfile /var/run/redis.pid
3. 指定端口，6379在手机按键上MERZ，而MERZ取自意大利歌女Alessia Merz的名字
   port 6379
4. 绑定的主机地址 bind 127.0.0.1
5. 当 客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
   timeout 300
6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose 》》 loglevel verbose
7. 日志记录方式，默认为标准输出，如果守护进程方式运行，又配置为日志记录方式为标准输出，则日志将会发送给/dev/null   logfile stdout
8. 设置数据库的数量，默认数据库为0，可以使用SELECT 命令指定数据库id
   databases 16
9. 指定多长时间内，多少次更新，将数据同步到数据文件，可以多条件配合
   save
   Redis默认配置文件中提供了三个条件：
   save 900 1
   save 300 10
   save 60 10000
   分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。
10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大
    rdbcompression yes
11. 指定本地数据库文件名，默认值为dump.rdb
    dbfilename dump.rdb
12. 指定本地数据库存放目录 dir ./
13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步
    slaveof
14. 当master服务设置了密码保护时，slav服务连接master的密码
    masterauth
15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH 命令提供密码，默认关闭
    requirepass foobared
16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息
    maxclients 128
17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区
    maxmemory
18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no
    appendonly no
19. 指定更新日志文件名，默认为appendonly.aof
    appendfilename appendonly.aof
20. 指定更新日志条件，共有3个可选值：
    no：表示等操作系统进行数据缓存同步到磁盘（快）
    always：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）
    everysec：表示每秒同步一次（折衷，默认值）
    appendfsync everysec
21. 指定是否启用虚拟内存机制，默认值为no，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中
    vm-enabled no
22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享
    vm-swap-file /tmp/redis.swap
23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0
    vm-max-memory 0
24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值
    vm-page-size 32
25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，在磁盘上每8个pages将消耗1byte的内存。
    vm-pages 134217728
26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4
    vm-max-threads 4
27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
    glueoutputbuf yes
28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法
    hash-max-zipmap-entries 64
    hash-max-zipmap-value 512
29. 指定是否激活重置哈希，默认为开启
    activerehashing yes
30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
    include /path/to/local.conf
    
#### JedisPool配置参数
    
    大部分是由JedisPoolConfig的对应项来赋值

maxActive：控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取；如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted。
maxIdle：控制一个pool最多有多少个状态为idle(空闲)的jedis实例；
whenExhaustedAction：表示当pool中的jedis实例都被allocated完时，pool要采取的操作；默认有三种。
WHEN_EXHAUSTED_FAIL --> 表示无jedis实例时，直接抛出NoSuchElementException；
WHEN_EXHAUSTED_BLOCK --> 则表示阻塞住，或者达到maxWait时抛出JedisConnectionException；
WHEN_EXHAUSTED_GROW --> 则表示新建一个jedis实例，也就说设置的maxActive无用；

maxWait：表示当borrow一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛JedisConnectionException；
testOnBorrow：获得一个jedis实例的时候是否检查连接可用性（ping()）；如果为true，则得到的jedis实例均是可用的；
testOnReturn：return 一个jedis实例给pool时，是否检查连接可用性（ping()）；
testWhileIdle：如果为true，表示有一个idle object evitor线程对idle object进行扫描，如果validate失败，此object会被从pool中drop掉；这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义；
timeBetweenEvictionRunsMillis：表示idle object evitor两次扫描之间要sleep的毫秒数；
numTestsPerEvictionRun：表示idle object evitor每次扫描的最多的对象数；
minEvictableIdleTimeMillis：表示一个对象至少停留在idle状态的最短时间，然后才能被idle object evitor扫描并驱逐；这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义；
softMinEvictableIdleTimeMillis：在minEvictableIdleTimeMillis基础上，加入了至少minIdle个对象已经在pool里面了。如果为-1，evicted不会根据idle time驱逐任何对象。如果minEvictableIdleTimeMillis>0，则此项设置无意义，且只有在timeBetweenEvictionRunsMillis大于0时才有意义；
lifo：borrowObject返回对象时，是采用DEFAULT_LIFO（last in first out，即类似cache的最频繁使用队列），如果为False，则表示FIFO队列；

JedisPoolConfig对一些参数的默认设置： 

    testWhileIdle=true 
    
    minEvictableIdleTimeMills=60000 
    
    timeBetweenEvictionRunsMillis=30000 
    
    numTestsPerEvictionRun=-1 
