# Shairport Sync 安装与使用

Shairport Sync 是适用于 Linux 和 FreeBSD 的 AirPlay 音频播放器。它播放从 Apple 设备和 AirPlay 源（例如OwnTone）流式传输的音频。官方代码仓库：https://github.com/mikebrady/shairport-sync

## CentOS

目前 CentOS 没有官方软件源，除了自行编译安装以外，最简单的方式是使用 Docker，只需如下一行命令。其中`Shairport CentOS`是自定义 Airplay 显示的名称，并自动调用系统默认的音频输出。更多用法参见：https://hub.docker.com/r/mikebrady/shairport-sync

```
docker run -d --restart unless-stopped --net host --device /dev/snd mikebrady/shairport-sync -a "Shairport CentOS"
```

此外，需关闭防火墙或开启防火墙以下端口：

```
firewall-cmd --zone=public --add-port=7000/tcp --permanent
firewall-cmd --zone=public --add-port=5353/udp --permanent
firewall-cmd --zone=public --add-port=319-320/udp --permanent
firewall-cmd --zone=public --add-port=32768-60999/udp --permanent
firewall-cmd --zone=public --add-port=32768-60999/tcp --permanent
firewall-cmd --reload
```

## Ubuntu

1. 安装并启动：
```
sudo apt install avahi-daemon
sudo apt install shairport-sync
sudo systemctl enable shairport-sync
sudo systemctl start shairport-sync
```

2. 配置文件

首先用`shairport-sync -h`命令查看音频输出设备名称，在该命令的结果中找到如下信息：
```
hardware output devices:
      "hw:HDMI"
      "hw:PCH"
```

打开配置文件`sudo vi /etc/shairport-sync.conf`，输入以下内容（原内容可以全部删除）并保存、重启`shairport-sync`，其中`output_device`为上一个命令查看到的设备名。
```
general =
{
    name = "Shairport Ubuntu";
    volume_range_db = 100;
};

alsa =
{
    output_device = "hw:PCH";
    mixer_control_name = "PCM";
};
```