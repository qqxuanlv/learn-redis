# Redis

REmote DIctionary Server(Redis) 是一个由 Salvatore Sanfilippo 写的 key-value 存储系统。

Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API。

它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

# Redis 优势

性能极高 – Redis 能读的速度是 110000 次/s,写的速度是 81000 次/s 。
丰富的数据类型 – Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
原子 – Redis 的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过 MULTI 和 EXEC 指令包起来。
丰富的特性 – Redis 还支持 publish/subscribe, 通知, key 过期等等特性。
Redis 与其他 key-value 存储有什么不同？
Redis 有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis 的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。

Redis 运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样 Redis 可以做很多内部复杂性很强的事情。同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。

### 以上 废话结束

# Redis 常见数据类型

## String: 字符串

ps:string 类型采用冗余分配内存，当字符串占用空间少于 1MB 时, 扩容一倍.当空间大于 1MB 时扩容每次只会增加 1mb 最大不超过 512MB.
常用操作：

### 单独 get set

set [key][value]
get [key]

### 批量 get set

mset [key1][value1] [key2][value2] ...
mget [key1][key2] [key3]

### set 扩展 添加过期时间

ps:单独设置过期时间(前提已经存在 key)
expire [key][second]

### 初始化添加键值对的时候设置过期时间

setex [key][second] [value] (setex = set + expire )
psetex [key][milliseconds] [value] (同上 单位毫秒)

### set 扩展 设置前先判断是否已存在（简单的'redis 锁'）

setnx [key][value]

### 计数增长

ps:如果 value 是整数时可以对 value 进行自增操作（自增不能超过 signed long 9223372036854775807 ）
set [age] 0
incr [key] (key 已存在,步长为 1）
incrby [key][num] (增加 num 个数量，其中 num 可以为正负数)

### 计数减少

decr [key]  
decrby [key][number] (同 incrby)

#### 使用场景

存储用户信息 、 计数 、 标识位 、 正常存数据 等等 ， 通常存储用户信息 需要经过序列化 以字符串的形式存入 ，然后反序列化回对象 。（其实可以使用 hash 去存储对象 ， 避免了序列化反序列化操作）。

## List: 列表

ps: Redis 中的 list 相当于 java 里的 linkedList (链表，插入删除时间复杂度 O(1),但是查询慢 O(n) .), 因为 redis 里面提供 lpush ,rpush , lpop , rpop 所以我们可以更加灵活的使用 list

### 向左边插入数据

lpush [key][value]

### 向右边插入数据

rpush [key][value]

### 左边弹出数据

lpop [key][value]

### 右边弹出数据

rpop [key][value]

### 左边阻塞弹出数据 (如果 list 中为空, 则一直等待数据插入，直到 timeout 结束 timeout 时间单位为秒)

blpop [key][timeout]

### 右边阻塞弹出数据 (如果 list 中为空 ,则一直等待数据插入，直到 timeout 结束 timeout 时间单位为秒)

brpop [key][timeout]

### 查看当前队列中长度

llen [key]

### 从一个列表的右边弹出一个值（没有，阻塞）插入到一个新的列表中

brpoplpush [source]

### 同上 不阻塞

rpoplpush source destination
移除列表的最后一个元素，并将该元素添加到另一个列表并返回

### 通过索引设置值

lset [key][index] [value]

### 以索引下标的方式获取值

lindex [key][index]

### 向表头插入数据

lpushx [key][value]

### 以范围查找获取 list 中的数据

lrange [key][startindex] [endIndex]

### 按范围+值匹配移除数据

lrem [key][count] [value]

COUNT 的值可以是以下几种：

count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
count = 0 : 移除表中所有与 VALUE 相等的值。

### 添加多个值

rpush [key][value1] [value2][value3] ...

### 替换已有的值

rpushx [key][value]

#### 总结

我们可以使用 list 提供的各种 api 实现队列（先进先出 ），栈 （后进先出） 或者阻塞队列的特性。
阻塞队列 也可以用来做消息通知 ,但是使用 redis List 作为消息队列 没办法处理消息确认（ack）所以如果你对消息通知不需要那么精准，不在意丢失 的情况下可以使用它。

阻塞队列 blpop 时 如果时间过长 redis 为了节省资源会自动断开一些连接， 所以在阻塞的连接也有可能被断开 这样就会产生异常
。

#### 实现原理

redis 中 list 底层存储结构其实不是简单的 linkedList 二十交快速链表的结构（quickList）,在链表中元素较少的情况下，会使用一块 '连续的内存存储'，这个结构是 ziplist（压缩列表），当数据量大的时候才会改成 quciklist.
这样做的好处是节省空间，一个普通的链表需要记录前驱节点 和后继节点 ， 但是我如果只存一个数据 也需要附加 prev 和 next 所以如果数据量少的情况下 使用连续内存存储那就会剩下 prev 和 next 节点

## Hash: 散列

ps:和 java 中的 hashmap 很像， 都是由数组加链表构成（1.8 之前的） ， redis 中的 hash key 只能存 string 类型,redis 中每个 hash 可以存储 232 - 1 键值对（40 多亿）。

### 设置 hashmap 单个属性值

hset [key][field] [value]

### 获取单个字段的值

hget [key][field]

### 设置多个属性值

hmset [key][field] [value][field2] [value2] ...

### 获取 key 多个字段的值

hmget [key][field1] [field2]

### 删除某个字段,或多个字段

hdel [key][field] [field2] ...

### 获取所有的字段和值

hgetall [key]

### 查询 key 中是否含有某个字段

hexists [key][field]

### 使 hash 的某个字段自增

hincrby [key][field] [increment]

### 使 hash 的某个字段自增（浮点）

hincrbyfloat [key][field] [increment]

### 当字段不存在的时候设置

hsetnx [key][field] [value]

### 获取 hash 表中所有值

hvals key

### 模糊查询

cursor - 游标。
pattern - 匹配的模式。
count - 指定从数据集里返回多少元素，默认值为 10 。

hscan [key][cursor 游标] match ["pattern 例子 m\*"][count]

### 无法使 hash 的单个字段过期失效

#### 原理

redis 为了提高性能 在计算 hash 的时候使用的是渐进的方式计算， （java 中 hashmap 过大时 rehash 很耗时 ，一次性全部 rehash） 在 rehash 的同时保留新旧两种 hash。开启定时任务 一点点把旧的 hash 地址中的值移动到新的 hash 中直到所有旧的 hash 移动完毕 。
当 hash 移除了最后一个元素，该结构会自动删除，内存被回收。

#### 存储用户信息

和 string 相比 优点是不用序列化和反序列化， 缺点是因为一个 hash 下有多个 key 表示对象的属性 比 string 会多占用很多空间

## Set: 集合

set 集合 和 java 数据结构 Set 很相似， 都维护着一组不重复的集合 ， 如果已有数据添加, 则添加失败。

### 向 set 集合中添加元素

sadd [key] [value]

### 获取集合中元素个数

scard [key]

### 比较两个集合有哪些不同

sdiff [key] [key1]

### 将两个集合的不同插入到新的集合中(差集)

sdiffstore [newkey] [key] [key1]

### 获取交集

sinter [key] [key2] ...

### 获取交集插入到新的集合中

sinterstore [newkey] [key] [key2] ...

### 判断元素是否存在集合中

sismember [key] [member]

### 获取集合中所有元素

smembers [key]

### 将元素从一个集合移动到另一个集合中

smove [source] [dest] [member]

### 移除并随机返回一个元素

spop [key]

### 返回集合中一个或多个随机数

srandmember [key] [count]

### 移除集合中一个或多个成员

srem [key] [member1] [member2] ...

### 返回集合中并集

sunion [key1] [key2]

### 返回并集插入到新的集合中

sunionstore [dest] [key1] [key2] ...

### 从[key]集合中第[cursor]开始模糊匹配[pattern]搜索 count 个元素

sscan [key] [cursor] match [pattern] [count]

## Sorted Set: 有序集合

Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储 40 多亿个成员)。

