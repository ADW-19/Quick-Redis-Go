# 4. Redis常用命令

## 4.1 键操作

### 4.1.1 概述

键操作命令用于管理Redis中的键，包括查找、检查、删除、设置过期时间等操作。

### 4.1.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `KEYS pattern` | 查找所有匹配给定模式的键 | `KEYS user:*` | O(N)，N为数据库中的键数量 |
| `EXISTS key` | 检查键是否存在 | `EXISTS user:1` | O(1) |
| `DEL key [key ...]` | 删除一个或多个键 | `DEL user:1 user:2` | O(N)，N为删除的键数量 |
| `EXPIRE key seconds` | 设置键的过期时间（秒） | `EXPIRE session 3600` | O(1) |
| `PEXPIRE key milliseconds` | 设置键的过期时间（毫秒） | `PEXPIRE session 3600000` | O(1) |
| `EXPIREAT key timestamp` | 设置键的过期时间戳（秒） | `EXPIREAT session 1678800000` | O(1) |
| `PEXPIREAT key milliseconds-timestamp` | 设置键的过期时间戳（毫秒） | `PEXPIREAT session 1678800000000` | O(1) |
| `TTL key` | 获取键的剩余过期时间（秒） | `TTL session` | O(1) |
| `PTTL key` | 获取键的剩余过期时间（毫秒） | `PTTL session` | O(1) |
| `PERSIST key` | 移除键的过期时间 | `PERSIST session` | O(1) |
| `RENAME key newkey` | 重命名键 | `RENAME user:1 user:2` | O(1) |
| `RENAMENX key newkey` | 当新键不存在时重命名键 | `RENAMENX user:1 user:2` | O(1) |
| `TYPE key` | 获取键的类型 | `TYPE user:1` | O(1) |
| `SCAN cursor [MATCH pattern] [COUNT count]` | 迭代数据库中的键 | `SCAN 0 MATCH user:* COUNT 10` | O(1)，每次迭代只处理COUNT个键 |

### 4.1.3 示例

```bash
# 查找所有以user:开头的键
127.0.0.1:6379> KEYS user:*
1) "user:1"
2) "user:2"
3) "user:3"

# 检查键是否存在
127.0.0.1:6379> EXISTS user:1
(integer) 1
127.0.0.1:6379> EXISTS user:10
(integer) 0

# 删除键
127.0.0.1:6379> DEL user:3
(integer) 1

# 设置键的过期时间为1小时
127.0.0.1:6379> EXPIRE session 3600
(integer) 1

# 获取键的剩余过期时间
127.0.0.1:6379> TTL session
(integer) 3595

# 移除键的过期时间
127.0.0.1:6379> PERSIST session
(integer) 1

# 重命名键
127.0.0.1:6379> RENAME user:1 user:10
OK

# 获取键的类型
127.0.0.1:6379> TYPE user:10
hash

# 使用SCAN迭代键
127.0.0.1:6379> SCAN 0 MATCH user:* COUNT 10
1) "0"
2) 1) "user:2"
   2) "user:10"
```

### 4.1.4 注意事项

- **KEYS命令**：在生产环境中应避免使用`KEYS`命令，因为它会遍历所有键，当键数量较多时会阻塞Redis。建议使用`SCAN`命令代替。
- **DEL命令**：删除大键时可能会阻塞Redis，建议使用`UNLINK`命令（Redis 4.0+）异步删除大键。
- **EXPIRE命令**：设置过期时间的键会被Redis自动删除，但删除操作可能会延迟执行。

## 4.2 字符串操作

### 4.2.1 概述

字符串是Redis最基本的数据类型，字符串操作命令用于管理字符串类型的键值对。

