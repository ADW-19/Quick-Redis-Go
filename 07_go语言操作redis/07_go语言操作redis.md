# 7. Go语言操作Redis

## 7.1 安装Redis客户端库

在Go语言中，常用的Redis客户端库有 `go-redis/redis/v8`。可以使用以下命令安装：

```bash
go get github.com/go-redis/redis/v8
```

## 7.2 基本操作

### 连接Redis

```go
package main

import (
	"context"
	"fmt"

	"github.com/go-redis/redis/v8"
)

func main() {
	ctx := context.Background()
	
	// 创建Redis客户端
	rdb := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "", // 密码，默认为空
		DB:       0,  // 数据库编号，默认为0
	})
	
	// 测试连接
	pong, err := rdb.Ping(ctx).Result()
	if err != nil {
		fmt.Println("连接Redis失败:", err)
		return
	}
	fmt.Println("连接Redis成功:", pong)
	
	// 关闭连接
	defer rdb.Close()
}
```

### 字符串操作

```go
// 设置键值对
err := rdb.Set(ctx, "key", "value", 0).Err()
if err != nil {
	fmt.Println("设置键值对失败:", err)
	return
}

// 获取键对应的值
val, err := rdb.Get(ctx, "key").Result()
if err != nil {
	fmt.Println("获取键值对失败:", err)
	return
}
fmt.Println("key:", val)

// 将键对应的值加1
val, err = rdb.Incr(ctx, "counter").Result()
if err != nil {
	fmt.Println("增加计数器失败:", err)
	return
}
fmt.Println("counter:", val)
```

### 哈希操作

```go
// 设置哈希字段的值
err := rdb.HSet(ctx, "user:1", "name", "张三", "age", 18).Err()
if err != nil {
	fmt.Println("设置哈希字段失败:", err)
	return
}

// 获取哈希字段的值
name, err := rdb.HGet(ctx, "user:1", "name").Result()
if err != nil {
	fmt.Println("获取哈希字段失败:", err)
	return
}
fmt.Println("name:", name)

// 获取哈希的所有字段和值
user, err := rdb.HGetAll(ctx, "user:1").Result()
if err != nil {
	fmt.Println("获取哈希所有字段失败:", err)
	return
}
fmt.Println("user:", user)
```

### 列表操作

```go
// 在列表左侧插入元素
err := rdb.LPush(ctx, "list", "a", "b", "c").Err()
if err != nil {
	fmt.Println("插入列表元素失败:", err)
	return
}

// 获取列表的长度
len, err := rdb.LLen(ctx, "list").Result()
if err != nil {
	fmt.Println("获取列表长度失败:", err)
	return
}
fmt.Println("list length:", len)

// 获取列表指定范围的元素
values, err := rdb.LRange(ctx, "list", 0, -1).Result()
if err != nil {
	fmt.Println("获取列表元素失败:", err)
	return
}
fmt.Println("list values:", values)
```

### 集合操作

```go
// 向集合添加元素
err := rdb.SAdd(ctx, "set", "a", "b", "c").Err()
if err != nil {
	fmt.Println("添加集合元素失败:", err)
	return
}

// 获取集合的所有元素
members, err := rdb.SMembers(ctx, "set").Result()
if err != nil {
	fmt.Println("获取集合元素失败:", err)
	return
}
fmt.Println("set members:", members)

// 判断元素是否在集合中
exists, err := rdb.SIsMember(ctx, "set", "a").Result()
if err != nil {
	fmt.Println("判断元素是否在集合中失败:", err)
	return
}
fmt.Println("a exists in set:", exists)
```

### 有序集合操作

```go
// 向有序集合添加元素
err := rdb.ZAdd(ctx, "zset", &redis.Z{
	Score:  10, 
	Member: "a",
}, &redis.Z{
	Score:  20, 
	Member: "b",
}, &redis.Z{
	Score:  30, 
	Member: "c",
}).Err()
if err != nil {
	fmt.Println("添加有序集合元素失败:", err)
	return
}

// 获取有序集合指定范围的元素
members, err := rdb.ZRange(ctx, "zset", 0, -1).Result()
if err != nil {
	fmt.Println("获取有序集合元素失败:", err)
	return
}
fmt.Println("zset members:", members)

// 获取有序集合指定范围的元素（按分数降序）
members, err = rdb.ZRevRange(ctx, "zset", 0, -1).Result()
if err != nil {
	fmt.Println("获取有序集合元素失败:", err)
	return
}
fmt.Println("zset members (reverse):", members)
```

## 7.3 连接池配置

### 连接池工作原理

连接池管理着与Redis服务器的连接，可以显著提高应用程序的性能。

#### 连接池工作原理图