redis 中 sort set 为了提高性能，支持随即插入和删除，不宜使用简单的数组结构 ，因为自带排序，每次插入的时候要重新找到插入点（二分法查找，数组比较适合，但是不适合链表）所以选用的基于跳跃链表的结构组成的，关于跳跃链表 你可以理解成

集合中 挑选出一些数据（这些数据有一定的间隔，举个例子 如数组 0 作为第一个 数组 4 作为第二个 数组 8 作为第三个） 作为 Level0 层 （最上层），

### 向集合内添加一个或者多个数据

zadd [key] [score1] [member1] [score2] [member2] ...

### 获取成员数量

zcard [key]

### 计算区间内的成员数量

zcount [key] [min] [max]

### 有序集合中对指定成员的分数加上增量 increment

zincrby [key] [increment] [member]

### 求交集插入到新的集合中

zinterstore [destination] [numkeys] key [key ...]

### 求区间范围内的元素数量

zlexcount [key] [min] [max]

### 通过索引返回成员

zrange [key] [start] [stop] [WITHSCORES]

```
redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES             # 显示整个有序集成员
1) "jack"
2) "3500"
3) "tom"
4) "5000"
5) "boss"
6) "10086"

```

### 通过字典区间返回有序集合的成员。

zrangebylex [key] [min] [max] [LIMIT offset count]

```
redis 127.0.0.1:6379> ZADD myzset 0 a 0 b 0 c 0 d 0 e 0 f 0 g
(integer) 7
redis 127.0.0.1:6379> ZRANGEBYLEX myzset - [c
1) "a"
2) "b"
3) "c"
redis 127.0.0.1:6379> ZRANGEBYLEX myzset - (c
1) "a"
2) "b"
redis 127.0.0.1:6379> ZRANGEBYLEX myzset [aaa (g
1) "b"
2) "c"
3) "d"
4) "e"
5) "f"
```

### 通过分数返回有序集合指定区间内的成员

ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

```
redis 127.0.0.1:6379> ZADD salary 2500 jack                        # 测试数据
(integer) 0
redis 127.0.0.1:6379> ZADD salary 5000 tom
(integer) 0
redis 127.0.0.1:6379> ZADD salary 12000 peter
(integer) 0

redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf               # 显示整个有序集
1) "jack"
2) "tom"
3) "peter"

redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf +inf WITHSCORES    # 显示整个有序集及成员的 score 值
1) "jack"
2) "2500"
3) "tom"
4) "5000"
5) "peter"
6) "12000"

redis 127.0.0.1:6379> ZRANGEBYSCORE salary -inf 5000 WITHSCORES    # 显示工资 <=5000 的所有成员
1) "jack"
2) "2500"
3) "tom"
4) "5000"

redis 127.0.0.1:6379> ZRANGEBYSCORE salary (5000 400000            # 显示工资大于 5000 小于等于 400000 的成员
1) "peter"
```

### 返回有序集合中指定成员的索引

zrank [key] [member]

### 删除元素

zrem [key] [member] [member ...]

### 移除有序集合中给定的字典区间的所有成员

zremrangebylex [key] [min] [max]

### 移除有序集合中给定的排名区间的所有成员

zremrangebyrank [key] [start] [stop]

### 移除有序集合中给定的分数区间的所有成员

zremrangebyscore key min max

### 返回有序集中指定区间内的成员，通过索引，分数从高到低

zrevrange [key] [start] [stop] [WITHSCORES]

### 返回有序集中指定分数区间内的成员，分数从高到低排序

zrevrangebyscore [key] [max] [min] [WITHSCORES]

### 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序

zrevrank key member

### 返回有序集中，成员的分数值

zscore [key] [member]

### 计算给定的一个或多个有序集的并集，并存储在新的 key 中

zunionstore destination numkeys key [key ...]

### 迭代有序集合中的元素（包括元素成员和元素分值）

zscan key cursor [MATCH pattern] [COUNT count]

# Redis 其他类型

## HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。
在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

hyperloglog 有三个 api

### 添加计算集合