### 4.2.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `SET key value [EX seconds] [PX milliseconds] [NX|XX]` | 设置键值对 | `SET name "Redis" EX 3600` | O(1) |
| `GET key` | 获取键对应的值 | `GET name` | O(1) |
| `SETNX key value` | 当键不存在时设置键值对 | `SETNX lock "1"` | O(1) |
| `SETEX key seconds value` | 设置键值对并指定过期时间 | `SETEX session 3600 "user:123"` | O(1) |
| `PSETEX key milliseconds value` | 设置键值对并指定过期时间（毫秒） | `PSETEX session 3600000 "user:123"` | O(1) |
| `INCR key` | 将键对应的值加1 | `INCR counter` | O(1) |
| `DECR key` | 将键对应的值减1 | `DECR counter` | O(1) |
| `INCRBY key increment` | 将键对应的值增加指定的数值 | `INCRBY counter 10` | O(1) |
| `DECRBY key decrement` | 将键对应的值减少指定的数值 | `DECRBY counter 5` | O(1) |
| `INCRBYFLOAT key increment` | 将键对应的值增加指定的浮点数 | `INCRBYFLOAT score 0.5` | O(1) |
| `APPEND key value` | 将值追加到键对应的值后面 | `APPEND message " world"` | O(1)，追加的字符串长度 |
| `STRLEN key` | 获取键对应的值的长度 | `STRLEN message` | O(1) |
| `GETRANGE key start end` | 获取键对应的值的指定范围 | `GETRANGE message 0 4` | O(1)，获取的字符串长度 |
| `SETRANGE key offset value` | 设置键对应的值的指定范围 | `SETRANGE message 0 "Hello"` | O(1)，设置的字符串长度 |
| `GETSET key value` | 获取键对应的值并设置新值 | `GETSET counter "0"` | O(1) |
| `MSET key value [key value ...]` | 批量设置键值对 | `MSET name "Redis" version "7.0"` | O(N)，N为设置的键数量 |
| `MGET key [key ...]` | 批量获取键对应的值 | `MGET name version` | O(N)，N为获取的键数量 |
| `MSETNX key value [key value ...]` | 当所有键不存在时批量设置键值对 | `MSETNX name "Redis" version "7.0"` | O(N)，N为设置的键数量 |

### 4.2.3 示例

```bash
# 设置键值对
127.0.0.1:6379> SET name "Redis"
OK

# 获取键对应的值
127.0.0.1:6379> GET name
"Redis"

# 当键不存在时设置键值对
127.0.0.1:6379> SETNX lock "1"
(integer) 1

# 设置带过期时间的键值对
127.0.0.1:6379> SETEX session 3600 "user:123"
OK

# 增加计数器
127.0.0.1:6379> INCR counter
(integer) 1
127.0.0.1:6379> INCRBY counter 10
(integer) 11

# 追加字符串
127.0.0.1:6379> APPEND message "Hello"
(integer) 5
127.0.0.1:6379> APPEND message " world"
(integer) 11

# 获取字符串长度
127.0.0.1:6379> STRLEN message
(integer) 11

# 获取字符串的指定范围
127.0.0.1:6379> GETRANGE message 0 4
"Hello"

# 批量设置键值对
127.0.0.1:6379> MSET name "Redis" version "7.0" author "Antirez"
OK

# 批量获取键值对
127.0.0.1:6379> MGET name version author
1) "Redis"
2) "7.0"
3) "Antirez"
```

### 4.2.4 注意事项

- **SET命令**：`SET`命令可以设置过期时间、条件设置（NX/XX）等选项，功能非常强大。
- **INCR命令**：`INCR`命令只能用于数值类型的字符串，否则会返回错误。
- **GETSET命令**：`GETSET`命令是原子操作，常用于实现计数器的重置。

## 4.3 哈希操作

### 4.3.1 概述

哈希是一个键值对集合，哈希操作命令用于管理哈希类型的键值对。

