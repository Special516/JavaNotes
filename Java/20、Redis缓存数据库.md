## 	Redis缓存数据库

#### 一、NoSQL概述：

NoSQL = Not Only SQL，泛指非关系型数据库。NoSQL 数据库的特点都是去掉关系数据库的关系型特性。数据之间无关系就很容易进行扩展。

NoSQL数据库具有非常高的读写性能，这得益于它的无关系性，数据库的结构简单。

**NoSQL 数据库的四大分类：**

- **KV键值**
- **文档型数据库：** bson 格式，MongoDB 基于分布式文件存储的数据库。
- **列存储数据库：** HBase，Cassandra
- **图关系数据库：** 存放的是关系，朋友圈社交网络，广告推广系统。Neo4J，InfoGrid

#### 二、初识Redis（REmote DIctionary Server）远程字典服务器：

Redis 是一个基于键值的存储服务系统。

**安装redis：**

- 下载 Linux 版 redis 并拷贝至虚拟机中
- 使用  `tar -zxvf` 命令解压文件
- 解压成功后，进入 redis 解压目录安装 redis 。注意：安装 redis 需要 `gcc` C语言编译器
- 使用 `yum install gcc-c++` 安装 gcc 编译器
- 使用 `make` 命令编译 redis，如果 `make` 后还有错运行 `make distclean` 之后再 make
- 使用 `make install` 完成最终安装 `make PREFIX=/usr/local/redis install` 安装到指定目录

**配置redis：**

默认情况下 redis 启动后无法进行其它操作，此时需要更改 redis.conf 配置文件。注意：修改前先备份配置文件。

```conf
################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize yes
```

使用 vi 命令打开 redis.conf 文件，将 `daemonize no` 修改为 `daemonize yes` 。在启动 redis 服务时，让 redis 加载修改后的配置文件。

```linux
./redis-service redis-conf
```

使用 ./ 运行服务，后面跟配置文件路径。

**设置 Redis 开机自启：**

- 进入 redis 解压文件夹，并进入 utils 文件夹

    ```linux
    cd /usr/Redis/redis-3.2.1/utils/
    ```

- 拷贝 redis_init_script 文件到 /etc/init.d 目录，并修改文件名为 redis

    ```linxu
    cp redis_init_script /etc/init.d/
    cd /etc/init.d/
    mv redis_init_script redis
    ```

- 修改配置文件

    ```linux
    vi redis
    ```

    ```linux
    #!/bin/sh
    #
    # Simple Redis init.d script conceived to work on Linux systems
    # as it does use of the /proc filesystem.
    # chkconfig: 2345 90 10
    # description: Redis is a persistent key-value database
    
    REDISPORT=6379
    EXEC=/usr/local/redis/bin/redis-server
    CLIEXEC=/usr/local/redis/bin/redis-cli
    
    PIDFILE=/var/run/redis_${REDISPORT}.pid
    CONF="/usr/local/redis/redis.conf"
    auth=123456
    
    case "$1" in
        start)
            if [ -f $PIDFILE ]
            then
                    echo "$PIDFILE exists, process is already running or crashed"
            else
                    echo "Starting Redis server..."
                    $EXEC $CONF
            fi
            ;;
        stop)
    "redis" 44L, 1189C
    ```

    > 文件中需要修改：
    >
    > REDISPORT=6379								#默认端口
    > EXEC=/usr/local/redis/bin/redis-server		#redis 服务所在路径
    > CLIEXEC=/usr/local/redis/bin/redis-cli			#redis 客户端所在目录
    >
    > PIDFILE=/var/run/redis_${REDISPORT}.pid
    > CONF="/usr/local/redis/redis.conf"			#redis 配置文件所在目录
    >
    > auth=123456    #设置密码
    >
    > **注意：** 
    >
    > 文件头部还需要添加下面两行配置
    >
    > \# chkconfig: 2345 90 10
    >
    > \# description: Redis is a persistent key-value database

- 修改读写权限

    ```linxun
    chmod +x /etc/init.d/redis
    ```

- 设置开机启动

    ```
    chkconfig redis on
    chkconfig --add redis
    ```

#### 三、redis配置文件

redis.conf 配置项说明如下：

1. Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程

​    `daemonize no` 更改为 `daemonize yes` 可以让 redis 以守护进程的方式启动

2. 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定

​    **pidfile /var/run/redis.pid** 

3.  指定Redis监听端口，默认端口为6379

​    `port 6379`

4. 绑定的主机地址

​    **bind 127.0.0.1** 为了能够让开发时连接到redis数据库，可以将该条设置注释掉

5. 当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能

​    **timeout 300**

6. 指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose

​    **loglevel verbose**

7. 日志记录方式，默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给/dev/null

​    **logfile stdout**

8. 设置数据库的数量，默认数据库为0，可以使用SELECT \<dbid> 命令在连接上指定数据库id

​    **databases 16**

9.  **指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合**

​    **save \<seconds> \<changes>**

​    **Redis默认配置文件中提供了三个条件：**

​    **save 900 1**

​    **save 300 10**

​    **save 60 10000**