pfadd [key] [value]
这里的 key 是你要统计的目标， value 是 一种标识位，举个例子我要统计 一个 api 今天有多少次调用， key 就可以设置成 当前 api 的路径， value 可以设置成 1 或者用户请求的 ip 地址

### 统计基数

pfcount [key] [key1]
单个 key 是统计 当前 key 的总数和， 多个 key 则选出所有 key 中最大的总数和
举个例子 pfadd a 1 2 3 使用 pfcount a 结果为 3
pfadd b 4 5 6 7 使用 pfcount a b 结果为 4

### 合并基数

PFMERGE destkey sourcekey [sourcekey ...]
pfmerge [最终合并 key] [原始 key]

### 释放 key

del [key] .通用

## GEO 地理位置计算

Redis GEO 主要用于存储地理位置信息，并对存储的信息进行操作，该功能在 Redis 3.2 版本新增。

Redis GEO 操作方法有：

geoadd：添加地理位置的坐标。
geopos：获取地理位置的坐标。
geodist：计算两个位置之间的距离。
georadius：根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
georadiusbymember：根据储存在位置集合里面的某个地点获取指定范围内的地理位置集合。
geohash：返回一个或多个位置对象的 geohash 值。

# 其他应用

## 分布式锁

可以使用 setnx 判断已设置（加锁） ， 然后配合 del key 解锁。但是这种方式会有问题， 如果调用中出现了异常 导致程序走不到 delkey 锁就一直存在变成了死锁。
当然可能会有考虑设置过期默认时间 ，比如 setnx lock ture . expire lock 5 ,但是 其实还有问题。 因为 setnx 和 expire 是两个操作，不是原子指令。 可能需要配合其他第三方库来使用。

### 分布式锁 不能解决超时问题

当设置了过期时间,但是处理业务函数时间过长。就会导致 程序未执行完，但是锁已经释放掉了。

### 推荐使用 redission

其实单独使用 redis 或多或少都会有一些问题。 这里推荐使用 Redission 官网： https://redisson.org/。， 中文地址：https://github.com/redisson/redisson/wiki/目录

## 延时队列

利用 redis list 中 阻塞队列实现

## 位图 Bitmap

我们都知道 8bit = 1b = kb，bitmap 就是通过最小的单位 bit 来进行 0 或者 1 的设置，表示某个元素对应的值或者状态。
一个 bit 的值，或者是 0，或者是 1；也就是说一个 bit 能存储的最多信息是 2。

位图并不是一种特殊的数据结构，其实本质上是二进制字符串，也可以看做是 byte 数组。可以使用普通的 get/set 直接获取和设置整个位图的内容，也可以使用位图操作 getbit/setbit 等将 byte 数组看成「位数组」来处理。

### 位图的优势:

基于最小的单位 bit 进行存储，所以非常省空间。
设置时候时间复杂度 O(1)、读取时候时间复杂度 O(n)，操作是非常快的
二进制数据的存储，进行相关计算的时候非常快
方便扩容
一般可以在如下场景使用：
用户签到
用户在线状态
统计活跃用户
各种状态值
常用命令
SETBIT
对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
SETBIT key offset value
offset 参数必须大于或等于 0 ，小于 2^32 (bit 映射被限制在 512 MB 之内)。

### GETBIT

对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
GETBIT key offset

### BITCOUNT

计算给定字符串中，被设置为 1 的比特位的数量。
BITCOUNT key

### BITPOS

返回位图中第一个值为 bit 的二进制位的位置。
BITPOS key bit [start] [end]

### BITOP

对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey 上。
BITOP operation destkey key [key …]
operation 可以是 AND 、 OR 、 NOT 、 XOR 这四种操作中的任意一种
BITOP AND destkey key [key ...] ，对一个或多个 key 求逻辑并，并将结果保存到 destkey 。

### BITFIELD

bitfield 有三个子指令，分别是 get/set/incrby，它们都可以对指定位片段进行读写，但是最多只能处理 64 个连续的位，如果超过 64 位，就得使用多个子指令，bitfield 可以一次执行多个子指令。

#### 适用于各类统计应用

记录用户的签到，每日在线情况等，可以将当天或者当天的偏移量对应的 bit 位设置为 1 即可，使用 BITCOUNT 可以轻松统计签到次数。

还有一种使用比较多的情况，就是设置各类状态值，例如商城的设置：是否可以评价订单，是否展示售罄商品，是否正常营业等状态值可以使用 bitmap 来存储

在性能方面，如前面提到的签到，即使运行 10 年，占用的空间也只是每个用户 10\*365 比特位(bit)，也即是每个用户 456 字节。对于这种大小的数据来说， BITCOUNT key [start] [end] 的处理速度就像 GET key 和 INCR key 这种 O(1) 复杂度的操作一样快。

当然如果你的 bitmap 数据非常大，那么可以考虑使用以下两种方法：

将一个大的 bitmap 分散到不同的 key 中，作为小的 bitmap 来处理。使用 Lua 脚本可以很方便地完成这一工作。
使用 BITCOUNT key [start] [end] 的 start 和 end 参数，每次只对所需的部分位进行计算，然后在进行累加。

## 搜索

### keys

在平时线上 Redis 维护中，有时候需要从成千上万个 key 中找到 特定前缀的 key 来手动处理数据。Redis 提供了简单粗暴的命令 keys 用来列出所有满足正则条件的 key

```
keys *
keys code*
keys code*hole

```

这个指令很简单，但是有两个明显缺点

1. 没有 offset、 limit 参数 ，一次性筛选出所有满足条件的 key,如果实例中有几百万个 key 满足条件，那你将看到满屏的字符串。
2. keys 算法时遍历算法 复杂度为 O(n),如果实例中有千万级 key ，这个指令就会导致 redis 服务卡顿，所有读写 redis 的其他指令都会被延后甚至会超市报错（单线程）

### scan

