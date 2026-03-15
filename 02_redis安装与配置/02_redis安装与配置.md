# 2. Redis安装与配置

## 2.1 Windows系统安装

### 2.1.1 下载Redis

1. 访问Redis官方网站（https://redis.io/download）。
2. 滚动到页面底部，找到Windows版本的下载链接。
3. 点击下载最新的Windows版本Redis压缩包（通常是.zip格式）。

### 2.1.2 安装Redis

1. 解压下载的压缩包到指定目录，例如 `D:\Redis`。
2. 打开解压后的目录，可以看到以下文件：
   - `redis-server.exe`：Redis服务器程序
   - `redis-cli.exe`：Redis命令行客户端
   - `redis.windows.conf`：Windows版本的配置文件
   - `redis-benchmark.exe`：Redis性能测试工具
   - `redis-check-aof.exe`：AOF文件检查工具
   - `redis-check-rdb.exe`：RDB文件检查工具

### 2.1.3 启动Redis服务

1. 打开命令提示符（以管理员身份运行）。
2. 进入Redis目录，执行以下命令启动Redis服务：
   ```bash
   redis-server.exe redis.windows.conf
   ```
3. 看到以下输出，表示Redis服务启动成功：
   ```
   [8640] 14 Mar 22:43:28.123 # Server started, Redis version 7.0.11
   [8640] 14 Mar 22:43:28.123 * The server is now ready to accept connections on port 6379
   ```

### 2.1.4 连接Redis服务

1. 另开一个命令提示符，进入Redis目录。
2. 执行以下命令连接到Redis服务：
   ```bash
   redis-cli.exe
   ```
3. 看到以下输出，表示连接成功：
   ```
   127.0.0.1:6379>
   ```
4. 输入 `ping` 命令，返回 `PONG` 表示服务正常运行：
   ```
   127.0.0.1:6379> ping
   PONG
   ```

### 2.1.5 配置Redis为Windows服务

为了方便管理，可以将Redis配置为Windows服务，这样它会在系统启动时自动运行。

1. 打开命令提示符（以管理员身份运行）。
2. 进入Redis目录，执行以下命令安装Redis服务：
   ```bash
   redis-server.exe --service-install redis.windows.conf --loglevel verbose
   ```
3. 看到以下输出，表示服务安装成功：
   ```
   [8640] 14 Mar 22:43:28.123 # Redis successfully installed as a service.
   ```
4. 启动Redis服务：
   ```bash
   redis-server.exe --service-start
   ```
5. 停止Redis服务：
   ```bash
   redis-server.exe --service-stop
   ```
6. 卸载Redis服务：
   ```bash
   redis-server.exe --service-uninstall
   ```

## 2.2 Linux系统安装

### 2.2.1 使用包管理器安装

#### Ubuntu/Debian系统

1. 更新软件包列表：
   ```bash
   sudo apt update
   ```
2. 安装Redis服务器：
   ```bash
   sudo apt install redis-server
   ```
3. 验证Redis服务是否安装成功：
   ```bash
   redis-cli ping
   ```
   返回 `PONG` 表示服务正常运行。

#### CentOS/RHEL系统

1. 安装EPEL仓库（如果尚未安装）：
   ```bash
   sudo yum install epel-release
   ```
2. 安装Redis服务器：
   ```bash
   sudo yum install redis
   ```
3. 启动Redis服务：
   ```bash
   sudo systemctl start redis
   ```
4. 设置Redis服务开机自启：
   ```bash
   sudo systemctl enable redis
   ```
5. 验证Redis服务是否运行：
   ```bash
   redis-cli ping
   ```
   返回 `PONG` 表示服务正常运行。

### 2.2.2 从源码编译安装

如果需要安装特定版本的Redis，可以从源码编译安装。

1. 安装依赖项：
   ```bash
   # Ubuntu/Debian
   sudo apt install build-essential tcl
   
   # CentOS/RHEL
   sudo yum install gcc make tcl
   ```
