# Transmission 安装与使用

[Transmission](https://transmissionbt.com) 是一种 BitTorrent 客户端，以跨平台的服务端和简洁的用户界面著称。

## 一. 安装及启停

```bash
yum install transmission transmission-daemon
systemctl start transmission-daemon
systemctl enable transmission-daemon
```

启动后即可通过浏览器 `http://IP:9091` 访问和使用 transmission 服务。

## 二. 修改配置文件

修改配置文件之前必须先停止 transmission-daemon，否则不生效，修改完成后再启动。

```bash
systemctl stop transmission-daemon
vi /var/lib/transmission/.config/transmission-daemon/settings.json
systemctl start transmission-daemon
```

配置项说明如下：

```javascript
{
    // 是否启用时段限速，默认关闭
    "alt-speed-enabled": false,

    // 时段限速下载最大值，默认50（KB/s）
    "alt-speed-down": 50,

    // 时段限速开始时间
    "alt-speed-time-begin": 540,

    // 时段限速结束时间
    "alt-speed-time-end": 1020,

    // 是否启用时段限速日期，默认关闭
    "alt-speed-time-enabled": false,

    // 时段限速日期（星期几），127表示每天
    "alt-speed-time-day": 127,

    // 时段限速值，默认50（KB/s）
    "alt-speed-up": 50,

    // IPv4地址绑定
    "bind-address-ipv4": "0.0.0.0",

    // IPv6地址绑定
    "bind-address-ipv6": "::",

    // 是否启用黑名单，默认关闭
    "blocklist-enabled": false,

    // 黑名单列表
    "blocklist-url": "http://www.example.com/blocklist",

    // 缓存大小（MB），建议设置为内存大小的1/6~1/4
    "cache-size-mb": 4,

    // 是否启用DHT网络，默认启用
    "dht-enabled": true,

    // 下载完成的保存路径
    "download-dir": "/mnt/usb/STATION/Download",

    // 是否启用下载队列，默认启用
    "download-queue-enabled": true,

    // 下载队列数，默认5，即最多同时下载5个任务
    "download-queue-size": 5,

    // 0：不加密，1：优先加密（默认），2：必须加密
    "encryption": 1,

    // 是否启用空闲时间停止做种，默认关闭
    "idle-seeding-limit-enabled": false,

    // 空闲多长时间后停止做种，默认30（分钟）
    "idle-seeding-limit": 30,

    // 是否启用未下载完成的保存路径，默认关闭
    "incomplete-dir-enabled": false,

    // 未下载完成的保存路径
    "incomplete-dir": "/var/lib/transmission/Downloads",

    // 是否启用LDP（用于在本地网络寻找节点）
    "lpd-enabled": false,

    // 日志消息等级，默认1
    "message-level": 1,
    "peer-congestion-algorithm": "",
    "peer-id-ttl-hours": 6,
    "peer-limit-global": 200,
    "peer-limit-per-torrent": 50,
    "peer-port": 51413,
    "peer-port-random-high": 65535,
    "peer-port-random-low": 49152,
    "peer-port-random-on-start": false,
    "peer-socket-tos": "default",
    "pex-enabled": true,
    "port-forwarding-enabled": true,

    // 预分配文件磁盘空间，0：关闭，1：快速（默认），2：完全
    "preallocation": 1,
    "prefetch-enabled": true,
    "queue-stalled-enabled": true,
    "queue-stalled-minutes": 30,

    // 是否启用上传下载比例，默认关闭
    "ratio-limit-enabled": false,

    // 上传下载比例达到多少停止做种
    "ratio-limit": 2,

    // 是否启用在未完成的文件名后添加后缀，默认开启
    "rename-partial-files": true,

    // 是否开启远程连接，默认启用
    "rpc-enabled": true,

    // 是否启用远程连接授权验证，默认关闭
    "rpc-authentication-required": false,

    // 远程连接地址绑定，默认0.0.0.0，表示任何地址都能访问
    "rpc-bind-address": "0.0.0.0",

    // 是否启用白名单，默认启用
    "rpc-host-whitelist-enabled": true,

    // 白名单列表，默认空
    "rpc-host-whitelist": "",

    // 远程连接用户名，默认空
    "rpc-username": "",

    // 远程连接密码，修改后会自动加密
    "rpc-password": "{73e73ac09d844dd25fa129ba88b6f996bb980501Q8JgkDQV",

    // 网页服务端口
    "rpc-port": 9091,

    // 远程接口路径
    "rpc-url": "/transmission/",

    // 是否启用白名单，默认关闭，启用后只有白名单地址才能远程连接
    "rpc-whitelist-enabled": false,

    // 白名单列表，默认127.0.0.1
    "rpc-whitelist": "127.0.0.1",

    "scrape-paused-torrents-enabled": true,

    // 是否在下载完成后执行脚本，默认关闭
    "script-torrent-done-enabled": false,

    // 下载完成后执行脚本的路径
    "script-torrent-done-filename": "",

    "seed-queue-enabled": false,
    "seed-queue-size": 10,

    // 是否启用下载限速，默认关闭
    "speed-limit-down-enabled": true,

    // 下载速度限制，默认100（KB/s）
    "speed-limit-down": 4096,

    // 是否启用上传限速，默认关闭
    "speed-limit-up-enabled": true,

    // 上传速度限制，默认100（KB/s）
    "speed-limit-up": 0,

    // 是否添加种子文件后自动开始，默认是
    "start-added-torrents": true,

    // 文件权限掩码：18相当于755权限，0相当于777权限
    "umask": 18,

    // 每个种子上传连接数，默认14
    "upload-slots-per-torrent": 14,

    // 是否启用UTP传输，默认启用
    "utp-enabled": true,

    // 是否启用自动监控种子目录，该选项需手动添加
    "watch-dir-enabled": true,

    // 自动监控种子目录路径，该选项需手动添加
    "watch-dir": "/mnt/usb/STATION/Torrents",

    // 是否自动删除监控目录的种子文件，默认否
    "trash-original-torrent-files": true
}
```