### 4.3.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `HSET key field value` | 设置哈希字段的值 | `HSET user:1 name "张三"` | O(1) |
| `HGET key field` | 获取哈希字段的值 | `HGET user:1 name` | O(1) |
| `HMSET key field1 value1 field2 value2 ...` | 批量设置哈希字段的值 | `HMSET user:1 name "张三" age 18` | O(N)，N为设置的字段数量 |
| `HMGET key field1 field2 ...` | 批量获取哈希字段的值 | `HMGET user:1 name age` | O(N)，N为获取的字段数量 |
| `HGETALL key` | 获取哈希的所有字段和值 | `HGETALL user:1` | O(N)，N为哈希的字段数量 |
| `HDEL key field1 field2 ...` | 删除哈希的字段 | `HDEL user:1 age` | O(N)，N为删除的字段数量 |
| `HLEN key` | 获取哈希的字段数量 | `HLEN user:1` | O(1) |
| `HEXISTS key field` | 判断哈希字段是否存在 | `HEXISTS user:1 name` | O(1) |
| `HKEYS key` | 获取哈希的所有字段 | `HKEYS user:1` | O(N)，N为哈希的字段数量 |
| `HVALS key` | 获取哈希的所有值 | `HVALS user:1` | O(N)，N为哈希的字段数量 |
| `HINCRBY key field increment` | 增加哈希字段的数值 | `HINCRBY user:1 score 10` | O(1) |
| `HINCRBYFLOAT key field increment` | 增加哈希字段的浮点数 | `HINCRBYFLOAT user:1 score 0.5` | O(1) |
| `HSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代哈希的字段和值 | `HSCAN user:1 0` | O(1)，每次迭代只处理COUNT个字段 |

### 4.3.3 示例

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

# 获取用户的所有值
127.0.0.1:6379> HVALS user:1
1) "张三"
2) "18"
3) "95"

# 批量获取用户信息
127.0.0.1:6379> HMGET user:1 name age score
1) "张三"
2) "18"
3) "95"
```

### 4.3.4 注意事项

- **HGETALL命令**：当哈希的字段数量较多时，`HGETALL`命令可能会阻塞Redis，建议使用`HSCAN`命令代替。
- **HMSET命令**：在Redis 4.0+中，`HMSET`命令已被废弃，建议使用`HSET`命令代替，`HSET`命令现在支持批量设置字段。

## 4.4 列表操作

### 4.4.1 概述

列表是一个有序的字符串集合，列表操作命令用于管理列表类型的键值对。

### 4.4.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `LPUSH key value [value ...]` | 在列表左侧插入一个或多个元素 | `LPUSH list a b c` | O(N)，N为插入的元素数量 |
| `RPUSH key value [value ...]` | 在列表右侧插入一个或多个元素 | `RPUSH list d e f` | O(N)，N为插入的元素数量 |
| `LPOP key` | 从列表左侧弹出一个元素 | `LPOP list` | O(1) |
| `RPOP key` | 从列表右侧弹出一个元素 | `RPOP list` | O(1) |
| `LLEN key` | 获取列表的长度 | `LLEN list` | O(1) |
| `LRANGE key start stop` | 获取列表指定范围的元素 | `LRANGE list 0 -1` | O(N)，N为获取的元素数量 |
| `LREM key count value` | 删除列表中指定数量的指定值 | `LREM list 2 a` | O(N)，N为列表的长度 |
| `LSET key index value` | 设置列表指定索引位置的元素值 | `LSET list 0 "first"` | O(N)，N为列表的长度 |
| `LINSERT key BEFORE|AFTER pivot value` | 在列表指定元素的前或后插入元素 | `LINSERT list BEFORE "b" "new"` | O(N)，N为列表的长度 |
| `LPOS key value [RANK rank] [COUNT count] [MAXLEN len]` | 查找列表中指定值的位置 | `LPOS list "a"` | O(N)，N为列表的长度 |
| `LTRIM key start stop` | 修剪列表，只保留指定范围的元素 | `LTRIM list 0 2` | O(N)，N为列表的长度 |
| `BLPOP key [key ...] timeout` | 阻塞式从列表左侧弹出元素 | `BLPOP list 0` | O(1) |
| `BRPOP key [key ...] timeout` | 阻塞式从列表右侧弹出元素 | `BRPOP list 0` | O(1) |
| `BRPOPLPUSH source destination timeout` | 阻塞式从源列表右侧弹出元素并压入目标列表左侧 | `BRPOPLPUSH list1 list2 0` | O(1) |

### 4.4.3 示例

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

# 在指定元素前插入新元素
127.0.0.1:6379> LINSERT list BEFORE "a" "new"
(integer) 4