2. 下载Redis源码：
   ```bash
   wget https://download.redis.io/releases/redis-7.0.11.tar.gz
   ```
3. 解压源码包：
   ```bash
   tar xzf redis-7.0.11.tar.gz
   ```
4. 进入源码目录：
   ```bash
   cd redis-7.0.11
   ```
5. 编译Redis：
   ```bash
   make
   ```
6. 测试编译结果：
   ```bash
   make test
   ```
7. 安装Redis：
   ```bash
   sudo make install
   ```
8. 配置Redis：
   ```bash
   sudo mkdir /etc/redis
   sudo cp redis.conf /etc/redis/
   ```
9. 创建Redis服务文件：
   ```bash
   sudo nano /etc/systemd/system/redis.service
   ```
   写入以下内容：
   ```
   [Unit]
   Description=Redis In-Memory Data Store
   After=network.target

   [Service]
   User=redis
   Group=redis
   ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
   ExecStop=/usr/local/bin/redis-cli shutdown
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```
10. 创建redis用户和组：
    ```bash
    sudo adduser --system --group --no-create-home redis
    sudo mkdir /var/lib/redis
    sudo chown redis:redis /var/lib/redis
    sudo chmod 770 /var/lib/redis
    ```
11. 启动Redis服务：
    ```bash
    sudo systemctl start redis
    ```
12. 设置Redis服务开机自启：
    ```bash
    sudo systemctl enable redis
    ```

### 2.2.3 服务管理

#### 使用systemctl管理Redis服务

- 启动Redis服务：
  ```bash
  sudo systemctl start redis
  ```
- 停止Redis服务：
  ```bash
  sudo systemctl stop redis
  ```
- 重启Redis服务：
  ```bash
  sudo systemctl restart redis
  ```
- 查看Redis服务状态：
  ```bash
  sudo systemctl status redis
  ```
- 设置Redis服务开机自启：
  ```bash
  sudo systemctl enable redis
  ```
- 禁用Redis服务开机自启：
  ```bash
  sudo systemctl disable redis
  ```

## 2.3 基本配置

### 2.3.1 配置文件位置

Redis的配置文件通常位于以下位置：
- Windows系统：`redis.windows.conf`（与Redis可执行文件在同一目录）
- Ubuntu/Debian系统：`/etc/redis/redis.conf`
- CentOS/RHEL系统：`/etc/redis.conf`

### 2.3.2 常用配置项

#### 网络配置

- `bind`：绑定的IP地址，默认为127.0.0.1，只允许本地访问。如果需要远程访问，可以设置为0.0.0.0。
  ```
  bind 127.0.0.1
  ```

- `port`：监听的端口，默认为6379。
  ```
  port 6379
  ```

- `tcp-backlog`：TCP连接队列长度，默认为511。
  ```
  tcp-backlog 511
  ```

- `timeout`：客户端连接超时时间（秒），默认为0，表示无超时。
  ```
  timeout 0
  ```

- `tcp-keepalive`：TCP keepalive时间（秒），默认为300。
  ```
  tcp-keepalive 300
  ```

#### 安全配置

- `requirepass`：设置Redis密码，默认为空。
  ```
  requirepass yourpassword
  ```

- `rename-command`：重命名危险命令，提高安全性。
  ```
  rename-command FLUSHALL ""
  rename-command FLUSHDB ""
  rename-command DEL ""
  ```

#### 内存配置

- `maxmemory`：最大内存使用量，默认无限制。
  ```
  maxmemory 2gb
  ```

