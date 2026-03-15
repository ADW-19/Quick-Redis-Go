# 3. Redis数据结构

## 3.1 字符串（Strings）

### 3.1.1 概述

字符串是Redis最基本的数据类型，也是最常用的数据类型。它可以存储任何形式的字符串，包括文本、二进制数据等。字符串的最大长度为512MB。

### 3.1.2 内部实现

Redis的字符串使用SDS（Simple Dynamic String）作为内部实现，SDS是Redis自己实现的一种动态字符串结构，它具有以下特点：

- **动态扩展**：当字符串长度增加时，会自动扩展内存。
- **空间预分配**：当字符串长度增加时，会预分配额外的空间，减少内存分配次数。
- **二进制安全**：可以存储任意二进制数据，包括''字符。
- **长度计算**：O(1)时间复杂度获取字符串长度。

#### SDS结构示意图

```
+--------+-------------------------------+------------------+
| 长度   | 已使用空间                    | 未使用空间       |
+--------+-------------------------------+------------------+
| 4字节  | 字符串内容                     | 额外分配的空间   |
+--------+-------------------------------+------------------+
```

### 3.1.3 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `SET key value` | 设置键值对 | `SET name "Redis"` |
| `GET key` | 获取键对应的值 | `GET name` |
| `SETNX key value` | 当键不存在时设置键值对 | `SETNX lock "1"` |
| `SETEX key seconds value` | 设置键值对并指定过期时间 | `SETEX session 3600 "user:123"` |
| `PSETEX key milliseconds value` | 设置键值对并指定过期时间（毫秒） | `PSETEX session 3600000 "user:123"` |
| `INCR key` | 将键对应的值加1 | `INCR counter` |
| `DECR key` | 将键对应的值减1 | `DECR counter` |
| `INCRBY key increment` | 将键对应的值增加指定的数值 | `INCRBY counter 10` |
| `DECRBY key decrement` | 将键对应的值减少指定的数值 | `DECRBY counter 5` |
| `INCRBYFLOAT key increment` | 将键对应的值增加指定的浮点数 | `INCRBYFLOAT score 0.5` |
| `APPEND key value` | 将值追加到键对应的值后面 | `APPEND message " world"` |
| `STRLEN key` | 获取键对应的值的长度 | `STRLEN message` |
| `GETRANGE key start end` | 获取键对应的值的指定范围 | `GETRANGE message 0 4` |
| `SETRANGE key offset value` | 设置键对应的值的指定范围 | `SETRANGE message 0 "Hello"` |
| `GETSET key value` | 获取键对应的值并设置新值 | `GETSET counter "0"` |

### 3.1.4 使用场景

- **缓存**：存储热点数据，如用户信息、商品信息等。
- **计数器**：实现网站访问量、点赞数、下载量等计数功能。
- **分布式锁**：使用SETNX命令实现分布式锁。
- **会话存储**：存储用户会话信息，如登录状态、购物车信息等。
- **验证码**：存储短信验证码、邮箱验证码等。

### 3.1.5 示例

```bash
# 设置键值对
127.0.0.1:6379> SET name "Redis"
OK

# 获取键对应的值
127.0.0.1:6379> GET name
"Redis"

# 增加计数器
127.0.0.1:6379> INCR page_views
(integer) 1
127.0.0.1:6379> INCR page_views
(integer) 2

# 设置带过期时间的键值对
127.0.0.1:6379> SETEX session 3600 "user:123"
OK

# 检查键是否存在
127.0.0.1:6379> EXISTS session
(integer) 1

# 获取键的剩余过期时间
127.0.0.1:6379> TTL session
(integer) 3595
```

## 3.2 哈希（Hashes）

### 3.2.1 概述

哈希是一个键值对集合，适合存储对象。每个哈希可以存储最多2^32-1个键值对。

### 3.2.2 内部实现

Redis的哈希使用两种数据结构实现：

- **压缩列表（ziplist）**：当哈希的字段数量较少且字段值较小时，使用压缩列表存储，节省内存。
- **哈希表（hashtable）**：当哈希的字段数量较多或字段值较大时，使用哈希表存储，提高查找性能。