# 获取插入后的列表元素
127.0.0.1:6379> LRANGE list 0 -1
1) "b"
2) "new"
3) "a"
4) "d"
```

### 4.4.4 注意事项

- **BLPOP和BRPOP命令**：这两个命令是阻塞式命令，当列表为空时会阻塞连接，直到列表中有元素或超时。
- **LREM命令**：`count`参数的含义：
  - `count > 0`：从列表左侧开始删除指定值的元素，删除`count`个。
  - `count < 0`：从列表右侧开始删除指定值的元素，删除`abs(count)`个。
  - `count = 0`：删除列表中所有指定值的元素。

## 4.5 集合操作

### 4.5.1 概述

集合是一个无序的字符串集合，不允许重复元素，集合操作命令用于管理集合类型的键值对。

### 4.5.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `SADD key member [member ...]` | 向集合添加一个或多个元素 | `SADD set a b c` | O(N)，N为添加的元素数量 |
| `SREM key member [member ...]` | 从集合删除一个或多个元素 | `SREM set c` | O(N)，N为删除的元素数量 |
| `SMEMBERS key` | 获取集合的所有元素 | `SMEMBERS set` | O(N)，N为集合的元素数量 |
| `SISMEMBER key member` | 判断元素是否在集合中 | `SISMEMBER set a` | O(1) |
| `SCARD key` | 获取集合的元素个数 | `SCARD set` | O(1) |
| `SPOP key [count]` | 从集合中随机弹出一个或多个元素 | `SPOP set` | O(1) |
| `SRANDMEMBER key [count]` | 从集合中随机获取一个或多个元素 | `SRANDMEMBER set 2` | O(N)，N为获取的元素数量 |
| `SUNION key [key ...]` | 获取多个集合的并集 | `SUNION set1 set2` | O(N)，N为所有集合的元素数量之和 |
| `SUNIONSTORE destination key [key ...]` | 获取多个集合的并集并存储到目标集合 | `SUNIONSTORE union_set set1 set2` | O(N)，N为所有集合的元素数量之和 |
| `SINTER key [key ...]` | 获取多个集合的交集 | `SINTER set1 set2` | O(N)，N为所有集合中元素数量最少的集合的元素数量 |
| `SINTERSTORE destination key [key ...]` | 获取多个集合的交集并存储到目标集合 | `SINTERSTORE inter_set set1 set2` | O(N)，N为所有集合中元素数量最少的集合的元素数量 |
| `SDIFF key [key ...]` | 获取多个集合的差集 | `SDIFF set1 set2` | O(N)，N为第一个集合的元素数量 |
| `SDIFFSTORE destination key [key ...]` | 获取多个集合的差集并存储到目标集合 | `SDIFFSTORE diff_set set1 set2` | O(N)，N为第一个集合的元素数量 |
| `SSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代集合的元素 | `SSCAN set 0` | O(1)，每次迭代只处理COUNT个元素 |

### 4.5.3 示例

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

# 从集合中随机获取元素
127.0.0.1:6379> SRANDMEMBER set1 2
1) "a"
2) "c"

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

### 4.5.4 注意事项

- **SMEMBERS命令**：当集合的元素数量较多时，`SMEMBERS`命令可能会阻塞Redis，建议使用`SSCAN`命令代替。
- **SRANDMEMBER命令**：当`count`参数为正数时，返回的元素不重复；当`count`参数为负数时，返回的元素可能重复。

## 4.6 有序集合操作

### 4.6.1 概述

有序集合是一个有序的字符串集合，每个元素都有一个分数，根据分数进行排序，有序集合操作命令用于管理有序集合类型的键值对。