1.复杂度虽然也是 O(n),但是它是通过游标分步进行的，不会阻塞线程。 2.提供 limit 参数，可以控制每次返回结果的最大条数， limit 只是一个 hint,返回的结果可多可少。 3.同 keys 一样，支持模式匹配。 4.服务器不需要为游标保存至黄台， 游标的唯一状态就是 scan 返回客户端的游标整数。 5.返回结果可能重复，需要客户端去重。 6.遍历的过程如果有数据修改，改动后的数据能不能遍历到是不确定的。 7.单词返回的结果是空的并不意味着遍历结束，而要看游标是否为 0。

#### scan 基本用法

scan [cursor] match [pattern] count [limmit ]

```
> scan 0 match key99* count 1000
1) "13976"
2)  1) "key9911"
    2) "key9912"
    3) "key9913"
    ....
> scan 13976 match key99* count 1000
1) "1996"
2)  1) "key9911"
    2) "key9912"
    3) "key9913"
    ....
```

我们也可以用 scan 查询 hash, set
hscan , zscan 用法同上

#### 在平时的业务开发中，要尽量避免大 key 的产生

## 布隆过滤器

布隆过滤器
什么是布隆过滤器
布隆过滤器（Bloom Filter）是 1970 年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。
布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。

实际工程的应用

实际上，布隆过滤器广泛应用于网页黑名单系统、垃圾邮件过滤系统、爬虫网址判重系统等，有人会想，我直接将网页 URL 存入数据库进行查找不就好了，或者建立一个哈希表进行查找不就 OK 了。

当数据量小的时候，这么思考是对的，但如果整个网页黑名单系统包含 100 亿个网页 URL，在数据库查找是很费时的，并且如果每个 URL 空间为 64B，那么需要内存为 640GB，一般的服务器很难达到这个需求。

那么，在这种内存不够且检索速度慢的情况下，不妨考虑下布隆过滤器，但业务上要可以忍受判断失误率。

位图（bitmap）

布隆过滤器其中重要的实现就是位图的实现，也就是位数组，并且在这个数组中每一个位置只占有 1 个 bit，而每个 bit 只有 0 和 1 两种状态。如上图 bitarray 所示！bitarray 也叫 bitmap，大小也就是布隆过滤器的大小。

假设一种有 k 个哈希函数，且每个哈希函数的输出范围都大于 m，接着将输出值对 k 取余（%m）,就会得到 k 个[0, m-1]的值，由于每个哈希函数之间相互独立，因此这 k 个数也相互独立，最后将这 k 个数对应到 bitarray 上并标记为 1（涂黑）。

等判断时，将输入对象经过这 k 个哈希函数计算得到 k 个值，然后判断对应 bitarray 的 k 个位置是否都为 1（是否标黑），如果有一个不为黑，那么这个输入对象则不在这个集合中，也就不是黑名单了！如果都是黑，那说明在集合中，但有可能会误，由于当输入对象过多，而集合也就是 bitarray 过小，则会出现大部分为黑的情况，那样就容易发生误判！因此使用布隆过滤器是需要容忍错误率的，即使很低很低！

布隆过滤器重要参数计算
通过上面的描述，我们可以知道，如果输入量过大，而 bitarray 空间的大小又很小，那么误判率就会上升。那么 bitarray 空间大小怎么确定呢？不要慌，已经有人通过数据推倒出公式了！！！哈哈，直接用～

假设输入对象个数为 n，bitarray 大小（也就是布隆过滤器大小）为 m，所容忍的误判率 p 和哈希函数的个数 k。计算公式如下：（小数向上取整）

注意：由于我们计算的 m 和 k 可能是小数，那么需要经过向上取整，此时需要重新计算误判率 p！

假设一个网页黑名单有 URL 为 100 亿，每个样本为 64B，失误率为 0.01%，经过上述公式计算后，需要布隆过滤器大小为 25GB，这远远小于使用哈希表的 640GB 的空间。

并且由于是通过 hash 进行查找的，所以基本都可以在 O(1)的时间完成！

也可以使用 guava 中的布隆过滤器

# 原理

## 线程模型

redis 是单线程程序，使用的是多路复用 io 模型。

### 非阻塞 io

当我们调用套接字的读写方法，默认它是阻塞的，比如 read 方法要传递进去一个 n ,表示最多读取那个字节然后再返回，如果一个字节都没有线程就会卡在那里，直到有新的数据到来或者连接关闭， read 方法才可以返回 ，线程才能继续处理。
write 方法一般不会则色，除非内核为套接字分配的写缓冲区已经满了，write 方法就会阻塞，直到缓冲区有空间空闲出来。
而非阻塞 io 是在套接字上提供一个选项 non_blocking ,当选项打开是 读写方法不会阻塞， 而是有多少读多少，能写多少写多少。（这里的多少却决于内核为套接字分配读写缓冲区的字节数），读写方法都会通过返回值告诉程序实际读写了多少字节。

### 事件轮询

非阻塞 io 有个问题，就是读数据，读了一部分就返回了。写也似乎一样，如果缓冲区满了，写不完，剩下的何时才应该继续写，线程应该得到通知。

事件轮询 api 就是用来解决这个问题的。最简单那的事件轮询 api 是 select 函数，它是操作系统提供给用户程序的 api，输入时读写描述符列表 read_fds & write_fds,输出时与之对应的可读可写事件。同时还提供了 timeout 参数，如果没有任何事件到来，最多等待 timeout 事件后线程进入阻塞状态。
一旦有任何事件到来就可以立即返回。 拿到事件后线程可以继续处理相应事件，处理完毕后 会重新轮询。

现代操作系统的多路复用 api 不再使用 select ,而是改用 epoll (linux) 和 kqueue (freebsd 和 macosx), 因为 select 系统调用在描述符特别多的时候性能会变得非常差。

## 通讯协议

redis 使用的是 resp 协议,直观的文本型协议。redis 作者认为性能的瓶颈不在于网络流量，而是取决于数据库的内部逻辑实现（协议占的比重不是那么多）
一共有 5 中最小单元类型， 单元结束后都会加上/r/n 换行符

1. 单字符串以“+” 符号开头。
2. 多行字符串以“\$” 符号开头，后面跟字符串长度。
3. 整数值以“:”开头，后跟整数的字符串形式。
4. 错误消息以“-”开头。
5. 数组以“\*”号开头，后跟数组的长度。
   NUll 和空串 都以“\$” 开头， null 后面长度为-1 ，空串为 0.