​    分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改。

10. 指定存储至本地数据库时是否压缩数据，默认为yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

​    **rdbcompression yes**

11. **指定本地数据库文件名，默认值为dump.rdb**

​    **dbfilename dump.rdb**

12. 指定本地数据库存放目录

​    **dir ./**

13. 设置当本机为slav服务时，设置master服务的IP地址及端口，在Redis启动时，它会自动从master进行数据同步

​    **slaveof \<masterip> \<masterport>**

14. 当master服务设置了密码保护时，slav服务连接master的密码

​    **masterauth \<master-password>**

15. 设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过AUTH <password>命令提供密码，默认关闭**

​    **requirepass foobared**

16. 设置同一时间最大客户端连接数，默认无限制，Redis可以同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis会关闭新的连接并向客户端返回max number of clients reached错误信息

​    **maxclients 128**

17. 指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到期的Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis新的vm机制，会把Key存放内存，Value会存放在swap区

​    **maxmemory \<bytes>**

18. 指定是否在每次更新操作后进行日志记录，Redis在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis本身同步数据文件是按上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no

​    **appendonly no**

19. 指定更新日志文件名，默认为appendonly.aof

​     **appendfilename appendonly.aof**

20. 指定更新日志条件，共有3个可选值：      **n**o：表示等操作系统进行数据缓存同步到磁盘（快）      **alwa**ys：表示每次更新操作后手动调用fsync()将数据写到磁盘（慢，安全）      **every**sec：表示每秒同步一次（折中，默认值）

​    **appendfsync everysec**

21. 指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

​     **vm-enabled no**

22. 虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

​     **vm-swap-file /tmp/redis.swap**

23. 将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

​     **vm-max-memory 0**

24. Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

​     **vm-page-size 32**

25. 设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

​     **vm-pages 134217728**

26. 设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

​     **vm-max-threads 4**

27. 设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

​    **glueoutputbuf yes**

28. 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

​    **hash-max-zipmap-entries 64**

​    **hash-max-zipmap-value 512**

29. 指定是否激活重置哈希，默认为开启（后面在介绍Redis的哈希算法时具体介绍）

​    **activerehashing yes**

30. 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件

​    **include /path/to/local.conf**

启动 redis 服务：

```shell
./bin/redis-server ./redis.con
```

客户端密码登录redis

```shell
用redis-cli 密码登陆（redis-cli -a  password） 
```

远程服务器访问

```shell
redis-cli –h IP地址 –p 端口 –a 密码
```

关闭redis，正常关闭 redis 也会做持久化操作

```shell
./bin/redis-cli shutdown     关闭redis服务，通过客户端进行shutdown
```

> redis获取中文乱码：在 redis-cli 后面加上 --raw 即可解决
>
> ./bin/redis-cli --raw -a 123456

#### 四、redis 基本命令

##### 1. redis 键（key）

| redis  命令              | 描述                                                      |
| ------------------------ | --------------------------------------------------------- |
| DEL key                  | 删除指定的key                                             |
| DUMP key                 | 序列化给定的key，并返回序列化的值                         |
| EXISTS key               | 检查指定的key是否存在                                     |
| EXPIRE key seconds       | 为给定key设置过期时间（单位：秒）                         |
| PEXPIRE key milliseconds | 设置key的过期时间（单位：毫秒）                           |
| TTL key                  | 返回指定key的剩余生存时间（单位：秒）-1：永久    -2：无效 |
| PTTL key                 | 返回指定key的剩余生存时间（单位：毫秒）                   |
| PERSIST                  | 移除key的过期时间，key将持久保持                          |
| KEYS pattern             | 查找所有符合给定模式的key                                 |
| MOVE key db              | 将当前数据库中的key移动到给定的数据库db中                 |
| RENAME  key  newkey      | 修改key的名称                                             |
| RENAMENX  key  newkey    | 仅当newkey不存在时，将key改名为newkey                     |
| TYPE  key                | 返回key所存储的值的类型                                   |

##### 2. redis key 的命名建议

redis 单个 key 可支持512M的大小

1. key 不要太长，尽量不要超过 1024 字节，这不仅消耗内存，而且会降低查找的效率
2. key 不要太短，太短的话，key 的可读性会降低
3. 在一个项目中 key 最好使用同一的命名模式，例如 user:123:password
4. key 的名称区分大小写

> 命名建议，使用冒号间隔：user:1:mane	user:2:age

#### 五、redis 命令

##### 1. Redis 字符串（string）

string 是 redis 最基本的数据类型，一个 key 对应一个 value。string 类型是二进制安全的，可以包含任何数据，一个键最大能存储512M。

| Redis 命令              | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| SET key value           | 设置指定 key 的值                                            |
| GET key                 | 获取指定 key 的值                                            |
| GETRANGE key start end  | 返回 key 中字符串的子字符                                    |
| GETSET key value        | 将给定 key 的值设为 value ，并返回 key 的旧值(old value)     |
| SETNX key value         | 只有在 key 不存在时设置 key 的值                             |
| STRLEN key              | 返回 key 所储存的字符串值的长度                              |
| INCR key                | 将 key 中储存的数字值增一                                    |
| INCRBY key increment    | 将 key 所储存的值加上给定的增量值（increment）               |
| DECR key                | 将 key 中储存的数字值减一                                    |
| DECRBY key decrement    | key 所储存的值减去给定的减量值（decrement）                  |
| APPEND key value        | 如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾 |
| SETEX key seconds value | 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位) |

应用场景：

1. string 通常用来保存单个字符串或JSON字符串
2. stirng是二进制安全的，完全可以把一个图片的内容作为字符串来存储
3. 计数器（常规计数：微博数、粉丝数）

##### 2. Redis 哈希（Hash）

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

| Redis 命令                               | 描述                                                |
| ---------------------------------------- | --------------------------------------------------- |
| HDEL key field1 [field2]                 | 删除一个或多个哈希表字段                            |
| HSET key field value                     | 将哈希表 key 中的字段 field 的值设为 value          |
| HSETNX key field value                   | 只有在字段 field 不存在时，设置哈希表字段的值       |
| HMSET key field1 value1 [field2 value2 ] | 同时将多个 field-value (域-值)对设置到哈希表 key 中 |
| HGET key field                           | 获取存储在哈希表中指定字段的值                      |
| HGETALL key                              | 获取在哈希表中指定 key 的所有字段和值               |
| HEXISTS key field                        | 查看哈希表 key 中，指定的字段是否存在               |
| HINCRBY key field increment              | 为哈希表 key 中的指定字段的整数值加上增量 increment |
| HKEYS key                                | 获取所有哈希表中的字段                              |
| HLEN key                                 | 获取哈希表中字段的数量                              |

##### 3. Redis 列表（List）

Redis列表是简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部（左边）或者尾部（右边）

| Redis 命令                           | 描述                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| LPUSH key value1 [value2]            | 将一个或多个值插入到列表头部                                 |
| LPUSHX key value                     | 将一个值插入到已存在的列表头部                               |
| RPUSH key value1 [value2]            | 在列表尾部添加一个或多个值                                   |
| RPUSHX key value                     | 为已存在的列表尾部添加值                                     |
| LPOP key                             | 移出并获取列表的第一个元素                                   |
| RPOP key                             | 移除列表的最后一个元素，返回值为移除的元素                   |
| LLEN key                             | 获取列表长度                                                 |
| LTRIM key start stop                 | 对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除 |
| LRANGE key start stop                | 获取列表指定范围内的元素，0表示第一个，1表示第二个，-1表示最后一个元素，-2表示倒数第二个元素 |
| LINDEX key index                     | 通过索引获取列表中的元素                                     |
| LINSERT key BEFORE/AFTER pivot value | 在列表的元素前或者后插入元素                                 |

##### 4. Redis 集合（Set）

Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。

Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)

| Redis 命令                                     | 描述                                                |
| ---------------------------------------------- | --------------------------------------------------- |
| SADD key member1 [member2]                     | 向集合添加一个或多个成员                            |
| SCARD key                                      | 获取集合的成员数                                    |
| SDIFF key1 [key2]                              | 返回给定所有集合的差集                              |
| SINTER key1 [key2]                             | 返回给定所有集合的交集                              |
| SUNION key1 [key2]                             | 返回所有给定集合的并集                              |
| SMEMBERS key                                   | 返回集合中的所有成员                                |
| SMOVE source destination member                | 将 member 元素从 source 集合移动到 destination 集合 |
| SREM key member1 [member2]                     | 移除集合中一个或多个成员                            |
| SPOP key                                       | 移除并返回集合中的一个随机元素                      |
| SRANDMEMBER key [count]                        | 返回集合中一个或多个随机数                          |
| SSCAN key cursor [MATCH pattern] [COUNT count] | 迭代集合中的元素                                    |

##### 5. Redis 有序集合（sorted set）

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复

| Redis 命令                                     | 描述                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| ZADD key score1 member1 [score2 member2]       | 向有序集合添加一个或多个成员，或者更新已存在成员的分数       |
| ZCARD key                                      | 获取有序集合的成员数                                         |
| ZCOUNT key min max                             | 计算在有序集合中指定区间分数的成员数                         |
| ZLEXCOUNT key min max                          | 在有序集合中计算指定字典区间内成员数量   对于一个所有成员的分值都相同的有序集合键 key 来说， 这个命令会返回该集合中， 成员介于 min 和 max 范围内的元素数量 |
| ZINCRBY key increment member                   | 有序集合中对指定成员的分数加上增量 increment                 |
| ZRANGE key start stop [WITHSCORES]             | 通过索引区间返回有序集合成指定区间内的成员                   |
| ZRANGEBYLEX key min max [LIMIT offset count]   | 通过字典区间返回有序集合的成员                               |
| ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] | 通过分数返回有序集合指定区间内的成员                         |
| ZRANK key member                               | 返回有序集合中指定成员的索引                                 |
| ZREM key member [member ...]                   | 移除有序集合中的一个或多个成员                               |
| ZREMRANGEBYLEX key min max                     | 移除有序集合中给定的字典区间的所有成员                       |
| ZREMRANGEBYRANK key start stop                 | 移除有序集合中给定的排名区间的所有成员                       |
| ZREMRANGEBYSCORE key min max                   | 移除有序集合中给定的分数区间的所有成员                       |
| ZREVRANGE key start stop [WITHSCORES]          | 返回有序集中指定区间内的成员，通过索引，分数从高到底         |
| ZREVRANGEBYSCORE key max min [WITHSCORES]      | 返回有序集中指定分数区间内的成员，分数从高到低排序           |
| ZREVRANK key member                            | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| ZSCORE key member                              | 返回有序集中，成员的分数值                                   |
| ZSCAN key cursor [MATCH pattern] [COUNT count] | 迭代有序集合中的元素（包括元素成员和元素分值）               |

##### 6. Redis HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

| Redis 命令                                | 描述                                      |
| ----------------------------------------- | ----------------------------------------- |
| PFADD key element [element ...]           | 添加指定元素到 HyperLogLog 中             |
| PFCOUNT key [key ...]                     | 返回给定 HyperLogLog 的基数估算值         |
| PFMERGE destkey sourcekey [sourcekey ...] | 将多个 HyperLogLog 合并为一个 HyperLogLog |

##### 7. Redis 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 客户端可以订阅任意数量的频道

| Redis 命令                                  | 描述                             |
| ------------------------------------------- | -------------------------------- |
| PSUBSCRIBE pattern [pattern ...]            | 订阅一个或多个符合给定模式的频道 |
| PUBSUB subcommand [argument [argument ...]] | 查看订阅与发布系统状态           |
| PUBLISH channel message                     | 将信息发送到指定的频道           |
| PUNSUBSCRIBE [pattern [pattern ...]]        | 退订所有给定模式的频道           |
| SUBSCRIBE channel [channel ...]             | 订阅给定的一个或多个频道的信息   |
| UNSUBSCRIBE [channel [channel ...]]         | 退订给定的频道                   |

##### 8. Redis 事物

Redis 事务可以一次执行多个命令， 并且带有以下重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

先以 **MULTI** 开始一个事务， 然后将多个命令入队到事务中， 最后由 **EXEC** 命令触发事务， 一并执行事务中的所有命令。

| Redis 命令          | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| MULTI               | 标记一个事务块的开始                                         |
| DISCARD             | 取消事务，放弃执行事务块内的所有命令                         |
| EXEC                | 执行所有事务块内的命令                                       |
| UNWATCH             | 取消 WATCH 命令对所有 key 的监视                             |
| WATCH key [key ...] | 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断 |



#### 六、Java 连接 Redis

1. 添加 Jedis 依赖

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>3.0.1</version>
    </dependency>
    ```

2. 代码连接redis

    ```java
    /**
     * Java端操作Redis服务器
     */
    public static void main(String[] args) {
        //1、连接redis服务器
        String host = "192.168.174.131";        //连接地址
        int port = 6379;                        //端口号
        Jedis jedis = new Jedis(host, port);    //创建Jedis对象
        jedis.auth("123456");          //设置密码
        System.out.println(jedis.ping());
    }
    ```

3. 创建 RedisUtil 工具类

    ```java
    public class RedisPoolUtil {
        private static JedisPool jedisPool;
        static {
            //1、连接池基本配置信息
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            jedisPoolConfig.setMaxTotal(5);   //最大连接数
            jedisPoolConfig.setMaxIdle(1);    //最大空闲数
            //2、创建连接池
            String host = "192.168.174.131";
            int port = 6379;
            jedisPool = new JedisPool(jedisPoolConfig, host, port);
        }
        public static Jedis getJedis(){
            Jedis resource = jedisPool.getResource();
            resource.auth("123456");
            return resource;
        }
        /**
         * 关闭连接
         * @param jedis 连接
         */
        public static void close(Jedis jedis){
            jedis.close();
        }
    }
    ```

4. 测试 Hash 

    ```java
    @Test
    public void test04() throws IllegalAccessException, InvocationTargetException {
        Jedis jedis = RedisPoolUtil.getJedis();
        String id = "1";
        String key = User.getKeyName() + id;
        if (jedis.exists(key)) {
            //redis中取出对象
            Map<String, String> map = jedis.hgetAll(key);
            User user = new User();
            BeanUtils.populate(user, map);//把map集合转换成对象
            System.out.println(user);
        } else {
            //数据库查询
            User user = new User();
            user.setId(1);
            user.setName("王小明");
            user.setAge(22);
            user.setGender("男");
            Map<String, String> map = new HashMap<>();
            Class<? extends User> userClass = user.getClass();
            Field[] declaredFields = userClass.getDeclaredFields();
            for (Field declaredField : declaredFields) {
                declaredField.setAccessible(true);//设置属性是可以访问的
                String name = declaredField.getName();//获取属性名
                Object o = declaredField.get(user);//获取属性值
                map.put(name, String.valueOf(o));//通过反射机制将对象转成map集合
            }
            jedis.hset(key, map);
        }
        RedisPoolUtil.close(jedis);
    }
    ```

#### 七、Spring-data 整合 Redis

1. 导入相关依赖，注意 jar 包之间的依赖冲突

    ```xml
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.10.2</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>2.1.8.RELEASE</version>
    </dependency>
    ```

2. 对实体Bean进行序列化操作

    ```java
    public class User implements Serializable
    ```

3. 编写相应的配置文件

    ```java
    @Configuration
    @ComponentScan(value = {"com.wf.redis"})
    public class SpringRedisConfig {
        /**
         * 1、配置连接池信息
         * @return JedisPoolConfig
         */
        @Bean
        public JedisPoolConfig jedisPoolConfig(){
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            jedisPoolConfig.setMaxTotal(20);//最大连接数
            jedisPoolConfig.setMaxIdle(5);//最大空闲数
            return jedisPoolConfig;
        }
        /**
         * 2、Spring整合Jedis
         * @return JedisConnectionFactory
         */
        @Bean
        public JedisConnectionFactory jedisConnectionFactory(JedisPoolConfig jedisPoolConfig){
            RedisStandaloneConfiguration configuration = new RedisStandaloneConfiguration();
            configuration.setHostName("192.168.174.131");
            configuration.setPort(6379);
            configuration.setPassword(RedisPassword.of("123456"));
            //获得默认的连接池构造器
            JedisClientConfiguration.JedisPoolingClientConfigurationBuilder builder =
                    (JedisClientConfiguration.JedisPoolingClientConfigurationBuilder) JedisClientConfiguration.builder();
            //指定jedisPoolConifig来修改默认的连接池构造器
            builder.poolConfig(jedisPoolConfig);
            //通过构造器来构造jedis客户端配置
            JedisClientConfiguration jedisClientConfiguration = builder.build();
            return new JedisConnectionFactory(configuration, jedisClientConfiguration);
        }
        /**
         * 3、RedisTemplate
         * @param factory JedisConnectionFactory
         * @return RedisTemplate
         */
        @Bean
        public RedisTemplate<String, Object> redisTemplate(JedisConnectionFactory factory) {
            RedisTemplate redisTemplate = new RedisTemplate();
            redisTemplate.setConnectionFactory(factory);
            Jackson2JsonRedisSerializer<Object> json = new Jackson2JsonRedisSerializer<>(Object.class);
            //redis键值序列化方式
            StringRedisSerializer string = new StringRedisSerializer();
            redisTemplate.setKeySerializer(string);
            redisTemplate.setValueSerializer(string);
            return redisTemplate;
        }
    }
    ```

#### 八、Redis 持久化

##### 1. RDB

RDB 是 redis 默认的持久化机制，RDB 相当于快照，保存的是一种状态。这种方式就是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为 dump.rdb

优点：快照保存数据极快、还原数据极快，使用于灾难备份

缺点：小内存机器不适合使用，RDB 机制符合要求就会照快照

快照条件：

- 服务器正常关闭 `./redis-cli shutdown` 

- key 满足一定条件

    ```shell
    save 900 1		#每900秒至少一个key发生变化
    save 300 10		#每300秒至少10个key发生变化
    save 60 10000	#每60秒至少10000个key发生变化
    ```

##### 2. AOF

由于快照方式是在一定间隔时间做一次的，如果 redis 意外宕机就会丢失最后一次快照的所有修改，如果应用要求不能丢失任何一次快照后的所有修改可以采用 AOF 持久化方式。

在使用 AOF 持久化方式时，redis 会将每一个收到的写命令都通过 write 函数追加到文件中(默认是 appendonly.aof)当 redis 重启时会通过重新执行文件中保存的写命令在内存中重建整个数据库的内容。

AOF 方式如下：

```shell
appendonly yes		#启用AOF持久化方式
appendfsync always	#收到命令就立刻写入磁盘，最慢，但是能保证完全的持久化
appendfsync everysec	#每秒写入磁盘一次，在性能和持久化方面做了很好的折中
appendfsync no			#完全依赖 os，性能最好，持久化没保证
```

缺点：持久化文件会越来越大

#### 九、Redis 主从复制

一次存储，多次读写，读多写少。

一个Redis服务可以有多个该服务的复制品，这个Redis服务称为Master，其它复制称为Slaves。

主从复制的好处：

- 读写分离，不仅可以提高服务器的负载能力，并且可以根据读请求的规模自由增加或减少从库的数量
- 数据被复制成了好几份，一台机器出现故障可以从其它机器快速恢复

Redis 主从复制配置：

要实现主从复制，只需要复制 redis.conf 的配置文件，并指定主机的地址和端口号即可。

1. 在从库中配置文件中修改端口号并指定主库的地址和端口号

    ```shell
    port 6380				  #从服务的端口号
    slaveof 127.0.0.1 6379		#指定主服务器
    ```

2.  配置文件：

    ```shell
    ##########redis-6379.conf配置文件如下：
    #主表的配置文件
    # Redis使用后台模式
    daemonize yes
    # 注释以下内容开启远程访问
    # bind 127.0.0.1
    # 修改启动端口为6379
    port 6379
    # 修改pidfile指向路径--Redis以守护进程方式运行时把pid写入文件
    pidfile /usr/local/redis-3.0.0/conf/redis_6379.pid
    #数据库的存放位置 自己定义
    dir /usr/local/redis-3.0.4/db/master/
    ```

    ```shell
    ##########redis-6380.conf配置文件如下：
    # Redis使用后台模式
    daemonize yes
    # 关闭保护模式
    #protected-mode no
    # 注释以下内容开启远程访问
    # bind 127.0.0.1
    # 修改启动端口为6380
    port 6380
    # 修改pidfile指向路径
    pidfile /usr/local/redis-3.0.0/conf/redis_6380.pid
    #数据库的存放位置
    dir /usr/local/redis-3.0.4/db/slave_one
    #Slaveof命令可以将当前服务器转变为指定服务器的从属服务器(slave server)。
    slaveof 127.0.0.1 6379
    ```

    ```shell
    ##########redis-6381.conf配置文件如下:
    # Redis使用后台模式
    daemonize yes
    # 关闭保护模式
    #protected-mode no
    # 注释以下内容开启远程访问
    # bind 127.0.0.1
    # 修改启动端口为6381
    port 6381
    # 修改pidfile指向路径
    pidfile /usr/local/redis-3.0.0/conf/redis_6381.pid
    #数据库的存放位置
    dir /usr/local/redis-3.0.4/db/slave_two/
    #Slaveof命令可以将当前服务器转变为指定服务器的从属服务器(slave server)。
    slaveof 127.0.0.1 6379
    ```

3.  启动 Redis 服务

    ```shell
    /usr/local/bin/redis-server /usr/local/redis-3.0.4/conf/redis-6379.conf 
    /usr/local/bin/redis-server /usr/local/redis-3.0.4/conf/redis-6380.conf 
    /usr/local/bin/redis-server /usr/local/redis-3.0.4/conf/redis-6381.conf 
    ```


#### 十、Redis 集群

##### 1、Redis 集群搭建方案

- Twitter 开发的 twemproxy
- 豌豆荚开发的 codis
- redis 官方的 redis-cluster

从 redis 3.0 之后版本支持  redis-cluster 集群，至少需要 3（Master）+ 3（Slave）才能建立集群。redis-cluster采用无中心结构，每个节点保存数据和整个集群状态，每个节点都和其它所有节点连接。

##### 2、Redis Cluster集群特点

- 所有的redis节点彼此互联（PING - PONG机制），内部使用二进制协议优化传输速度和带宽
- 节点的 fail 是通过集群中超过半数的节点检测失效时才生效
- 客户端与 redis 节点直连，不需要中间 proxy 层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可
- redis-cluster 把所有的物理节点映射到【0 ~ 16383】slot 上（不一定是平均分配），cluster 负责维护
- redis 集群预分好 16384 个哈希槽，当需要在 redis 集群中放置一个 key-value 时，redis 先对 key 使用 crc16 算法算出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0 ~ 16383 之间的哈希值槽，redis 会根据节点数量大致均等的将哈希槽映射到 不同节点

##### 3、Redis 容错

投票过程是集群中所有 master 参与，如果半数以上 master 节点与 master 节点通信超时，认为当前 master 节点挂掉。如果集群中任意 master 挂掉，且当前 master 没有 slave ，集群进入 fail 状态，也可以理解为集群的 slave 映射不完整时【0 ~ 16383】进入 fail 状态，如果集群超过半数以上 master 挂掉，无论是否有 slave ，集群进入 fail 状态

##### 4、Redis Cluster 集群搭建

集群中至少应该有奇数个节点（Master），搭建集群至少需要3台主机，同时每个节点至少有一个备份节点。

集群搭建步骤：（一台服务器中启动多个redis服务）

1. 创建 redis 节点安装目录

    ```shell
    mkdir /usr/local/redis_cluster		#指定目录下创建redis_cluster
    ```

2. 在 redis_cluster 下创建 7001 - 7006 个文件夹

    ```shell
    mkdir 7001 7002 7003 7004 7005 7006
    ```

3. 将 redis.conf 分别拷贝到这 6 个文件夹中

4. 分别修改配置文件（以 7001 为例）

    ```shell
    # 1、注释掉 ip 绑定
    # bind 127.0.0.1
    
    # 2、修改端口号（不同主机可不修改）
    port 7001
    
    # 3、设置后台启动
    daemonize yes
    
    # 4、修改 pidfile 路径
    pidfile /var/run/redis_7001.pid
    
    # 5、开启集群支持
    cluster-enabled yes
    
    # 6、配置每个节点的配置文件
    cluster-config-file nodes-7001.conf
    
    # 7、配置集群节点响应时间
    cluster-node-timeout 15000
    
    # 配置密码
    masterauth 123456
    requirepass 123456
    ```

5. 启动各个 redis 节点

    ```shell
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7001/redis.conf
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7002/redis.conf
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7003/redis.conf
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7004/redis.conf
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7005/redis.conf
    ./redis/redis-5.0.3/src/redis-server redis_cluster/7006/redis.conf
    ```

6. 启动集群

    ```shell
    ./redis/bin/redis-cli -a 123456 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
    
    # -a 输入密码
    # --cluster create 创建集群
    # --cluster-replicas 每个创建的主服务器都有一个从服
    ```
    
    ```shell
    Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
    >>> Performing hash slots allocation on 6 nodes...
    Master[0] -> Slots 0 - 5460
    Master[1] -> Slots 5461 - 10922
    Master[2] -> Slots 10923 - 16383
    Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
    Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
    Adding replica 127.0.0.1:7006 to 127.0.0.1:7003
    >>> Trying to optimize slaves allocation for anti-affinity
    [WARNING] Some slaves are in the same host as their master
    M: ef8cc77f1e29cda1dead9ec0f69b447e65d75569 127.0.0.1:7001
       slots:[0-5460] (5461 slots) master
    M: fd322f719fdf43180b93cd39e282bcbf6a045b81 127.0.0.1:7002
       slots:[5461-10922] (5462 slots) master
    M: c2cd0ea3648ea528c763a647f0e6e942fe824846 127.0.0.1:7003
       slots:[10923-16383] (5461 slots) master
    S: 3e2697fa7df4bc7591362db996477fd8501b4ba2 127.0.0.1:7004
       replicates fd322f719fdf43180b93cd39e282bcbf6a045b81
    S: f4d92fe2efa6d123850a0cbc6769f6d35fc94746 127.0.0.1:7005
       replicates c2cd0ea3648ea528c763a647f0e6e942fe824846
    S: fdf06cc47550a1ce36a29ff2d38ffe2f6ab516fb 127.0.0.1:7006
       replicates ef8cc77f1e29cda1dead9ec0f69b447e65d75569
    Can I set the above configuration? (type 'yes' to accept): yes
    >>> Nodes configuration updated
    >>> Assign a different config epoch to each node
    >>> Sending CLUSTER MEET messages to join the cluster
    Waiting for the cluster to join
    ....
    >>> Performing Cluster Check (using node 127.0.0.1:7001)
    M: ef8cc77f1e29cda1dead9ec0f69b447e65d75569 127.0.0.1:7001
       slots:[0-5460] (5461 slots) master
       1 additional replica(s)
    S: fdf06cc47550a1ce36a29ff2d38ffe2f6ab516fb 127.0.0.1:7006
       slots: (0 slots) slave
       replicates ef8cc77f1e29cda1dead9ec0f69b447e65d75569
    M: c2cd0ea3648ea528c763a647f0e6e942fe824846 127.0.0.1:7003
       slots:[10923-16383] (5461 slots) master
       1 additional replica(s)
    M: fd322f719fdf43180b93cd39e282bcbf6a045b81 127.0.0.1:7002
       slots:[5461-10922] (5462 slots) master
       1 additional replica(s)
    S: 3e2697fa7df4bc7591362db996477fd8501b4ba2 127.0.0.1:7004
       slots: (0 slots) slave
       replicates fd322f719fdf43180b93cd39e282bcbf6a045b81
    S: f4d92fe2efa6d123850a0cbc6769f6d35fc94746 127.0.0.1:7005
       slots: (0 slots) slave
       replicates c2cd0ea3648ea528c763a647f0e6e942fe824846
    [OK] All nodes agree about slots configuration.
    >>> Check for open slots...
    >>> Check slots coverage...
    [OK] All 16384 slots covered.
    ```

##### 5、Redis Cluster 集群验证

1. 连接集群

    ```shell
    ./redis/bin/redis-cli -h 127.0.0.1 -p 7001 -c
    # -h IP地址
    # -p 端口号
    # -c 连接到集群
    127.0.0.1:7001> auth 123456
    ```

    > 注意：如果设置了 Redis 的密码 requirepass ，连接集群的时候需要在命令后面跟上 `-a [password]` 否则如果 redis 进行重定向保存数据的时候目标 redis 库会因为没有密码而保存失败

2. 查看集群节点信息

    ```shell
    127.0.0.1:7001> info replication
    # Replication
    role:master
    connected_slaves:1
    slave0:ip=127.0.0.1,port=7006,state=online,offset=210,lag=0
    master_replid:389d1d968d0ddb4a4589202dbe3bfef9c3b08b10
    master_replid2:0000000000000000000000000000000000000000
    master_repl_offset:210
    second_repl_offset:-1
    repl_backlog_active:1
    repl_backlog_size:1048576
    repl_backlog_first_byte_offset:1
    repl_backlog_histlen:210
    ```

    ```shell
    127.0.0.1:7001> cluster nodes
    99ff458d7fbd1ba3c9e82eea7dd45a8d5246265b 127.0.0.1:7004@17004 slave cd7d0a0b12300a9fc7aea9e8d442d98eb1454919 0 1558760465000 4 connected
    
    cd7d0a0b12300a9fc7aea9e8d442d98eb1454919 127.0.0.1:7002@17002 master - 0 1558760462487 2 connected 5461-10922
    
    74f8c850ce3a93a411da2c0d9033d5db3a8d6a15 127.0.0.1:7005@17005 slave dffb1fca664d421ff19628070d1f32904e393ab0 0 1558760462000 5 connected
    
    dffb1fca664d421ff19628070d1f32904e393ab0 127.0.0.1:7003@17003 master - 0 1558760463493 3 connected 10923-16383
    
    22992be6506feac45c57113b0b249d75c65e3722 127.0.0.1:7001@17001 myself,master - 0 1558760463000 1 connected 0-5460
    
    59532b019f2b093aa9801deaf1e1c6afd4eddbcf 127.0.0.1:7006@17006 slave 22992be6506feac45c57113b0b249d75c65e3722 0 1558760465506 6 connected
    ```


#### 十一、Java 连接 Redis 集群

使用 `JedisCluster` 连接 Redis 集群。添加依赖：

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.0.1</version>
</dependency>
```