- `maxmemory-policy`：内存不足时的淘汰策略，默认为noeviction。
  可选值：
  - `noeviction`：不淘汰任何键，拒绝写入操作
  - `allkeys-lru`：从所有键中淘汰最近最少使用的键
  - `volatile-lru`：从设置了过期时间的键中淘汰最近最少使用的键
  - `allkeys-random`：从所有键中随机淘汰
  - `volatile-random`：从设置了过期时间的键中随机淘汰
  - `volatile-ttl`：从设置了过期时间的键中淘汰剩余TTL最小的键
  ```
  maxmemory-policy volatile-lru
  ```

#### 持久化配置

- `save`：指定在多少秒内有多少次修改后，执行一次RDB快照。
  ```
  save 900 1
  save 300 10
  save 60 10000
  ```
  表示：
  - 900秒内有1次修改
  - 300秒内有10次修改
  - 60秒内有10000次修改
  满足任意条件都会执行RDB快照。

- `dir`：指定RDB和AOF文件的存储目录，默认为`./`。
  ```
  dir /var/lib/redis
  ```

- `dbfilename`：指定RDB文件的名称，默认为`dump.rdb`。
  ```
  dbfilename dump.rdb
  ```

- `appendonly`：是否启用AOF持久化，默认为no。
  ```
  appendonly yes
  ```

- `appendfilename`：指定AOF文件的名称，默认为`appendonly.aof`。
  ```
  appendfilename "appendonly.aof"
  ```

- `appendfsync`：指定AOF文件的同步策略，默认为everysec。
  可选值：
  - `always`：每次写入都同步，最安全但性能最差
  - `everysec`：每秒同步一次，平衡安全性和性能
  - `no`：由操作系统决定何时同步，性能最好但最不安全
  ```
  appendfsync everysec
  ```

- `aof-use-rdb-preamble`：是否启用混合持久化，默认为yes（Redis 4.0+）。
  ```
  aof-use-rdb-preamble yes
  ```

#### 主从复制配置

- `replicaof`：设置从服务器的主服务器地址和端口。
  ```
  replicaof 192.168.1.100 6379
  ```

- `masterauth`：设置主服务器的密码（如果主服务器设置了密码）。
  ```
  masterauth yourpassword
  ```

- `replica-serve-stale-data`：当从服务器与主服务器断开连接时，是否继续提供服务，默认为yes。
  ```
  replica-serve-stale-data yes
  ```

- `replica-read-only`：设置从服务器是否为只读模式，默认为yes。
  ```
  replica-read-only yes
  ```

#### 集群配置

- `cluster-enabled`：是否启用集群模式，默认为no。
  ```
  cluster-enabled yes
  ```

- `cluster-config-file`：指定集群配置文件的名称，默认为`nodes.conf`。
  ```
  cluster-config-file nodes.conf
  ```

- `cluster-node-timeout`：集群节点超时时间（毫秒），默认为15000。
  ```
  cluster-node-timeout 15000
  ```

- `cluster-replica-validity-factor`：从节点选举主节点的有效性因子，默认为10。
  ```
  cluster-replica-validity-factor 10
  ```

- `cluster-migration-barrier`：集群迁移屏障，默认为1。
  ```
  cluster-migration-barrier 1
  ```

- `cluster-require-full-coverage`：是否要求集群全覆盖，默认为yes。
  ```
  cluster-require-full-coverage yes
  ```

### 2.3.3 配置文件管理

#### 编辑配置文件

使用文本编辑器编辑Redis配置文件，例如：

- Windows系统：使用记事本或Notepad++编辑`redis.windows.conf`。
- Linux系统：使用vim或nano编辑配置文件，例如：
  ```bash
  sudo vim /etc/redis/redis.conf
  ```

#### 重新加载配置

修改配置文件后，需要重启Redis服务或使用`CONFIG REWRITE`命令重新加载配置：

1. 重启Redis服务：
   ```bash
   sudo systemctl restart redis
   ```

2. 使用`CONFIG REWRITE`命令重新加载配置：
   ```bash
   redis-cli
   CONFIG REWRITE
   ```

## 2.4 环境变量配置

### 2.4.1 设置Redis环境变量