### 4.6.2 常用命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `ZADD key score member [score member ...]` | 向有序集合添加一个或多个元素 | `ZADD zset 10 a 20 b 30 c` | O(M*log(N))，M为添加的元素数量，N为有序集合的元素数量 |
| `ZREM key member [member ...]` | 从有序集合删除一个或多个元素 | `ZREM zset c` | O(M*log(N))，M为删除的元素数量，N为有序集合的元素数量 |
| `ZRANGE key start stop [WITHSCORES]` | 获取有序集合指定范围的元素（按分数升序） | `ZRANGE zset 0 -1 WITHSCORES` | O(log(N)+M)，M为获取的元素数量，N为有序集合的元素数量 |
| `ZREVRANGE key start stop [WITHSCORES]` | 获取有序集合指定范围的元素（按分数降序） | `ZREVRANGE zset 0 -1 WITHSCORES` | O(log(N)+M)，M为获取的元素数量，N为有序集合的元素数量 |
| `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` | 获取分数在指定范围内的元素 | `ZRANGEBYSCORE zset 15 30 WITHSCORES` | O(log(N)+M)，M为获取的元素数量，N为有序集合的元素数量 |
| `ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]` | 获取分数在指定范围内的元素（按分数降序） | `ZREVRANGEBYSCORE zset 30 15 WITHSCORES` | O(log(N)+M)，M为获取的元素数量，N为有序集合的元素数量 |
| `ZRANK key member` | 获取元素在有序集合中的排名（按分数升序） | `ZRANK zset a` | O(log(N))，N为有序集合的元素数量 |
| `ZREVRANK key member` | 获取元素在有序集合中的排名（按分数降序） | `ZREVRANK zset a` | O(log(N))，N为有序集合的元素数量 |
| `ZSCORE key member` | 获取元素的分数 | `ZSCORE zset a` | O(1) |
| `ZINCRBY key increment member` | 增加元素的分数 | `ZINCRBY zset 5 a` | O(log(N))，N为有序集合的元素数量 |
| `ZCARD key` | 获取有序集合的元素个数 | `ZCARD zset` | O(1) |
| `ZCOUNT key min max` | 获取分数在指定范围内的元素个数 | `ZCOUNT zset 10 30` | O(log(N))，N为有序集合的元素数量 |
| `ZREMRANGEBYRANK key start stop` | 删除指定排名范围内的元素 | `ZREMRANGEBYRANK zset 0 1` | O(log(N)+M)，M为删除的元素数量，N为有序集合的元素数量 |
| `ZREMRANGEBYSCORE key min max` | 删除分数在指定范围内的元素 | `ZREMRANGEBYSCORE zset 10 20` | O(log(N)+M)，M为删除的元素数量，N为有序集合的元素数量 |
| `ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]` | 获取多个有序集合的并集并存储到目标集合 | `ZUNIONSTORE union_zset 2 zset1 zset2` | O(N)+O(M*log(M))，N为所有集合的元素数量之和，M为结果集合的元素数量 |
| `ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]` | 获取多个有序集合的交集并存储到目标集合 | `ZINTERSTORE inter_zset 2 zset1 zset2` | O(N*K)+O(M*log(M))，N为所有集合中元素数量最少的集合的元素数量，K为集合数量，M为结果集合的元素数量 |
| `ZSCAN key cursor [MATCH pattern] [COUNT count]` | 迭代有序集合的元素 | `ZSCAN zset 0` | O(1)，每次迭代只处理COUNT个元素 |

### 4.6.3 示例

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

# 删除排名前2的用户
127.0.0.1:6379> ZREMRANGEBYRANK ranking 0 1
(integer) 2

