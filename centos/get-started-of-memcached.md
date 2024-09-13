# Memcached 服务安装

Memcached分为服务端和客户端（PHP扩展），需分别安装。客户端有Memcache和Memcached之分，两者略有区别，后者是前者的升级版，但Windows下无法使用。在PHP代码中，Memcache使用connect方法连接服务端，Memcached则使用addServer方法。Memcached扩展可与PHP其他扩展一起安装，参见PHP章节。

## 一. 安装服务端

```bash
yum -y install memcached
```

## 二. 查看配置项

```bash
cat /etc/sysconfig/memcached
```

设置开机启动及日常启停

```bash
systemctl enable memcached
systemctl start memcached
systemctl stop memcached
systemctl restart memcached
```

## 三. 命令行连接

需先安装 Telnet：

```bash
yum list | grep telnet
yum install telnet-server.x86_64 telnet.x86_64
```

通过`telnet 127.0.0.1 11211`进入控制台，输入`stats`查看运行数据，`stats reset`清空运行数据。