### 单行字符串 helloworld

+helloworld \r\n

### 多行字符串

\$11\r\n hello world \r\n

### 整数

```
:1024\r\n
```

### 错误

```
-WRONGTYPE Operation against akey holding the wrong kind of value \r\n
```

### 数组

```
*3\r\n:1\r\n:2\r\n:3\r\n
```

## 持久化

redis 持久化主要分两种 1.rdb 2.aof,其中 rdb 表示的是快照， 而 aof 表示用户客户端所有命令产生的日志集合

1.RDB : 在指定时间间隔能对你的数据进行快照存储。
2.AOF : 记录每次对服务器写的操作，当服务器重启时会重新执行这些命令来恢复数据。

### 持久化配置

#### RDB 的持久化配置

```
# 时间策略
# 表示900s内如果有1条是写入命令，就触发产生一次快照，可以理解为就进行一次备份
save 900 1
# 表示300s内有10条写入，就产生快照
save 300 10
# 表示60秒内有10000个更改，产生快照
save 60 10000

# 文件名称
dbfilename dump.rdb

# 文件保存路径
dir /home/work/app/redis/data/

# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes

# 是否压缩
rdbcompression yes

# 导入时是否检查
rdbchecksum yes

```

stop-writes-on-bgsave-error yes 这个配置也是非常重要的一项配置，这是当备份进程出错时，主进程就停止接受新的写入操作，是为了保护持久化的数据一致性问题。如果自己的业务有完善的监控系统，可以禁止此项配置， 否则请开启。

关于压缩的配置 rdbcompression yes ，建议没有必要开启，毕竟 Redis 本身就属于 CPU 密集型服务器，再开启压缩会带来更多的 CPU 消耗，相比硬盘成本，CPU 更值钱。

当然如果你想要禁用 RDB 配置，也是非常容易的，只需要在 save 的最后一行写上：save ""

#### AOF 配置

```
# 是否开启aof
appendonly yes

# 文件名称
appendfilename "appendonly.aof"

# 同步方式
appendfsync everysec

# aof重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof时如果有错如何处理
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes

```

#### 还是重点解释一些关键的配置：

```
appendfsync everysec 它其实有三种模式:

always：把每个写命令都立即同步到aof，很慢，但是很安全
everysec：每秒同步一次，是折中方案
no：redis不处理交给OS来处理，非常快，但是也最不安全

一般情况下都采用 everysec 配置，这样可以兼顾速度与安全，最多损失1s的数据。
aof-load-truncated yes 如果该配置启用，在加载时发现aof尾部不正确是，会向客户端写入一个log，但是会继续执行，如果设置为 no ，发现错误就会停止，必须修复后才能重新加载。
```

#### 工作原理

工作原理
关于原理部分，我们主要来看 RDB 与 AOF 是如何完成持久化的，他们的过程是如何。
在介绍原理之前先说下 Redis 内部的定时任务机制，定时任务执行的频率可以在配置文件中通过 hz 10 来设置（这个配置表示 1s 内执行 10 次，也就是每 100ms 触发一次定时任务）。
该值最大能够设置为：500，但是不建议超过：100，因为值越大说明执行频率越频繁越高，这会带来 CPU 的更多消耗，从而影响主进程读写性能。
定时任务使用的是 Redis 自己实现的 TimeEvent，它会定时去调用一些命令完成定时任务，这些任务可能会阻塞主进程导致 Redis 性能下降。
因此我们在配置 Redis 时，一定要整体考虑一些会触发定时任务的配置，根据实际情况进行调整。

RDB 原理
在 Redis 中 RDB 持久化的触发分为两种：自己手动触发与 Redis 定时触发。
针对 RDB 方式的持久化，手动触发可以使用：

save：会阻塞当前 Redis 服务器，直到持久化完成，线上应该禁止使用。
bgsave：该触发方式会 fork 一个子进程，由子进程负责持久化过程，因此阻塞只会发生在 fork 子进程的时候。

而自动触发的场景主要是有以下几点：

根据我们的 save m n 配置规则自动触发；
从节点全量复制时，主节点发送 rdb 文件给从节点完成复制操作，主节点会触发 bgsave；
执行 debug reload 时；
执行 shutdown 时，如果没有开启 aof，也会触发。

由于 save 基本不会被使用到，我们重点看看 bgsave 这个命令是如何完成 RDB 的持久化的。

rdb 使用操作系统的多进程 cow(copy on write)机制来实现快照持久化。

### COW(Copy On Write) 机制

COW(Copy On Write) 机制属于操作系统处理多进程下的一种机制，Redis 在持久化的时候会调用 glibc 函数 fork 一个子进程。父子进程会共享内存里面的代码段和数据段。

所以持久化的时候是完全交给子进程，而父进程继续处理客户端请求，所以在持久化的时候操作系统采用 COW 机制进程数据段页面的分离。数据段是由很多操作系统的页面组合而成，当父进程对其中一个页面进行数据修改的时候，先将被父子线程共享的这一个页面复制并分离出来，然后直接对复制的页面进程修改，而此时子进程对应的页面是没有修改的。

Redis 采用该机制的简单流程如下。Linux 在 fork 之后，操作系统会将父进程的所有内存也权限设置为 read-only，然后子进程的地址空间指向父进程。当父进程只读时没有问题，当有写内存时，CPU 硬件检测到内存也是 read-only，于是会触发页异常中断（page-fault），陷入到操作系统的一个中断例程。
中断例程中，操作系统采用 cow 机制会触发异常的也复制一份，于是父子进程各自持有独立的一份，如果这个时候又大量写入操作，会产生大量的分页错误（页异常中断 page-fault），从而触发 cow 机制。
之所以称之为快照也就是说在子进程创建的那一时刻开始。内存的数据就固定下来了，不会发生变化。