```java
@Test
public void redisTest(){

    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(20);     //最大连接个数
    config.setMaxIdle(5);       //最大空闲连接个数
    config.setMaxWaitMillis(-1);//获取连接时的最大等待毫秒数，若超时则抛异常。-1代表不确定的毫秒数

    HashSet<HostAndPort> node = new HashSet<>();
    node.add(new HostAndPort("192.168.174.131", 7001));
    node.add(new HostAndPort("192.168.174.131", 7002));
    node.add(new HostAndPort("192.168.174.131", 7003));
    node.add(new HostAndPort("192.168.174.131", 7004));
    node.add(new HostAndPort("192.168.174.131", 7005));
    node.add(new HostAndPort("192.168.174.131", 7006));
    //参数说明：节点信息、连接超时、数据读取超时、最大尝试次数、密码、连接池配置信息
    JedisCluster jedisCluster = new JedisCluster(node, 15000, 5000, 10, "123456", config);
    jedisCluster.set("name", "张三");
    System.out.println(jedisCluster.get("name"));

}
```

> **注意：** 如果集群设置有密码，使用 new JedisCluster(node, 15000, 5000, 10, "123456", config) 创建连接。
>
> **问题：** 创建连接后无法进行 set、get 操作。查看异常信息后发现获取的集群信息为 127.0.0.1:7001 程序在得到节点信息后并不是访问虚拟机中的节点插入数据，而是本地 redis，在本地找不到 redis 进而抛出异常。
>
> **解决方法：** 关闭所有 redis 节点，删除 dump.rdb 和 node-7001.conf 等文件，重新创建 redis 集群。将 `./redis/bin/redis-cli -a 123456 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1` 命令改为 `./redis/bin/redis-cli -a 123456 --cluster create 192.168.174.131:7001 192.168.174.131:7002 192.168.174.131:7003 192.168.174.131:7004 192.168.174.131:7005 192.168.174.131:7006 --cluster-replicas 1` 