#### 压缩列表示意图

```
+--------+--------+--------+--------+--------+--------+
| 字段1  | 值1    | 字段2  | 值2    | 字段3  | 值3    |
+--------+--------+--------+--------+--------+--------+
```

#### 哈希表示意图

```
+--------+--------+
| 索引   | 键值对 |
+--------+--------+
| 0      | 字段1:值1 |
| 1      | 字段2:值2 |
| 2      | 字段3:值3 |
+--------+--------+
```

### 3.2.3 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `HSET key field value` | 设置哈希字段的值 | `HSET user:1 name "张三"` |
| `HGET key field` | 获取哈希字段的值 | `HGET user:1 name` |
| `HMSET key field1 value1 field2 value2 ...` | 批量设置哈希字段的值 | `HMSET user:1 name "张三" age 18` |
| `HMGET key field1 field2 ...` | 批量获取哈希字段的值 | `HMGET user:1 name age` |
| `HGETALL key` | 获取哈希的所有字段和值 | `HGETALL user:1` |
| `HDEL key field1 field2 ...` | 删除哈希的字段 | `HDEL user:1 age` |
| `HLEN key` | 获取哈希的字段数量 | `HLEN user:1` |
| `HEXISTS key field` | 判断哈希字段是否存在 | `HEXISTS user:1 name` |
| `HKEYS key` | 获取哈希的所有字段 | `HKEYS user:1` |
| `HVALS key` | 获取哈希的所有值 | `HVALS user:1` |
| `HINCRBY key field increment` | 增加哈希字段的数值 | `HINCRBY user:1 score 10` |
| `HINCRBYFLOAT key field increment` | 增加哈希字段的浮点数 | `HINCRBYFLOAT user:1 score 0.5` |
| `HSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代哈希的字段和值 | `HSCAN user:1 0` |

### 3.2.4 使用场景

- **存储对象**：存储用户信息、商品信息、订单信息等对象。
- **配置管理**：存储应用程序的配置信息。
- **统计数据**：存储各种统计数据，如用户活跃度、商品销量等。

### 3.2.5 示例

```bash
# 设置用户信息
127.0.0.1:6379> HMSET user:1 name "张三" age 18 gender "男" score 90
OK

# 获取用户姓名
127.0.0.1:6379> HGET user:1 name
"张三"

# 获取用户的所有信息
127.0.0.1:6379> HGETALL user:1
1) "name"
2) "张三"
3) "age"
4) "18"
5) "gender"
6) "男"
7) "score"
8) "90"

# 增加用户分数
127.0.0.1:6379> HINCRBY user:1 score 5
(integer) 95

# 删除用户性别字段
127.0.0.1:6379> HDEL user:1 gender
(integer) 1