redis 在持久化的时候会调用 glibc 的 fork 函数产生一个子进程，快照持久化交给子进程处理，父进程处理客户端请求。
子进程刚产生的时候和父进程共享内存代码段和数据段，子进程做数据持久化，不会修改现有的内存数据结构， 它只是对数据结构进行遍历读取，然后持久化到磁盘中。但是父进程不一样，他必须持续服务客户端请求，然后对内存数据进行不断的修改。
这个时候会使用操作系统的 cow 机制进行数据段页面的分离。

RDB 的优缺点
优点：
性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，保证了 redis 的高性能
重启恢复数据的时候。数据量比较大时候，Redis 直接解析 RDB 二进制文件，生成对应的数据存储在内存中，比 AOF 的启动效率更高
缺点
数据安全性低，因为是间隔一段时间进行持久化，如果在持久化之间发生了故障，会丢失数据，这也就决定了该方式更适合在数据要求不严谨的时候采用
系统性能耗费，根据上文提到的 Redis 执行 cow 机制时，可以看到大量的分页错误会耗费不少性能在复制上。

### AOF（Append Only File - 仅追加文件）

根据上文，快照在某些情况下不是可行的选择，所以 AOF 很好的支持了。

#### AOF 原理

该方式非常简单：也就是修改内存的操作命令都会记录下来，加入 AOF 日志记录都是 Redis 实例创建以来的所有修改性指令序列，所以恢复也就是顺序执行所有执行。

具体步骤
命令写入-> [append] ->AOF 缓冲 ->同步策略 ->[sync] ->AOF 文件->重写策略

Redis 使用单线程相应命令，如果每次写 AOF 文件命令都追加到硬盘，会极大地影响处理性能，所以 Redis 会先写入到 aof 缓冲区，根据用户配置的同步硬盘策略写入到 aof 文件中，这个策略可以通过 appendfsync 参数配置，

1. always：每一次写操作都会调用一次 fsync，这时数据是最安全的，当然，由于每次都会执行 fsync，所以其性能也会受到影响
2. no：Redis 不会主动调用 fsync 去将 AOF 日志内容同步到磁盘，所以这一切就完全依赖于操作系统的调试了。对大多数 Linux 操作系统，是每 30 秒进行一次 fsync，将缓冲区中的数据写到磁盘上。
3. everysec：Redis 会默认每隔一秒进行一次 fsync 调用，将缓冲区中的数据写到磁盘。但是当这一次的 fsync 调用时长超过 1 秒时。Redis 会采取延迟 fsync 的策略，再等一秒钟。也就是在两秒后再进行 fsync，这一次的 fsync 就不管会执行多长时间都会进行。这时候由于在 fsync 时文件描述符会被阻塞，所以当前的写操作就会阻塞。
   注意，这也是影响 Redis 性能的参数之一，建议采用 appendfsync everysec（缺省方式）

#### AOF 重写

所谓重写，Redis 在长期运行过程中日志会越来越大，在恢复的时候会非常好使，所以我们的目的就是对日志做瘦身

会从以下几点做瘦身：

无效命令可以删除，比如 del key1、hdel key2、srem keys、set a111、set a222 等，直接用最终的数据生成命令保存下来就行
多条命令可以删除，如：lpush list a、lpush list b、lpush list c 可以转化为：lpush list a b c
等等，就不列举了
Redis 使用 bgrewriteaof 指令做瘦身，主要也是开辟一个子进程对内存遍历转化为一系列指令，并序列化到新的文件中，接下来再将操作期间的增量 AOF 日志追加到新的日志文件中，最终替换了旧的。

AOF 重写机制两种方式触发

手动触发：bgrewriteaof 指令
自动触发：根据 auto-aof-rewrite-min-size 和 auto-aof-rewrite-percentage 参数确定自动触发时机
auto-aof-rewrite-min-size：表示运行 AOF 重写时文件最小体积，默认为 64MB。
auto-aof-rewrite-percentage：代表当前 AOF 文件空间 （aof_current_size）和上一次重写后 AOF 文件空间（aof_base_size）的比值。
auto-aof-rewrite-min-size 100auto-aof-rewrite-percentage 64mb ```
如上代表 AOF 文件的大小小于 64mb(默认值)，且当前 AOF 文件大小比基准大小增长了 100%时会触发。

#### AOF 优缺点

优点
数据安全，aof 持久化配置 appendfsync 属性，有 always，每执行一次命令操作就记录到 aof 文件一次
缺点
数据集大的时候，比如 RDB 启动效率低

#### 混合持久化（Redis 4.0 版本）

我们根据上文知道，RDB 恢复会存在大量数据，AOF 恢复性能又较慢，所以在 Redis4.0 中，采用混合持久化，将 RDB 文件内存和增量的 AOF 日志文件放在一起，这里的 AOF 日志不再是全量日志。而是自持久化开始到持久化结束的这段时间的增量日志，通常较小，重启效率因此大幅得到提升
加载的时候，首先会识别 AOF 文件是否以 REDIS 字符串开头，如果是就按照 RDB 格式加载，加载完成后继续按 AOF 加载剩余的部分

## 管道

Redis 是一种基于客户端-服务端模型以及请求/响应协议的 TCP 服务。这意味着通常情况下一个请求会遵循以下步骤：
客户端向服务端发送一个查询请求，并监听 Socket 返回，通常是以阻塞模式，等待服务端响应。
服务端处理命令，并将结果返回给客户端。

read 和 write 操作只是缓冲区里读数据 而不走网络， 网络发送和接收都交给内核 异步进行。

### Redis 管道技术

Redis 管道技术可以在服务端未响应时，客户端可以继续向服务端发送请求，并最终一次性读取所有服务端的响应。

### 管道技术的优势

管道技术最显著的优势是提高了 redis 服务的性能。

## 事务

redis 事务比较简单，事务模型不是很严格，所以不能像普通数据库那样使用事务。

主要指令有 multi、exec、discard
其中 multi 表示事务开始，exec 指事务的执行，discard 表示丢弃事务

服务器在执行 exec 之前只是把命令插入到事务队列中，一旦接受 exec 指令才开始整个事务。

redis 不保证原子性(出错时（比如 incr string），后面的命令依然执行)，它只保证了命令串行化（不被其他事务打断）

```
测试命令如下
> multi
OK
> set str a
QUEUED
> incr str
QUEUED
> set str2 a
QUEUED
> exec