# 获取剩余的用户
127.0.0.1:6379> ZREVRANGE ranking 0 -1 WITHSCORES
1) "user:3"
2) "300"
3) "user:2"
4) "200"
5) "user:1"
6) "150"
```

### 4.6.4 注意事项

- **ZADD命令**：在Redis 3.0+中，`ZADD`命令支持`NX`、`XX`、`CH`等选项，可以更灵活地控制添加行为。
- **ZRANGEBYSCORE命令**：`min`和`max`参数可以使用特殊值，如`-inf`（负无穷）和`+inf`（正无穷）。
- **ZUNIONSTORE和ZINTERSTORE命令**：这两个命令可以指定权重和聚合方式，非常灵活。

## 4.7 其他命令

### 4.7.1 事务命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `MULTI` | 开始事务 | `MULTI` | O(1) |
| `EXEC` | 执行事务 | `EXEC` | O(N)，N为事务中的命令数量 |
| `DISCARD` | 取消事务 | `DISCARD` | O(1) |
| `WATCH key [key ...]` | 监视键，用于乐观锁 | `WATCH user:1` | O(1) |

### 4.7.2 发布/订阅命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `PUBLISH channel message` | 向频道发布消息 | `PUBLISH news "Hello World"` | O(N)，N为订阅该频道的客户端数量 |
| `SUBSCRIBE channel [channel ...]` | 订阅频道 | `SUBSCRIBE news` | O(1) |
| `UNSUBSCRIBE [channel [channel ...]]` | 取消订阅频道 | `UNSUBSCRIBE news` | O(1) |
| `PSUBSCRIBE pattern [pattern ...]` | 订阅匹配模式的频道 | `PSUBSCRIBE news:*` | O(1) |
| `PUNSUBSCRIBE [pattern [pattern ...]]` | 取消订阅匹配模式的频道 | `PUNSUBSCRIBE news:*` | O(1) |

### 4.7.3 脚本命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `EVAL script numkeys key [key ...] arg [arg ...]` | 执行Lua脚本 | `EVAL "return redis.call('GET', KEYS[1])" 1 name` | 取决于脚本的复杂度 |
| `EVALSHA sha1 numkeys key [key ...] arg [arg ...]` | 执行Lua脚本（使用脚本哈希） | `EVALSHA <sha1> 1 name` | 取决于脚本的复杂度 |
| `SCRIPT LOAD script` | 加载Lua脚本 | `SCRIPT LOAD "return redis.call('GET', KEYS[1])"` | O(N)，N为脚本的长度 |
| `SCRIPT EXISTS sha1 [sha1 ...]` | 检查脚本是否存在 | `SCRIPT EXISTS <sha1>` | O(N)，N为脚本的数量 |
| `SCRIPT FLUSH` | 清除所有脚本 | `SCRIPT FLUSH` | O(1) |
| `SCRIPT KILL` | 终止正在执行的脚本 | `SCRIPT KILL` | O(1) |

### 4.7.4 服务器命令

| 命令 | 描述 | 示例 | 时间复杂度 |
|------|------|------|------------|
| `PING` | 测试连接 | `PING` | O(1) |
| `INFO [section]` | 获取服务器信息 | `INFO` | O(1) |
| `CONFIG GET parameter` | 获取配置参数 | `CONFIG GET maxmemory` | O(1) |
| `CONFIG SET parameter value` | 设置配置参数 | `CONFIG SET maxmemory 2gb` | O(1) |
| `CONFIG REWRITE` | 重写配置文件 | `CONFIG REWRITE` | O(1) |
| `FLUSHDB` | 清空当前数据库 | `FLUSHDB` | O(N)，N为数据库中的键数量 |
| `FLUSHALL` | 清空所有数据库 | `FLUSHALL` | O(N)，N为所有数据库中的键数量 |
| `SAVE` | 同步保存数据到磁盘 | `SAVE` | O(N)，N为数据库中的键数量 |
| `BGSAVE` | 异步保存数据到磁盘 | `BGSAVE` | O(N)，N为数据库中的键数量 |
| `BGREWRITEAOF` | 异步重写AOF文件 | `BGREWRITEAOF` | O(N)，N为数据库中的键数量 |
| `SHUTDOWN [SAVE|NOSAVE]` | 关闭服务器 | `SHUTDOWN SAVE` | O(N)，N为数据库中的键数量 |

## 4.8 命令使用技巧

### 4.8.1 批量操作

- **使用MSET/MGET**：批量设置和获取键值对，减少网络往返时间。
- **使用HMSET/HMGET**：批量设置和获取哈希字段，减少网络往返时间。
- **使用管道（Pipeline）**：将多个命令打包发送，减少网络往返时间。

### 4.8.2 避免阻塞操作

- **避免使用KEYS命令**：在生产环境中使用SCAN命令代替。
- **避免使用HGETALL命令**：在哈希字段较多时使用HSCAN命令代替。
- **避免使用SMEMBERS命令**：在集合元素较多时使用SSCAN命令代替。
- **避免使用ZRANGE命令**：在有序集合元素较多时使用ZSCAN命令代替。

### 4.8.3 原子操作

- **使用INCR/DECR命令**：实现原子性的计数器。
- **使用SETNX命令**：实现分布式锁。
- **使用事务**：保证一组命令的原子性。
- **使用Lua脚本**：实现复杂的原子操作。

## 4.9 小结

本章节详细介绍了Redis的常用命令，包括键操作、字符串操作、哈希操作、列表操作、集合操作、有序集合操作以及其他命令。每种命令都包含了详细的描述、示例和时间复杂度分析。

通过本章节的学习，您应该能够熟练使用Redis的各种命令，并掌握命令的使用技巧和注意事项。在实际应用中，您需要根据具体的业务场景选择合适的命令，并注意避免使用可能阻塞Redis的命令。

在接下来的章节中，我们将详细介绍Redis的持久化机制、高可用方案以及在Go语言后端开发中的实际应用。