# 获取用户的所有字段
127.0.0.1:6379> HKEYS user:1
1) "name"
2) "age"
3) "score"
```

## 3.3 列表（Lists）

### 3.3.1 概述

列表是一个有序的字符串集合，支持在两端进行插入和删除操作。列表的最大长度为2^32-1个元素。

### 3.3.2 内部实现

Redis的列表使用两种数据结构实现：

- **压缩列表（ziplist）**：当列表的元素数量较少且元素值较小时，使用压缩列表存储，节省内存。
- **双向链表（linkedlist）**：当列表的元素数量较多或元素值较大时，使用双向链表存储，提高插入和删除性能。

#### 双向链表示意图

```
+--------+--------+--------+
| 头节点 | 中间节点 | 尾节点 |
+--------+--------+--------+
| 元素1  | 元素2  | 元素3  |
+--------+--------+--------+
| 指向下一个节点 | 指向下一个节点 | 指向null |
| 指向null   | 指向上一个节点 | 指向上一个节点 |
+--------+--------+--------+
```

### 3.3.3 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `LPUSH key value1 value2 ...` | 在列表左侧插入一个或多个元素 | `LPUSH list a b c` |
| `RPUSH key value1 value2 ...` | 在列表右侧插入一个或多个元素 | `RPUSH list d e f` |
| `LPOP key` | 从列表左侧弹出一个元素 | `LPOP list` |
| `RPOP key` | 从列表右侧弹出一个元素 | `RPOP list` |
| `LLEN key` | 获取列表的长度 | `LLEN list` |
| `LRANGE key start stop` | 获取列表指定范围的元素 | `LRANGE list 0 -1` |
| `LREM key count value` | 删除列表中指定数量的指定值 | `LREM list 2 a` |
| `LSET key index value` | 设置列表指定索引位置的元素值 | `LSET list 0 "first"` |
| `LINSERT key BEFORE|AFTER pivot value` | 在列表指定元素的前或后插入元素 | `LINSERT list BEFORE "b" "new"` |
| `LPOS key value [RANK rank] [COUNT count] [MAXLEN len]` | 查找列表中指定值的位置 | `LPOS list "a"` |
| `LTRIM key start stop` | 修剪列表，只保留指定范围的元素 | `LTRIM list 0 2` |
| `BLPOP key1 key2 ... timeout` | 阻塞式从列表左侧弹出元素 | `BLPOP list 0` |
| `BRPOP key1 key2 ... timeout` | 阻塞式从列表右侧弹出元素 | `BRPOP list 0` |
| `BRPOPLPUSH source destination timeout` | 阻塞式从源列表右侧弹出元素并压入目标列表左侧 | `BRPOPLPUSH list1 list2 0` |

### 3.3.4 使用场景

- **消息队列**：使用列表的LPUSH和RPOP命令实现简单的消息队列。
- **最新消息**：使用LPUSH和LRANGE命令实现最新消息列表。
- **任务队列**：使用LPUSH和BLPOP命令实现任务队列。
- **栈**：使用LPUSH和LPOP命令实现栈。
- **队列**：使用LPUSH和RPOP命令实现队列。

### 3.3.5 示例

```bash
# 在列表左侧插入元素
127.0.0.1:6379> LPUSH list a b c
(integer) 3

# 在列表右侧插入元素
127.0.0.1:6379> RPUSH list d e f
(integer) 6

# 获取列表的长度
127.0.0.1:6379> LLEN list
(integer) 6

# 获取列表的所有元素
127.0.0.1:6379> LRANGE list 0 -1
1) "c"
2) "b"
3) "a"
4) "d"
5) "e"
6) "f"

# 从列表左侧弹出元素
127.0.0.1:6379> LPOP list
"c"

# 从列表右侧弹出元素
127.0.0.1:6379> RPOP list
"f"

# 修剪列表，只保留前3个元素
127.0.0.1:6379> LTRIM list 0 2
OK

# 获取修剪后的列表元素
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "a"
3) "d"
```

## 3.4 集合（Sets）

### 3.4.1 概述

集合是一个无序的字符串集合，不允许重复元素。集合的最大元素个数为2^32-1。

### 3.4.2 内部实现

Redis的集合使用两种数据结构实现：

- **整数集合（intset）**：当集合中的元素都是整数且元素数量较少时，使用整数集合存储，节省内存。
- **哈希表（hashtable）**：当集合中的元素包含非整数或元素数量较多时，使用哈希表存储，提高查找性能。

#### 哈希表示意图

```
+--------+--------+
| 索引   | 元素   |
+--------+--------+
| 0      | a      |
| 1      | b      |
| 2      | c      |
+--------+--------+
```

### 3.4.3 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `SADD key member1 member2 ...` | 向集合添加一个或多个元素 | `SADD set a b c` |
| `SREM key member1 member2 ...` | 从集合删除一个或多个元素 | `SREM set c` |
| `SMEMBERS key` | 获取集合的所有元素 | `SMEMBERS set` |
| `SISMEMBER key member` | 判断元素是否在集合中 | `SISMEMBER set a` |
| `SCARD key` | 获取集合的元素个数 | `SCARD set` |
| `SPOP key [count]` | 从集合中随机弹出一个或多个元素 | `SPOP set` |
| `SRANDMEMBER key [count]` | 从集合中随机获取一个或多个元素 | `SRANDMEMBER set 2` |
| `SUNION key1 key2 ...` | 获取多个集合的并集 | `SUNION set1 set2` |
| `SUNIONSTORE destination key1 key2 ...` | 获取多个集合的并集并存储到目标集合 | `SUNIONSTORE union_set set1 set2` |
| `SINTER key1 key2 ...` | 获取多个集合的交集 | `SINTER set1 set2` |
| `SINTERSTORE destination key1 key2 ...` | 获取多个集合的交集并存储到目标集合 | `SINTERSTORE inter_set set1 set2` |
| `SDIFF key1 key2 ...` | 获取多个集合的差集 | `SDIFF set1 set2` |
| `SDIFFSTORE destination key1 key2 ...` | 获取多个集合的差集并存储到目标集合 | `SDIFFSTORE diff_set set1 set2` |
| `SSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代集合的元素 | `SSCAN set 0` |

