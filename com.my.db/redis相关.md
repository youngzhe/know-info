#redis的持久化
```
定义：Redis是一个支持持久化的内存数据库，通过持久化机制把内存中的数据同步到硬盘文件来保证数据持
久化。当Redis重启后通过把硬盘文件重新加载到内存，就能达到恢复数据的目的。
实现：单独创建fork()一个子进程，将当前父进程的数据库数据复制到子进程的内存中，然后由子进程
写入到临时文件中，持久化的过程结束了，再用这个临时文件替换上次的快照文件，然后子进程退出，
内存释放。
RDB：是Redis默认的持久化方式。按照一定的时间周期策略把内存的数据以快照的形式保存到硬盘的二
进制文件。即Snapshot快照存储，对应产生的数据文件为dump.rdb，通过配置文件中的save参数来定
义快照的周期。（ 快照可以是其所表示的数据的一个副本，也可以是数据的一个复制品。）
AOF：Redis会将每一个收到的写命令都通过Write函数追加到文件最后，类似于MySQL的binlog。当
Redis重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。
当两种方式同时开启时，数据恢复Redis会优先选择AOF恢复。
```
#数据类型
```
(一)String
这个其实没啥好说的，最常规的set/get操作，value可以是String也可以是数字。一般做一些复杂的计
数功能的缓存。
(二)hash
这里value存放的是结构化的对象，比较方便的就是操作其中的某个字段。博主在做单点登录的时候，
就是用这种数据结构存储用户信息，以cookieId作为key，设置30分钟为缓存过期时间，能很好的模拟
出类似session的效果。
(三)list
使用List的数据结构，可以做简单的消息队列的功能。另外还有一个就是，可以利用lrange命令，做基
于redis的分页功能，性能极佳，用户体验好。本人还用一个场景，很合适—取行情信息。就也是个生产
者和消费者的场景。LIST可以很好的完成排队，先进先出的原则。
(四)set
因为set堆放的是一堆不重复值的集合。所以可以做全局去重的功能。为什么不用JVM自带的Set进行去
重？因为我们的系统一般都是集群部署，使用JVM自带的Set，比较麻烦，难道为了一个做一个全局去
重，再起一个公共服务，太麻烦了。
另外，就是利用交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有喜好等功能
(五)sorted set
sorted set多了一个权重参数score,集合中的元素能够按score进行排列。可以做排行榜应用，取TOP N
操作。
```
#过期策略及内存淘汰机制
```
定时删除：用一个定时器来负责监视key,过期则自动删除。虽然内存及时释放，但是十分消耗CPU资源。
在大并发请求下，CPU要将时间应用在处理请求，而不是删除key,因此没有采用这一策略.
惰性删除：在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除。
如果定期删除没删除key。然后你也没即时去请求key，也就是说惰性删除也没生效。这样，
redis的内存会越来越高。那么就应该采用内存淘汰机制。
内存淘汰配置：redis.conf中的配置 maxmemory-policy  volatile-lru  【从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰】
```
#redis为什么是单线程的
```
Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了（毕竟采用多线程会有很多麻烦！）Redis利用队列技术将并发访问变为串行访问
1）绝大部分请求是纯粹的内存操作（非常快速）
2）采用单线程,避免了不必要的上下文切换和竞争条件
3）非阻塞IO优点：
    速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
    支持丰富数据类型，支持string，list，set，sorted set，hash
    支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
    丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除如何解决redis的并发竞争key问题
```
#redis常见性能问题和解决方式
```
(1) Master 最好不要做任何持久化工作，如 RDB 内存快照和 AOF 日志文件
(2) 如果数据比较重要，某个 Slave 开启 AOF 备份数据，策略设置为每秒同步一次
(3) 为了主从复制的速度和连接的稳定性， Master 和 Slave 最好在同一个局域网内
(4) 尽量避免在压力很大的主库上增加从库
(5) 主从复制不要用图状结构，用单向链表结构更为稳定，即： Master <- Slave1 <- Slave2 <-Slave3…
```
#为什么redis操作是原子的，怎么保证
```
Redis的操作之所以是原子性的，是因为Redis是单线程的。
Redis本身提供的所有API都是原子操作，Redis中的事务其实是要保证批量操作的原子性。
多个命令在并发中也是原子性的吗？
不一定， 将get和set改成单命令操作，incr 。使用Redis的事务，或者使用Redis+Lua==的方式实现.
```
#Redis事务
```
Redis事务功能是通过MULTI、EXEC、DISCARD和WATCH 四个原语实现的
Redis会将一个事务中的所有命令序列化，然后按顺序执行。
1.redis 不支持回滚“Redis 在事务失败时不进行回滚，而是继续执行余下的命令”， 所以 Redis 的内部可以保持简单且快速。
2.如果在一个事务中的命令出现错误，那么所有的命令都不会执行；
3.如果在一个事务中出现运行错误，那么正确的命令会被执行。
1）MULTI命令用于开启一个事务，它总是返回OK。 MULTI执行之后，客户端可以继续向服务器发送任意多条命令，这些命令不会立即被执行，而是被放到一个队列中，当EXEC命令被调用时，所有队列中的命令才会被执行。
2）EXEC：执行所有事务块内的命令。返回事务块内所有命令的返回值，按命令执行的先后顺序排列。当操作被打断时，返回空值 nil 。
3）通过调用DISCARD，客户端可以清空事务队列，并放弃执行事务， 并且客户端会从事务状态中退出。
4）WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。 可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令。
```
#redis的命令：
```
连接远程redis服务： redis-cli -h host -p port -a password
                                    redis 127.0.0.1:6379> PING
redis设置key值：redis 127.0.0.1:6379> SET runoobkey redis
将 key 中储存的数字值增一：incr key
                                               decr key
string的命令：set  key  velue
                        get key
hash的命令： 【是string类型的映射表和value，适合存储对象】
                        hset  key  [value是对象]
                        hget  key   field  [获取hash对指定的字段的值]
list的命令：【简单的字符串列表，按插入顺序排序，可添加一个元素到列表头部{左边}，或者尾部{右边}】
                   lpush  key  value  【将一个或者多个元素插入列表的头部】
                   lpop  key  【移除并获取列表的第一个元素】 
set的命令：【string类型元素成员，无序，不可重复。通过hash表实现】
                   SADD key member1 [member2] 【向集合中添加一个或者多个成员】
                  SDIFF key1 [key2]  【返回第一个集合与其他集合的差异】
                  SINTER key1 [key2]   【返回给定所有集合的交集】
sorted set命令：【string类型元素成员，不允许重复，每个元素都带一个score{可同}，进行排序】 
                   ZADD key score1 member1 [score2 member2]    向有序集合添加一个或多个成员，或者更新已存在成员的分数】
                    ZRANGE runoobkey 0 10 WITHSCORES 【获取key 前十个元素带score的值】
```