# 结果为
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
```

可以看出，出现了错误 并没有中断

使用事务时 每一个指令 都会加入到事务缓存队列中需要经过一次网络读写 ，当事务指令过多时，网络 IO 时间会变得很长，所以通常需要配合管道使用。

#### watch

redis 我们已经知道它是单线程基于事件轮询方式处理的，所以一般来讲所有操作都不会产生并发问题。
但是如果我们想修改里面的值 比如把某个人的用户余额进行修改。（不是简单的加减，翻个倍） 那么我们就需要把 redis 里的余额部分取出来，然后以代码的形式修改后 再重新写入回去。如果是多个客户端并发执行那这部分会产生问题。
我们可以使用分布式锁，它是一种悲观锁。我们也可以使用 watch（乐观锁） 来进行监听。
在执行 multi 之前先 watch 需要修改的变量。在执行 exec 时判断 当前 watch 的变量是否有被人改动过， 如果有 则直接返回空。 我们可以通过返回值来判断是否已经被人修改。

## 订阅

### redis 订阅模式 不推荐 （消息易丢失）

PubSub 的生产者传递过来一个消息，redis 会直接找到想赢的消费者传递过去，如果没有消费者，消息会被直接丢弃。
如果开始有三个消费者，一个消费者突然挂掉了，生产这会继续发送消息，另外两个消费者可以持续收到消息，但是当挂掉的消费者重新连上的时候，在断连期间生产者发送的消息，对于这个消费者来说就是丢失的。
如果 redis 停机重启， 消息不会持久化。

# 集群

### 一、主从同步/复制

通过持久化功能，Redis 保证了即使在服务器重启的情况下也不会丢失（或少量丢失）数据，因为持久化会把内存中数据保存到硬盘上，重启会从硬盘上加载数据。 但是由于数据是存储在一台服务器上的，如果这台服务器出现硬盘故障等问题，也会导致数据丢失。

为了避免单点故障，通常的做法是将数据库复制多个副本以部署在不同的服务器上，这样即使有一台服务器出现故障，其他服务器依然可以继续提供服务。为此， Redis 提供了复制（replication）功能，可以实现当一台数据库中的数据更新后，自动将更新的数据同步到其他数据库上。

在复制的概念中，数据库分为两类，一类是主数据库（master），另一类是从数据库(slave）。主数据库可以进行读写操作，当写操作导致数据变化时会自动将数据同步给从数据库。而从数据库一般是只读的，并接受主数据库同步过来的数据。一个主数据库可以拥有多个从数据库，而一个从数据库只能拥有一个主数据库。

#### 主从数据库的配置：

主数据库不用配置，从数据库的配置文件（redis.conf）中可以加载主数据库的信息，也可以在启动时，使用 redis-server --port 6380 --slaveof 127.0.0.1 6379 命令指明主数据库的 IP 和端口。从数据库一般是只读，可以改为可写，但写入的数据很容易被主同步没，所以还是只读就可以。也可以在运行时使用 slaveof ip port 命令，停止原来的主，切换成刚刚设置的主 slaveof no one 会把自己变成主。

#### 主从复制原理：

从数据库连接主数据库，发送 SYNC 命令；
主数据库接收到 SYNC 命令后，开始执行 BGSAVE 命令生成 RDB 文件并使用缓冲区记录此后执行的所有写命令；
主数据库 BGSAVE 执行完后，向所有从数据库发送快照文件，并在发送期间继续记录被执行的写命令；
从数据库收到快照文件后丢弃所有旧数据，载入收到的快照；
主数据库快照发送完毕后开始向从数据库发送缓冲区中的写命令；
从数据库完成对快照的载入，开始接收命令请求，并执行来自主数据库缓冲区的写命令；（从数据库初始化完成）
主数据库每执行一个写命令就会向从数据库发送相同的写命令，从数据库接收并执行收到的写命令（从数据库初始化完成后的操作）
出现断开重连后，2.8 之后的版本会将断线期间的命令传给重数据库，增量复制。
主从刚刚连接的时候，进行全量同步；全同步结束后，进行增量同步。当然，如果有需要，slave 在任何时候都可以发起全量同步。Redis 的策略是，无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步。

```
优点：

支持主从复制，主机会自动将数据同步到从机，可以进行读写分离；
为了分载 Master 的读操作压力，Slave 服务器可以为客户端提供只读操作的服务，写服务仍然必须由 Master 来完成；
Slave 同样可以接受其它 Slaves 的连接和同步请求，这样可以有效的分载 Master 的同步压力；
Master Server 是以非阻塞的方式为 Slaves 提供服务。所以在 Master-Slave 同步期间，客户端仍然可以提交查询或修改请求；
Slave Server 同样是以非阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis 则返回同步之前的数据；
#缺点：

Redis 不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复；
主机宕机，宕机前有部分数据未能及时同步到从机，切换 IP 后还会引入数据不一致的问题，降低了系统的可用性；
如果多个 Slave 断线了，需要重启的时候，尽量不要在同一时间段进行重启。因为只要 Slave 启动，就会发送 sync 请求和主机全量同步，当多个 Slave 重启的时候，可能会导致 Master IO 剧增从而宕机。
Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂；
```

### 二、哨兵模式

第一种主从同步/复制的模式，当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。

哨兵模式是一种特殊的模式，首先 Redis 提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。

哨兵模式的作用：

通过发送命令，让 Redis 服务器返回监控其运行状态，包括主服务器和从服务器；
当哨兵监测到 master 宕机，会自动将 slave 切换成 master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机；
然而一个哨兵进程对 Redis 服务器进行监控，也可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

故障切换的过程：

假设主服务器宕机，哨兵 1 先检测到这个结果，系统并不会马上进行 failover 过程，仅仅是哨兵 1 主观的认为主服务器不可用，这个现象成为主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行 failover 操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为客观下线。这样对于客户端而言，一切都是透明的。

哨兵模式的配置：

配置一主二从和三个哨兵的 Redis 服务器来演示这个过程

主从服务器配置

```
# 使得Redis服务器可以跨网络访问
bind 0.0.0.0

