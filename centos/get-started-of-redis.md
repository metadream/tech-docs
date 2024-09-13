# Redis 安装与配置

### 安装

```text
yum install redis
```

### 启动

```text
systemctl start redis
systemctl enable redis
```

### 验证

```text
redis-cli ping
```

如果返回 PONG，说明 Redis 安装成功。

### 配置

```text
vi /etc/redis.conf

# 使用密码登录
requirepass foobared

# 允许远程连接（需开启防火墙 6379 端口）
bind 0.0.0.0
```

配置完成后需重启 Redis 服务。