#redis与数据库缓存一致性问题
    第一种方案：采用延时双删策略
        先删除缓存；
        再写数据库；
        休眠500毫秒；
        再次删除缓存
    从理论上来说，给缓存设置过期时间，是保证最终一致性的解决方案。所有的写操作以数据库为准，只要到达缓存过期时间，则后面的读请求自然会从数据库中读取新值然后回填缓存。
    该方案的弊端
        结合双删策略+缓存超时设置，这样最差的情况就是在超时时间内数据存在不一致，而且又增加了写请求的耗时。
    
    第二种方案：异步更新缓存(基于订阅binlog的同步机制)
    MySQL binlog增量订阅消费+消息队列+增量数据更新到redis
    读Redis：热数据基本都在Redis
    写MySQL:增删改都是操作MySQL
    更新Redis数据：MySQ的数据操作binlog，来更新到Redis
    
    数据操作主要分为两大块：
    一个是全量(将全部数据一次写入到redis)
    一个是增量（实时更新）
    这里说的是增量,指的是mysql的update、insert、delate变更数据。
    2）读取binlog后分析 ，利用消息队列,推送更新各台的redis缓存数据。
    这样一旦MySQL中产生了新的写入、更新、删除等操作，就可以把binlog相关的消息推送至Redis，Redis再根据binlog中的记录，对Redis进行更新。


##缓存雪崩
    同一时间大面积失效，那一瞬间Redis跟没有一样，那这个数量级别的请求直接打到数据库几乎是灾难性的，你想想如果打挂的是一个用户服务的库，那其他依赖他的库所有的接口几乎都会报错
    处理缓存雪崩简单，在批量往Redis存数据的时候，把每个Key的失效时间都加个随机值就好了，这样可以保证数据不会在同一时间大面积失效，我相信，Redis这点流量还是顶得住的。
    setRedis（Key，value，time + Math.random() * 10000）；
    
    如果Redis是集群部署，将热点数据均匀分布在不同的Redis库中也能避免全部失效的问题，不过本渣我在生产环境中操作集群的时候，单个服务都是对应的单个Redis分片，是为了方便数据的管理，但是也同样有了可能会失效这样的弊端，失效时间随机是个好策略。

##缓存穿透/缓存击穿
    缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，我们数据库的 id 都是1开始自增上去的，如发起为id值为 -1 的数据或 id 为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大，严重会击垮数据库
    像这种你如果不对参数做校验，数据库id都是大于0的，我一直用小于0的参数去请求你，每次都能绕开Redis直接打到数据库，数据库也查不到，每次都这样，并发高点就容易崩掉了
    
    缓存击穿嘛，这个跟缓存雪崩有点像，但是又有一点不一样，缓存雪崩是因为大面积的缓存失效，打崩了DB，而缓存击穿不同的是缓存击穿是指一个Key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个Key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个完好无损的桶上凿开了一个洞。
    缓存穿透我会在接口层增加校验，比如用户鉴权校验，参数做校验，不合法的参数直接代码Return，比如：id 做基础校验，id <=0的直接拦截等。
    从缓存取不到的数据，在数据库中也没有取到，这时也可以将对应Key的Value对写为null、位置错误、稍后重试这样的值具体取啥问产品，或者看具体的场景，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。
    这样可以防止攻击用户反复用同一个id暴力攻击，但是我们要知道正常用户是不会在单秒内发起这么多次请求的，那网关层Nginx本渣我也记得有配置项，可以让运维大大对单个IP每秒访问次数超出阈值的IP都拉黑。
    布隆过滤器（Bloom Filter）这个也能很好的防止缓存穿透的发生，他的原理也很简单就是利用高效的数据结构和算法快速判断出你这个Key是否在数据库中存在，不存在你return就好了，存在你就去查了DB刷新KV再return。
    
    