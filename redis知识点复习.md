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

## 将元素从一个集合移动到另一个集合中

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

#### 原理

虽然跟 java set 类似， 但是底层逻辑是不一样的。 redis 中 set 为了提高性能，支持随即插入和删除，不宜使用简单的数组结构 ，因为自带排序，每次插入的时候要重新找到插入点（二分法查找，数组比较适合，但是不适合链表）所以选用的基于跳跃链表的结构组成的，关于跳跃链表 你可以理解成

集合中 挑选出一些数据（这些数据有一定的间隔，举个例子 如数组 0 作为第一个 数组 4 作为第二个 数组 8 作为第三个） 作为 Level0 层 （最上层），

## Sorted Set: 有序集合

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

## GEO

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

## 位图 Bitmap

## 简单限流

## 漏斗限流

## scan

## 布隆过滤器

# 原理

## 线程模型

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

Redis 采用该机制的简单流程如下。Lunix 在 fork 之后，操作系统会将父进程的所有内存也权限设置为 read-only，然后子进程的地址空间指向父进程。当父进程只读时没有问题，当有写内存时，CPU 硬件检测到内存也是 read-only，于是会触发页异常中断（page-fault），陷入到操作系统的一个中断例程。
中断例程中，操作系统采用 cow 机制会触发异常的也复制一份，于是父子进程各自持有独立的一份，如果这个时候又大量写入操作，会产生大量的分页错误（页异常中断 page-fault），从而触发 cow 机制。
之所以称之为快照也就是说在子进程创建的那一时刻开始。内存的数据就固定下来了，不会发生变化。

redis 在持久化的时候会调用 glibc 的 fork 函数产生一个子进程，快照持久化交给子进程处理，父进程处理客户端请求。
子进程刚产生的时候和父进程共享内存代码段和数据段，子进程做数据持久化，不会修改现有的内存数据结构， 它只是对数据结构进行遍历读取，然后持久化到磁盘中。但是父进程不一样，他必须持续服务客户端请求，然后对内存数据进行不断的修改。
这个时候会使用操作系统的 cow 机制进行数据段页面的分离。

## 管道

## 事务

## 订阅

## 小对象压缩

# 集群

## 主从

## 哨兵 Sentinel

## Codis

## 集群 Cluster

# 扩展
