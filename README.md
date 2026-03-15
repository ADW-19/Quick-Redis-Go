# Quick-redis-Go

![Redis](https://img.shields.io/badge/Redis-6.0%2B-blue.svg)
![Go](https://img.shields.io/badge/Go-1.16%2B-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

Quick-redis-Go是一个面向Go语言后端开发人员的Redis基础知识手册及专业业务实战指南。本项目旨在帮助新入行的开发人员快速掌握Redis的核心概念、基本操作以及在实际业务场景中的应用。


## 📚 项目结构

```
redis/
├── 01_redis基础介绍/          # Redis基础知识
├── 02_redis安装与配置/        # Redis安装和配置
├── 03_redis数据结构/          # Redis数据结构
├── 04_redis常用命令/          # Redis常用命令
├── 05_redis持久化机制/        # Redis持久化
├── 06_redis高可用方案/        # Redis高可用
├── 07_go语言操作redis/        # Go语言Redis客户端
├── 08_实际业务场景实战/       # 实际业务场景
│   ├── 8.1 缓存设计.md        # 缓存设计
│   ├── 8.2 分布式锁.md        # 分布式锁
│   ├── 8.3 计数器.md          # 计数器
│   ├── 8.4 排行榜.md          # 排行榜
│   ├── 8.5 消息队列.md        # 消息队列
│   ├── 8.6 验证码系统.md      # 验证码系统
│   ├── 8.7 秒杀系统.md        # 秒杀系统
│   ├── 8.8 千万级数据定时批量更新.md  # 批量更新
│   ├── 8.9 缓存查询数据.md    # 缓存查询
│   ├── 8.10 限流系统.md       # 限流系统
│   └── 8.11 会话管理.md       # 会话管理
└── README.md                 # 项目说明
```

## 🎯 适用人群

- 新入行的开发人员
- 想学习Redis的Go语言后端开发者
- 需要在实际业务中使用Redis的工程师
- 准备面试的技术人员

## 📖 内容概览

### 基础篇
- **Redis基础介绍**：Redis的核心概念、特点和应用场景
- **Redis安装与配置**：Redis的安装、配置和启动
- **Redis数据结构**：String、Hash、List、Set、Sorted Set等数据结构
- **Redis常用命令**：各种数据结构的常用命令和使用方法
- **Redis持久化机制**：RDB、AOF和混合持久化
- **Redis高可用方案**：主从复制、Sentinel和Cluster模式

### 实战篇
- **Go语言操作Redis**：使用go-redis/redis库操作Redis
- **缓存设计**：多级缓存、缓存穿透、缓存击穿、缓存雪崩
- **分布式锁**：基于Redis的分布式锁实现
- **计数器**：原子计数、限流计数等
- **排行榜**：基于Sorted Set的排行榜实现
- **消息队列**：基于List的消息队列实现
- **验证码系统**：安全的验证码生成和验证
- **秒杀系统**：高并发秒杀解决方案
- **千万级数据定时批量更新**：分布式批量处理
- **缓存查询数据**：优化查询性能
- **限流系统**：滑动窗口、令牌桶算法
- **会话管理**：分布式会话共享

## 🚀 快速开始

### 环境要求
- Go 1.16+
- Redis 6.0+

### 安装Redis

#### Linux
```bash
sudo apt update
sudo apt install redis-server
sudo systemctl start redis-server
```

#### macOS
```bash
brew install redis
brew services start redis
```

#### Windows
下载Redis安装包并运行安装程序：[Redis Windows](https://github.com/microsoftarchive/redis/releases)

### 安装Go Redis客户端

```bash
go get github.com/go-redis/redis/v8
```

## 🔧 使用示例

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
		Addr: "localhost:6379",
	})
	
	// 测试连接
	pong, err := rdb.Ping(ctx).Result()
	if err != nil {
		fmt.Printf("连接失败: %v\n", err)
		return
	}
	fmt.Printf("连接成功: %s\n", pong)
	
	// 设置键值对
	err = rdb.Set(ctx, "key", "value", 0).Err()
	if err != nil {
		fmt.Printf("设置失败: %v\n", err)
		return
	}
	fmt.Println("设置成功")
	
	// 获取值
	val, err := rdb.Get(ctx, "key").Result()
	if err != nil {
		fmt.Printf("获取失败: %v\n", err)
		return
	}
	fmt.Printf("获取成功: %s\n", val)
}
```

## 📝 贡献指南

1. Fork本仓库
2. 创建 feature 分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add some amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 打开Pull Request


## 🙏 致谢

- Redis官方文档
- go-redis/redis库
- 所有为Redis生态系统做出贡献的开发者


---

**Happy Coding!** 🎉