### 3.4.4 使用场景

- **去重**：存储唯一的元素，如用户ID、商品ID等。
- **交集/并集/差集操作**：如获取共同关注的用户、共同喜欢的商品等。
- **随机数生成**：使用SRANDMEMBER命令生成随机数。
- **标签系统**：存储用户的标签、商品的分类等。

### 3.4.5 示例

```bash
# 向集合添加元素
127.0.0.1:6379> SADD set1 a b c d
(integer) 4
127.0.0.1:6379> SADD set2 c d e f
(integer) 4

# 获取集合的所有元素
127.0.0.1:6379> SMEMBERS set1
1) "a"
2) "b"
3) "c"
4) "d"

# 判断元素是否在集合中
127.0.0.1:6379> SISMEMBER set1 a
(integer) 1
127.0.0.1:6379> SISMEMBER set1 e
(integer) 0

# 获取集合的元素个数
127.0.0.1:6379> SCARD set1
(integer) 4

# 获取两个集合的并集
127.0.0.1:6379> SUNION set1 set2
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"

# 获取两个集合的交集
127.0.0.1:6379> SINTER set1 set2
1) "c"
2) "d"

# 获取两个集合的差集
127.0.0.1:6379> SDIFF set1 set2
1) "a"
2) "b"
```

## 3.5 有序集合（Sorted Sets）

### 3.5.1 概述

有序集合是一个有序的字符串集合，每个元素都有一个分数，根据分数进行排序。有序集合的最大元素个数为2^32-1。

### 3.5.2 内部实现

Redis的有序集合使用两种数据结构实现：

- **压缩列表（ziplist）**：当有序集合的元素数量较少且元素值较小时，使用压缩列表存储，节省内存。
- **跳表（skiplist）+ 哈希表**：当有序集合的元素数量较多或元素值较大时，使用跳表和哈希表存储，提高查找和排序性能。

#### 跳表示意图

```
+--------+--------+--------+--------+
| 层3    | 元素1 (10) |        | 元素3 (30) |
+--------+--------+--------+--------+
| 层2    | 元素1 (10) | 元素2 (20) | 元素3 (30) |
+--------+--------+--------+--------+
| 层1    | 元素1 (10) | 元素2 (20) | 元素3 (30) |
+--------+--------+--------+--------+
| 层0    | 元素1 (10) | 元素2 (20) | 元素3 (30) |
+--------+--------+--------+--------+
```