# 设置密码
requirepass "123456"

# 以下有关slaveof的配置只是配置从服务器，主服务器不需要配置
# 指定主服务器
slaveof 192.168.11.128 6379
# 主服务器密码
masterauth 123456
```

哨兵配置

```
# 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
sentinel monitor mymaster 192.168.11.128 6379 2
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
```

配置 3 个哨兵，每个哨兵的配置都是一样的。在 Redis 安装目录下有一个 sentinel.conf 文件，copy 一份进行修改

启动

注意启动的顺序。首先是主机（192.168.11.128）的 Redis 服务进程，然后启动从机的 Redis 服务进程，最后启动 3 个哨兵的服务进程。

哨兵模式的工作方式：

每个 Sentinel（哨兵）进程以每秒钟一次的频率向整个集群中的 Master 主服务器，Slave 从服务器以及其他 Sentinel（哨兵）进程发送一个 PING 命令。
如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为主观下线（SDOWN）
如果一个 Master 主服务器被标记为主观下线（SDOWN），则正在监视这个 Master 主服务器的所有 Sentinel（哨兵）进程要以每秒一次的频率确认 Master 主服务器的确进入了主观下线状态
当有足够数量的 Sentinel（哨兵）进程（大于等于配置文件指定的值）在指定的时间范围内确认 Master 主服务器进入了主观下线状态（SDOWN）， 则 Master 主服务器会被标记为客观下线（ODOWN）
在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有 Master 主服务器、Slave 从服务器发送 INFO 命令。
当 Master 主服务器被 Sentinel（哨兵）进程标记为客观下线（ODOWN）时，Sentinel（哨兵）进程向下线的 Master 主服务器的所有 Slave 从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
若没有足够数量的 Sentinel（哨兵）进程同意 Master 主服务器下线， Master 主服务器的客观下线状态就会被移除。若 Master 主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master 主服务器的主观下线状态就会被移除。
优点：

哨兵模式是基于主从模式的，所有主从的优点，哨兵模式都具有。
主从可以自动切换，系统更健壮，可用性更高。
缺点：

Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。

### 三、Cluster 集群

Redis 的哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，所以在 redis3.0 上加入了 Cluster 集群模式，实现了 Redis 的分布式存储，也就是说每台 Redis 节点上存储不同的内容。

集群的配置

根据官方推荐，集群部署至少要 3 台以上的 master 节点，最好使用 3 主 3 从六个节点的模式。在测试环境中，只能在一台机器上面开启 6 个服务实例来模拟。

1、修改配置文件

将 redis.conf 的配置文件复制 6 份（文件名最好加上端口后缀），然后开始修改配置文件中的参数

```
#开启redis的集群模式
cluster-enabled yes

#配置集群模式下的配置文件
cluster-config-file nodes-6379.conf

#集群内节点之间支持最长响应时间
cluster-node-timeout 15000
```

2、修改完毕之后启动 6 个 Redis 服务

3、快速部署集群

6 个 Redis 服务启动成功之后，借助 redis-tri.rb 工具可以快速的部署集群，如果本机没有该命令行需要自行安装，只需要执行/redis-trib.rb create --replicas 1 127.0.0.1:6380 127.0.0.1:6381 127.0.0.1:6382 127.0.0.1:6383 127.0.0.1:6384 127.0.0.1:6385 就可以成功创建集群。

创建集群可能会出现的错误

#这是由于创建集群中的某一个服务中曾经插入过数据，并且已经产生了持久化文件，此时需要 flushall 命令清空所有数据
[ERR] Node 127.0.0.1:6380 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0

#这是由于之前创建集群遗留的配置文件导致的问题，使用命令 cluster reset 即可
redis-4.1.0/lib/redis/client.rb:124:in `call': ERR Slot 935 is already busy
集群的部署会在后续的文章中进行详细的说明和测试，这里就不详细说明了

集群的特点

所有的 redis 节点彼此互联(PING-PONG 机制),内部使用二进制协议优化传输速度和带宽。

节点的 fail 是通过集群中超过半数的节点检测失效时才生效。

客户端与 Redis 节点直连,不需要中间代理层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。

集群的工作方式

在 Redis 的每一个节点上，都有这么两个东西，一个是插槽（slot），它的的取值范围是：0-16383。还有一个就是 cluster，可以理解为是一个集群管理的插件。当我们的存取的 Key 到达的时候，Redis 会根据 crc16 的算法得出一个结果，然后把结果对 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

为了保证高可用，redis-cluster 集群引入了主从模式，一个主节点对应一个或者多个从节点，当主节点宕机的时候，就会启用从节点。当其它主节点 ping 一个主节点 A 时，如果半数以上的主节点与 A 通信超时，那么认为主节点 A 宕机了。如果主节点 A 和它的从节点 A1 都宕机了，那么该集群就无法再提供服务了。





# Redis 时间复杂度O(n)级别的指令

## String类型
MSET、MSETNX、MGET
## List类型
LPUSH、RPUSH、LRANGE、LINDEX、LSET、LINSERT
LINDEX、LSET、LINSERT 这三个命令谨慎使用

## Hash类型
HDEL、HGETALL、HKEYS/HVALS
HGETALL、HKEYS/HVALS 谨慎使用

## Set类型
SADD、SREM、SRANDMEMBER、SPOP、
SMEMBERS、SUNION/SUNIONSTORE、SINTER/SINTERSTORE、SDIFF/SDIFFSTORE
第二行命令谨慎使用

## Sorted Set类型
ZADD、ZREM、
ZRANGE/ZREVRANGE、ZRANGEBYSCORE/ZREVRANGEBYSCORE、ZREMRANGEBYRANK/ZREMRANGEBYSCORE
第二行时间复杂度 O(log(N)+M)，需要谨慎使用

## 其他常用命令
DEL、KEYS
KEYS 谨慎使用

