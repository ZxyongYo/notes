# Redis

## 1 Redis 特点

- 1. 支持数据持久化

     Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

  2. 支持多种数据结构

     Redis不仅仅支持简单的key-value类型的数据，同时还提供了list、set、zset、hash等数据结构的存储。

  3. 支持数据备份

## 2 Redis安装

### 2.1.1 Redis服务

- 启动Redis
  1. 前台启动：redis-server
  2. 后台启动：redis-server &
  3. 根据配置文件启动： redis-server redis.conf &
- 关闭Redis服务
  1. redis-cli shutdown ：推荐使用，redis先完成数据操作，然后再关闭。
  2. kill -9 pid ： 不考虑当前应用是否有数据正在执行操作，直接关闭。

### 2.1.2 Redis客户端

- 启动redis客户端：redis-cli （默认 ip127.0.0.1，端口6379）
- 指定ip和端口：redis-cli -h127.0.0.1 -p6379

- 查看redis服务器的统计信息：info [section]
- redis默认使用16个库，从0到15，， 可以在redis.conf文件中更改 databases 16。
- 默认使用第0个数据库，使用 select index 切换

- 查看当前数据库中key的数目：dbsize
- 查看当前数据库中有哪些key： keys *
- 清空当前数据库：flushdb
- 清空所有数据库：flushall

- 获得redis的所有配置值：config get *

### 2.1.3 Redis的5种数据结构

- 字符串类型：string；列表类型：list；集合类型：set；有序集合类型：zset；哈希类型：hash。

## 3.1 Redis常用操作

### 3.1.1 Redis的keys操作

- keys pattern ：查询数据库种的key

  使用通配符： *表示0个或多个字符；？表示单个字符；[ ]表示匹配[ ]内的一个字符，

- exists key [key...]：判断key是否存在；返回一个整数表示存在key的数量；

- move key db ：移动key到指定数据库，成功返回1，失败返回0；

- ttl key：查看key的剩余生存时间，单位秒，返回-1：没有设置key生存时间；返回-2：key不存在；

- expire key seconds：设置key的生存时间，超过时间key自动删除。

- type key：查看key所存储值的数据类型；

- rename key newkey：将key改名为newkey；当key和newkey相同，或者key不存在时，返回一个错误；

  当newkey已经存在，rename命令将覆盖旧值；

- del key [key...]：删除key，不存在的key忽略；返回删除key的数量；

### 3.1.2 字符串类型

- set key value ：将字符串值value添加到key中，如果key已存在，会把之前的值覆盖掉；
- get key：获取key中设置的字符串值，存在返回对应的value，不存在返回nil；
- append key value：如果key存在，将value追加到key原值的末尾，不存在则set key value；
- strlen key：返回key对应value的长度；
- incr / decr key：将key中存储的数字值加 / 减1，如果key不存在，key值会被先初始化为0，再执行incr/decr；

- incrby / decrby key offset：将key中存储的数字值加 / 减offset，如果key不存在，key值会被先初始化为0，再执行；

- getrange key startIndex endIndex：获取字符串值从startIndex到endIndex结束的子字符串，负数代表从末尾开始；
- setrange key offsetIndex value：用value覆盖key存储的值从offset开始；

- setex key seconds value：设置key的值，并将key的生存时间设为seconds，如果key已存在，覆盖旧值；
- setnx key value：如果key不存在，则设置值，存在则不设置；

- mset / mget ：设置/获取一个或多个key

### 3.1.3 列表（list）

- rpush / lpush key value [value...]：将一个或多个value插入到列表key的最右 / 左边，返回加入后列表的长度；
- lrange key startIndex endIndex：获取列表key中指定下标区间内的元素，下标越界不会报错，返回获取到的元素列表；

- lpop / rpop key：移除并返回列表key头/尾的第一个元素；

- lindex key index：获取列表key中下标为index的元素；

- llen key：获取列表key的长度；

- lrem key count value：根据count的值，移除列表中与参数value相等的元素，

  count>0 ：从列表左侧向右移除；count<0：从列表右侧向左移除；count=0：移出列表中所有与value相等的值；