### 3.5.3 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `ZADD key score1 member1 score2 member2 ...` | 向有序集合添加一个或多个元素 | `ZADD zset 10 a 20 b 30 c` |
| `ZREM key member1 member2 ...` | 从有序集合删除一个或多个元素 | `ZREM zset c` |
| `ZRANGE key start stop [WITHSCORES]` | 获取有序集合指定范围的元素（按分数升序） | `ZRANGE zset 0 -1 WITHSCORES` |
| `ZREVRANGE key start stop [WITHSCORES]` | 获取有序集合指定范围的元素（按分数降序） | `ZREVRANGE zset 0 -1 WITHSCORES` |
| `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` | 获取分数在指定范围内的元素 | `ZRANGEBYSCORE zset 15 30 WITHSCORES` |
| `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]` | 获取分数在指定范围内的元素（按分数降序） | `ZREVRANGEBYSCORE zset 30 15 WITHSCORES` |
| `ZRANK key member` | 获取元素在有序集合中的排名（按分数升序） | `ZRANK zset a` |
| `ZREVRANK key member` | 获取元素在有序集合中的排名（按分数降序） | `ZREVRANK zset a` |
| `ZSCORE key member` | 获取元素的分数 | `ZSCORE zset a` |
| `ZINCRBY key increment member` | 增加元素的分数 | `ZINCRBY zset 5 a` |
| `ZCARD key` | 获取有序集合的元素个数 | `ZCARD zset` |
| `ZCOUNT key min max` | 获取分数在指定范围内的元素个数 | `ZCOUNT zset 10 30` |
| `ZREMRangeByRank key start stop` | 删除指定排名范围内的元素 | `ZREMRANGEBYRANK zset 0 1` |
| `ZREMRangeByScore key min max` | 删除分数在指定范围内的元素 | `ZREMRANGEBYSCORE zset 10 20` |
| `ZUNIONSTORE destination numkeys key1 key2 ... [WEIGHTS weight1 weight2 ...] [AGGREGATE SUM|MIN|MAX]` | 获取多个有序集合的并集并存储到目标集合 | `ZUNIONSTORE union_zset 2 zset1 zset2` |
| `ZINTERSTORE destination numkeys key1 key2 ... [WEIGHTS weight1 weight2 ...] [AGGREGATE SUM|MIN|MAX]` | 获取多个有序集合的交集并存储到目标集合 | `ZINTERSTORE inter_zset 2 zset1 zset2` |
| `ZSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代有序集合的元素 | `ZSCAN zset 0` |

### 3.5.4 使用场景

- **排行榜**：根据分数进行排名，如游戏排行榜、商品销量排行榜、用户积分排行榜等。
- **优先级队列**：根据分数设置优先级，分数越高优先级越高。
- **范围查询**：根据分数范围查询元素，如查询某个分数段的用户。
- **时间序列数据**：使用时间戳作为分数，存储时间序列数据。

### 3.5.5 示例

```bash
# 向有序集合添加元素
127.0.0.1:6379> ZADD ranking 100 user:1 200 user:2 300 user:3 400 user:4 500 user:5
(integer) 5

# 获取排行榜前3名（按分数降序）
127.0.0.1:6379> ZREVRANGE ranking 0 2 WITHSCORES
1) "user:5"
2) "500"
3) "user:4"
4) "400"
5) "user:3"
6) "300"

# 获取用户:2的分数
127.0.0.1:6379> ZSCORE ranking user:2
"200"

# 增加用户:1的分数
127.0.0.1:6379> ZINCRBY ranking 50 user:1
"150"

# 获取分数在200-400之间的用户
127.0.0.1:6379> ZRANGEBYSCORE ranking 200 400 WITHSCORES
1) "user:2"
2) "200"
3) "user:3"
4) "300"
5) "user:4"
6) "400"