为了方便在任意目录执行Redis命令，可以将Redis的可执行文件目录添加到系统环境变量中。

#### Windows系统

1. 右键点击「此电脑」→「属性」→「高级系统设置」→「环境变量」。
2. 在「系统变量」中找到「Path」变量，点击「编辑」。
3. 点击「新建」，输入Redis可执行文件的目录路径，例如 `D:\Redis`。
4. 点击「确定」保存修改。
5. 打开新的命令提示符，输入 `redis-cli` 测试是否可以在任意目录执行Redis命令。

#### Linux系统

1. 编辑`~/.bashrc`文件：
   ```bash
   vim ~/.bashrc
   ```
2. 在文件末尾添加以下内容：
   ```bash
   export PATH=$PATH:/usr/local/bin
   ```
3. 保存并退出编辑器。
4. 执行以下命令使环境变量生效：
   ```bash
   source ~/.bashrc
   ```
5. 输入 `redis-cli` 测试是否可以在任意目录执行Redis命令。

## 2.5 常见问题与解决方案

### 2.5.1 端口占用

**问题**：启动Redis服务时提示端口6379已被占用。

**解决方案**：
1. 查看占用端口6379的进程：
   ```bash
   # Windows
   netstat -ano | findstr :6379
   
   # Linux
   sudo lsof -i :6379
   ```
2. 终止占用端口的进程：
   ```bash
   # Windows
   taskkill /PID <进程ID> /F
   
   # Linux
   sudo kill <进程ID>
   ```
3. 重新启动Redis服务。

### 2.5.2 远程连接失败

**问题**：无法从远程主机连接到Redis服务。

**解决方案**：
1. 检查Redis配置文件中的`bind`配置项，确保已设置为0.0.0.0或允许远程访问的IP地址。
2. 检查防火墙是否允许端口6379的访问：
   ```bash
   # Windows
   打开「控制面板」→「系统和安全」→「Windows Defender防火墙」→「高级设置」→「入站规则」，添加允许端口6379的规则。
   
   # Linux
   sudo ufw allow 6379
   # 或
   sudo firewall-cmd --add-port=6379/tcp --permanent
   sudo firewall-cmd --reload
   ```
3. 检查Redis是否设置了密码，如果设置了密码，连接时需要提供密码：
   ```bash
   redis-cli -h <Redis服务器IP> -p 6379 -a <密码>
   ```

### 2.5.3 内存不足

**问题**：Redis服务启动失败，提示内存不足。

**解决方案**：
1. 检查系统内存使用情况：
   ```bash
   # Windows
   tasklist | sort /+5
   
   # Linux
   free -h
   ```
2. 调整Redis配置文件中的`maxmemory`配置项，设置合理的内存限制。
3. 选择合适的内存淘汰策略，如`volatile-lru`或`allkeys-lru`。

### 2.5.4 持久化文件过大

**问题**：Redis的RDB或AOF文件过大，占用过多磁盘空间。

**解决方案**：
1. 定期备份并清理持久化文件。
2. 调整RDB快照的触发条件，减少快照频率。
3. 启用AOF重写，减少AOF文件大小：
   ```bash
   redis-cli
   BGREWRITEAOF
   ```
4. 考虑使用混合持久化模式，结合RDB和AOF的优点。

## 2.6 小结

本章节详细介绍了Redis在Windows和Linux系统上的安装方法，以及Redis的基本配置和常见问题解决方案。通过本章节的学习，您应该能够：

- 在Windows系统上安装和配置Redis
- 在Linux系统上使用包管理器或源码编译安装Redis
- 理解并配置Redis的核心配置项
- 管理Redis服务的启动、停止和重启
- 解决Redis安装和配置过程中遇到的常见问题

在接下来的章节中，我们将详细介绍Redis的数据结构、常用命令、持久化机制、高可用方案以及在Go语言后端开发中的实际应用。