```
+----------------+                 +----------------+
|   应用程序     |                 |   Redis服务器  |
+----------------+                 +----------------+
        |                                   |
        |    获取连接                       |
        +------------------->              |
        |                   |              |
        |                   v              |
        |           +----------------+     |
        |           |   连接池       |     |
        |           |   (空闲连接)   |     |
        |           +----------------+     |
        |                   |              |
        |                   |  建立连接    |
        |                   +------------->|
        |                   |              |
        |                   |  使用连接    |
        |<------------------|              |
        |                   |              |
        |    释放连接       |              |
        +------------------->              |
        |                   |              |
        |                   v              |
        |           +----------------+     |
        |           |   连接池       |     |
        |           |   (归还连接)   |     |
        |           +----------------+     |
```

### 连接池配置示例

```go
rdb := redis.NewClient(&redis.Options{
	Addr:         "localhost:6379",
	Password:     "",
	DB:           0,
	PoolSize:     10,  // 连接池大小
	MinIdleConns: 5,   // 最小空闲连接数
	MaxRetries:   3,   // 最大重试次数
	DialTimeout:  5 * time.Second,  // 连接超时时间
	ReadTimeout:  3 * time.Second,  // 读取超时时间
	WriteTimeout: 3 * time.Second,  // 写入超时时间
	PoolTimeout:  4 * time.Second,  // 从连接池获取连接的超时时间
})
```

### 连接池参数说明

| 参数 | 说明 | 推荐值 |
|------|------|--------|
| `PoolSize` | 连接池的最大连接数 | 根据并发量设置，通常为CPU核心数的2-4倍 |
| `MinIdleConns` | 最小空闲连接数 | 通常设置为PoolSize的一半 |
| `MaxRetries` | 最大重试次数 | 3-5次 |
| `DialTimeout` | 连接超时时间 | 5秒 |
| `ReadTimeout` | 读取超时时间 | 3秒 |
| `WriteTimeout` | 写入超时时间 | 3秒 |
| `PoolTimeout` | 获取连接超时时间 | 4秒 |

## 7.4 管道操作

管道（Pipeline）允许客户端一次性发送多个命令，减少网络往返时间，提高性能。

### 管道工作原理

#### 管道操作流程图

```
传统方式：
+----------------+     命令1      +----------------+     响应1     +----------------+
|   客户端       | --------------> |   Redis服务器  | <------------ |   客户端       |
+----------------+                 +----------------+                 +----------------+
        |                                                                   |
        |     命令2                                                         |
        +------------------>                                               |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     响应2     +----------------+      |
        |           |   Redis服务器  | <------------ |   客户端       |      |
        |           +----------------+                 +----------------+      |
        |                                                                   |
        |     命令3                                                         |
        +------------------>                                               |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     响应3     +----------------+      |
        |           |   Redis服务器  | <------------ |   客户端       |      |
        |           +----------------+                 +----------------+      |

管道方式：
+----------------+     批量命令    +----------------+     批量响应   +----------------+
|   客户端       | --------------> |   Redis服务器  | <------------ |   客户端       |
+----------------+                 +----------------+                 +----------------+
```

### 管道操作示例

```go
// 创建管道
pipe := rdb.Pipeline()

// 添加多个命令到管道
incr := pipe.Incr(ctx, "pipeline_counter")
pipe.Expire(ctx, "pipeline_counter", time.Hour)

// 执行管道命令
_, err := pipe.Exec(ctx)
if err != nil {
	fmt.Println("执行管道命令失败:", err)
	return
}

// 获取命令结果
fmt.Println("pipeline_counter:", incr.Val())
```

### 管道操作注意事项

- 管道中的命令是原子执行的，但管道本身不是原子操作
- 管道中的命令之间没有依赖关系
- 管道可以显著减少网络往返时间，提高性能

## 7.5 事务操作

Redis事务通过MULTI、EXEC、DISCARD、WATCH等命令实现，保证多个命令的原子性执行。

### 事务工作原理

#### 事务执行流程图

```
+----------------+     WATCH命令   +----------------+     MULTI命令   +----------------+
|   客户端       | --------------> |   Redis服务器  | --------------> |   Redis服务器  |
+----------------+                 +----------------+                 +----------------+
        |                                                                   |
        |     监视键                                                         |
        +------------------>                                               |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     EXEC命令   +----------------+      |
        |           |   检查键是否    | --------------> |   执行事务      |      |
        |           |   被修改       |                 +----------------+      |
        |           +----------------+                                        |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     响应      +----------------+      |
        |           |   返回执行结果  | <------------ |   客户端       |      |
        |           +----------------+                 +----------------+      |
```

### 事务操作示例

```go
// 使用事务实现原子操作
err := rdb.Watch(ctx, func(tx *redis.Tx) error {
	// 获取当前值
	n, err := tx.Get(ctx, "balance").Int()
	if err != nil && err != redis.Nil {
		return err
	}
	
	// 开启事务
	_, err = tx.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
		// 在事务中执行多个命令
		pipe.Set(ctx, "balance", n+100, 0)
		pipe.Set(ctx, "log", "add 100", 0)
		return nil
	})
	
	return err
}, "balance")

if err != nil {
	fmt.Println("事务执行失败:", err)
	return
}
fmt.Println("事务执行成功")
```