# 获取用户:3的排名（按分数降序）
127.0.0.1:6379> ZREVRANK ranking user:3
(integer) 2
```

## 3.6 其他数据结构

### 3.6.1 位图（Bitmaps）

#### 概述

位图是一种特殊的字符串，它可以对每个位进行操作，适合存储布尔值。

#### 位图示意图

```
+--------+--------+--------+--------+--------+
| 位0    | 位1    | 位2    | 位3    | 位4    |
+--------+--------+--------+--------+--------+
| 0      | 1      | 0      | 1      | 0      |
+--------+--------+--------+--------+--------+
```

#### 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `SETBIT key offset value` | 设置位图指定位置的值 | `SETBIT user:online 1 1` |
| `GETBIT key offset` | 获取位图指定位置的值 | `GETBIT user:online 1` |
| `BITCOUNT key [start end]` | 统计位图中值为1的位数 | `BITCOUNT user:online` |
| `BITOP operation destkey key1 key2 ...` | 对多个位图执行位操作 | `BITOP AND result bitmap1 bitmap2` |
| `BITPOS key value [start end]` | 查找位图中指定值的位置 | `BITPOS user:online 1` |

#### 使用场景

- **用户在线状态**：使用位图存储用户的在线状态，1表示在线，0表示离线。
- **布隆过滤器**：使用位图实现布隆过滤器，用于判断一个元素是否存在于集合中。
- **权限管理**：使用位图存储用户的权限，每个位代表一种权限。

### 3.6.2 HyperLogLog

#### 概述

HyperLogLog是一种用于基数统计的数据结构，它可以以极小的空间代价估计一个集合的基数（元素个数）。

#### 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `PFADD key element1 element2 ...` | 向HyperLogLog添加元素 | `PFADD unique:users user:1 user:2 user:3` |
| `PFCOUNT key1 key2 ...` | 统计HyperLogLog的基数 | `PFCOUNT unique:users` |
| `PFMERGE destkey sourcekey1 sourcekey2 ...` | 合并多个HyperLogLog | `PFMERGE unique:all unique:users1 unique:users2` |

#### 使用场景

- **独立访客统计**：统计网站的独立访客数量。
- **用户行为统计**：统计用户的独立操作次数。

### 3.6.3 地理空间（Geospatial）

#### 概述

地理空间数据结构用于存储地理位置信息，并支持基于地理位置的查询。

#### 常用命令

| 命令 | 描述 | 示例 |
|------|------|------|
| `GEOADD key longitude latitude member [longitude latitude member ...]` | 添加地理位置信息 | `GEOADD locations 116.4042 39.9153 beijing 121.4737 31.2304 shanghai` |
| `GEODIST key member1 member2 [unit]` | 计算两个地理位置之间的距离 | `GEODIST locations beijing shanghai km` |
| `GEOHASH key member1 member2 ...` | 获取地理位置的Geohash编码 | `GEOHASH locations beijing shanghai` |
| `GEOPOS key member1 member2 ...` | 获取地理位置的经纬度 | `GEOPOS locations beijing shanghai` |
| `GEORADIUS key longitude latitude radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC]` | 查找指定范围内的地理位置 | `GEORADIUS locations 116.4042 39.9153 1000 km WITHCOORD WITHDIST` |
| `GEORADIUSBYMEMBER key member radius unit [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC]` | 查找指定成员周围的地理位置 | `GEORADIUSBYMEMBER locations beijing 1000 km` |

#### 使用场景

- **附近的人**：查找用户周围的其他用户。
- **位置查询**：查找指定范围内的商家、景点等。
- **距离计算**：计算两个位置之间的距离。

## 3.7 小结

本章节详细介绍了Redis的各种数据结构，包括字符串、哈希、列表、集合、有序集合以及位图、HyperLogLog和地理空间等特殊数据结构。每种数据结构都有其特定的特点和应用场景：

- **字符串**：适合存储简单的键值对、计数器等。
- **哈希**：适合存储对象，如用户信息、商品信息等。
- **列表**：适合实现消息队列、最新消息等。
- **集合**：适合存储唯一元素，实现去重、交集/并集/差集操作等。
- **有序集合**：适合实现排行榜、优先级队列等。
- **位图**：适合存储布尔值，如用户在线状态、布隆过滤器等。
- **HyperLogLog**：适合基数统计，如独立访客统计等。
- **地理空间**：适合存储地理位置信息，实现附近的人、位置查询等。

通过本章节的学习，您应该能够根据不同的业务场景选择合适的Redis数据结构，并掌握各种数据结构的常用命令和使用方法。在接下来的章节中，我们将详细介绍Redis的常用命令、持久化机制、高可用方案以及在Go语言后端开发中的实际应用。