- ltrim key startIndex endIndex：截取key的指定小标区间的元素，并赋值给key；

- lset key index value：将列表key下标为index的元素的值设置为value；

- linsert key before / fater pivot value：将value插入到列表key中pivot之前或之后的位置；

### 3.1.4 集合（set）

set是string类型的无序不重复集合

- sadd key member [member...] ：将一个或多个元素加入到集合key当中，已经存在的元素会被忽略，不会再加入。

  返回加入到集合的新元素的个数；

- smembers key ：获取集合key中的所有成员元素；

- sismember key member ：判断member匀速是否是集合key的元素；

- scard key：获取集合里面的元素个数，返回key的元素个数；

- srem key member [member...]：移除集合中一个或多个元素，返回成功移除元素的个数；

- srandmember key [count]：只提供key，随机返回集合中一个元素，

  提供count时，count正数，返回包含count个数元素的集合，集合元素不重复

  count负数，返回一个count绝对值的长度的集合，集合种元素可能会重复多次；

- spop key [count]：随机从集合中删除一个或count个元素；

- smove src dest member ：将member元素从src集合移动到dest集合，member不存在，smove不执行，

  如果dest存在member，则从src中删除member；

- sdiff key key [key ...]：返回指定集合的差集，以第一个集合为准进行比较，即第一个集合中有但在其他集合中都没有的元素；

- sinter key key [key ....] ：返回指定集合的交集；
- sunion key key [key...]：返回指定几个的并集，如果有元素重复，保留一个；

### 3.1.5 哈希（hash）

Redis的hash是一个string类型的key和value的映射表，这里的value是一系列的键值对，hash适合用于存储对象。

- hset key field value [field value ...]：将键值对设置到哈希表key中，如果key不存在，则新建哈希列表，然后执行赋值，如果key的field已经存在，则value值覆盖。
- hget key field：获取哈希表key中给定域field的值。
- hmget key field [field ...]：获取key中一个或多个域的值；
- hgetall key：获取哈希表key中所有的域和值；
- hdel key field [field ...]：删除哈希表key中的一个或多个指定域field，返回删除成功的数量；
- hlen key：获取哈希表key中field的个数；
- hexists key field：查看哈希表key中给定field是否存在；
- hkeys key：查看哈希表key中所有field；
- hvals key：查看哈希表中所有值；
- hincrby key field int：给哈希表key中的field增加int；
- hincrbyfloat key field float：给哈希表key中的field增加float；
- hsetnx key field value：将哈希表key中的field设为value，当field存在才设置；

### 3.1.6 有序集合（zset）

Redis有序集合zset和集合set一样也是string类型元素的集合，且不允许重复的成员。不同的是zset的每个元素都会关联一个分数（分数可以重复），redis通过分数来为结合中的成员进行从小到大的排序。

- zadd key score member [score member...]：将一个或多个member元素及其score值加入到有序集合key中，如果member存在集合中，则覆盖原来的值，score可以是整数或浮点数；
- zrange key startIndex endIndex [WITHSCORES]：查询有序集合，指定区间内的元素。集合成员按score值从小到大来排序；
- zrangebyscore key min max [WITHSCORES] [LIMIT offset count]：获取有序集合key中，所有介于min和max之间（包括min和max）的成员，

- zrem key member [member...]：删除有序集合key中的一个或多个成员，不存在的成员忽略；
- zcard key：获取有序集合key的元素成员的个数；

- zcount key min max：返回有序集合key中，score值在min和max之间（包括等于min或max）的成员的数量；
- zrank key member：获取有序集合key中成员member的排名；

- ascore key member：获取有序集合key中元素member的分数；
- zrevrank key member：获取有序集合key中成员member的排名，有序集合成员按score值从大到小排列，从0开始排名，score最大是0；
- zrevrange key startIndex endIndex [WITHSCORES]：查询有序集合，指定区间内的元素。

- zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]： 获取有序集合key中，所有score值介于max和min之间的成员；