### 事务注意事项

- Redis事务不支持回滚，如果某个命令执行失败，其他命令仍会执行
- WATCH命令用于乐观锁，可以监视一个或多个键，如果在事务执行前这些键被修改，事务将不会执行
- 事务中的命令会按顺序执行，中间不会被其他命令插入

## 7.6 发布订阅

Redis发布订阅（Pub/Sub）是一种消息通信模式，发送者（pub）发送消息，订阅者（sub）接收消息。

### 发布订阅工作原理

#### 发布订阅架构图

```
+----------------+                 +----------------+
|   发布者1      |                 |   Redis服务器  |
+----------------+                 +----------------+
        |                                   |
        |     发布消息                       |
        +------------------>               |
        |                   |               |
        |                   v               |
        |           +----------------+     |
        |           |   频道A       |     |
        |           +----------------+     |
        |                   |               |
        |                   | 转发消息      |
        |                   +------------->|
        |                   |               |
        |                   v               |
        |           +----------------+     |
        |           |   订阅者1      |     |
        |           +----------------+     |
        |                   |               |
        |                   v               |
        |           +----------------+     |
        |           |   订阅者2      |     |
        |           +----------------+     |
        |                                   |
+----------------+                         |
|   发布者2      |                         |
+----------------+                         |
        |                                   |
        |     发布消息                       |
        +------------------>               |
```

### 发布订阅示例

```go
// 订阅频道
pubsub := rdb.Subscribe(ctx, "channel1")
defer pubsub.Close()

// 接收消息
ch := pubsub.Channel()
for msg := range ch {
	fmt.Printf("收到消息: %s\n", msg.Payload)
}

// 发布消息
err := rdb.Publish(ctx, "channel1", "Hello, Redis!").Err()
if err != nil {
	fmt.Println("发布消息失败:", err)
	return
}
```

### 发布订阅注意事项

- 发布订阅是实时的，如果订阅者离线，会错过消息
- 发布订阅不支持消息持久化
- 发布订阅适合实时通知场景，不适合可靠消息传递

## 7.7 分布式锁

Redis可以用来实现分布式锁，确保在分布式环境中的互斥访问。

### 分布式锁工作原理

#### 分布式锁流程图

```
+----------------+     请求锁      +----------------+     SETNX命令   +----------------+
|   客户端1      | --------------> |   Redis服务器  | --------------> |   Redis服务器  |
+----------------+                 +----------------+                 +----------------+
        |                                                                   |
        |     获取锁成功                                                     |
        +<------------------                                               |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     执行业务    +----------------+      |
        |           |   执行业务逻辑  | --------------> |   客户端1      |      |
        |           +----------------+                 +----------------+      |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     释放锁    +----------------+      |
        |           |   DEL命令      | --------------> |   Redis服务器  |      |
        |           +----------------+                 +----------------+      |
        |                                                                   |
+----------------+     请求锁      +----------------+     SETNX命令   +----------------+
|   客户端2      | --------------> |   Redis服务器  | --------------> |   Redis服务器  |
+----------------+                 +----------------+                 +----------------+
        |                                                                   |
        |     获取锁失败                                                     |
        +<------------------                                               |
        |                   |                                               |
        |                   v                                               |
        |           +----------------+     等待重试    +----------------+      |
        |           |   等待锁释放    | --------------> |   客户端2      |      |
        |           +----------------+                 +----------------+      |
```

### 分布式锁示例

```go
// 获取分布式锁
func acquireLock(ctx context.Context, rdb *redis.Client, key string, expiration time.Duration) (bool, error) {
	// 使用SET命令的NX选项实现分布式锁
	result, err := rdb.SetNX(ctx, key, "locked", expiration).Result()
	if err != nil {
		return false, err
	}
	return result, nil
}

// 释放分布式锁
func releaseLock(ctx context.Context, rdb *redis.Client, key string) error {
	// 使用Lua脚本确保只有锁的持有者才能释放锁
	script := `
	if redis.call("get", KEYS[1]) == ARGV[1] then
		return redis.call("del", KEYS[1])
	else
		return 0
	end
	`
	_, err := rdb.Eval(ctx, script, []string{key}, "locked").Result()
	return err
}

// 使用分布式锁
func main() {
	ctx := context.Background()
	
	// 获取锁
	acquired, err := acquireLock(ctx, rdb, "my_lock", 10*time.Second)
	if err != nil {
		fmt.Println("获取锁失败:", err)
		return
	}
	
	if !acquired {
		fmt.Println("锁已被占用")
		return
	}
	
	// 确保锁会被释放
	defer releaseLock(ctx, rdb, "my_lock")
	
	// 执行业务逻辑
	fmt.Println("执行业务逻辑...")
	time.Sleep(5 * time.Second)
}
```

### 分布式锁注意事项

- 分布式锁需要设置过期时间，防止死锁
- 释放锁时需要验证锁的持有者，防止误删其他客户端的锁
- 可以使用Redisson等成熟的分布式锁实现