#### 十二、Spring 整合 redisTemplate 连接 Redis 集群

1. 添加依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.10.2</version>
</dependency>

<!-- spring-data-redis -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.1.8.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.1.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.54</version>
</dependency>
```

2. 编写配置方法

```java
@Configuration
@PropertySource("classpath:redis_cluster.properties")
public class SpringRedisClusterConfig {

    @Value("${redis.cluster.nodes}")
    private String nodes;
    @Value("${redis.cluster.password}")
    private String password;
    @Value("${redis.connectionTimeout}")
    private Integer connectionTimeout;
    @Value("${redis.soTimeout}")
    private Integer soTimeout;
    @Value("${redis.maxAttempts}")
    private Integer maxAttempts;
    @Value("${redis.maxTotal}")
    private Integer maxTotal;
    @Value("${redis.maxIdle}")
    private Integer maxIdle;

    @Bean
    public JedisPoolConfig jedisPoolConfig() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMaxWaitMillis(-1);
        return config;
    }
    
    @Bean
    public RedisClusterConfiguration redisClusterConfiguration() {
        RedisClusterConfiguration configuration = new RedisClusterConfiguration();
        String[] split = nodes.split(",");
        for (String s : split) {
            String[] node = s.split(":");
            configuration.addClusterNode(new RedisNode(node[0], Integer.parseInt(node[1])));
        }
        configuration.setPassword(RedisPassword.of(password));
        return configuration;
    }
    
    @Bean
    public JedisConnectionFactory jedisConnectionFactory(RedisClusterConfiguration redisClusterConfiguration, JedisPoolConfig jedisPoolConfig){
        return new JedisConnectionFactory(redisClusterConfiguration, jedisPoolConfig);
    }
    
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(JedisConnectionFactory factory){
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        FastJsonRedisSerializer<Object> jsonRedisSerializer = new FastJsonRedisSerializer<>(Object.class);
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}
```

3. 测试

```java
@Test
public void test(){
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(SpringRedisClusterConfig.class);
    RedisTemplate redisTemplate = applicationContext.getBean(RedisTemplate.class);
    ValueOperations<Object, Object> valueOperations = redisTemplate.opsForValue();
    valueOperations.set("name", "李四");
    System.out.println(valueOperations.get("name"));
}
```

