# Debian 12 Bookworm 服务器指南

## 0. 下载与安装

到官网 https://www.debian.org/download 下载网络安装映像，用
[Rufus](https://rufus.ie/zh) 将其刻录成 U
盘启动盘。刻录时，如果是新式的UEFI主板分区类型选择GPT，老式主板选MBR，其他默认即可。

启动电脑后进入 BIOS 选择U盘启动。系统将进入 Debian
安装界面，方便起见可选择图形化安装方式（仅限安装过程）。Debian
并无服务器和客户端之分，只是作为服务器安装的软件略有不同，在安装过程中注意选择。
对于分区，至少一个系统盘、一个EFI引导分区、一个Swap分区（8G内存使用等量交换分区）。

## 1. 必备工具

```
apt install net-tools
apt install vim
```

安装后的vim无法复制，需修改`/usr/share/vim/vim90/defaults.vim`配置文件（vim90是版本，注意区分），找到下面的内容：

```
if has('mouse')
  if &term =~ 'xterm'
    set mouse=a
  else
    set mouse=nvi
  endif
endif
```

将其修改为（多一个减号）：

```
if has('mouse')
  if &term =~ 'xterm'
    set mouse-=a
  else
    set mouse=nvi
  endif
endif
```

## 2. SSH 服务

安装过程中可选择是否安装 SSH Server，如未选择也可后续执行 ‵apt install ssh‵
命令单独安装，然后修改配置文件 ‵/etc/ssh/sshd_config‵。该目录下有 ‵ssh_config‵
和 ‵sshd_config‵
两个文件，前者用于客户端，后者才是服务端的配置。保证以下内容是开启状态（不存在则添加）：

```
PermitRootLogin yes
PasswordAuthentication yes
```

重启 SSH（作为 systemd 服务时 ssh 是 sshd 的别名），并确保开机启动:

```
systemctl restart ssh
systemctl enable ssh
```

## 3. 设置静态IP

`vi /etc/network/interfaces` 打开网络配置文件，将网卡对应信息修改为：

```
auto enp2s0
iface enp2s0 inet static
address 10.0.0.2
netmask 255.255.255.0
gateway 10.0.0.1
dns-nameservers 10.0.0.1
```

重启网络：

```
systemctl restart networking
```

## 4. 挂载硬盘

创建挂载点：

```
mkdir /data
```

用 `blkid` 查看所有已识别的硬盘，复制要挂载的硬盘的 UUID，`vi /etc/fstab`
打开文件系统挂载表，在末尾增加如下内容：

```
UUID=xxxxx /data ext4 defaults 0 0
```

## 5. 环境配置

`vi .bashrc` 打开环境配置文件，在末尾增加如下内容后，执行 `source .bashrc`
使其生效：

```
export PS1="\[\e[35m\][\u@\h:\W]$ \[\e[m\]"
alias ll='ls -lF'
alias la='ls -lAF'
```

## 6. 删除多余用户

Debian 安装时必须创建一个非 root
用户，如果平时用不上可以删掉、同时删除它的家目录：

```
deluser xehu --remove-home
```

## 7. SAMBA

### 7.1 添加组和用户

```
apt install samba
groupadd SAMBA
useradd xehu -M -s /sbin/nologin -g SAMBA
pdbedit -a xehu
```

### 7.2 配置 Samba

配置文件路径`vi /etc/samba/smb.conf`，具体内容略。配置完之后设置目录的所有者权限。

```
chown -R xehu:SAMBA /data
```

### 7.3 启动 Samba

作为服务时，smb 是 smbd 的别名。

```
systemctl restart smb
systemctl enable smb
```

### 7.4 禁用 nmbd

```
systemctl stop nmbd
systemctl disable nmbd
```

## 8. MPD

安装声卡应用工具及MPD：

```
apt install alsa-utils
apt install mpd
```

`vi /etc/mpd.conf` 打开配置文件，修改或确保以下内容处于非注释状态：

```
bind_to_address "0.0.0.0"
music_directory "/home/music"
audio_output {
  type  "alsa"
  name  "ALSA Device"
}
```

`alsamixer` 命令打开音频设置器，将 `Master` 音量调至最大，启动 MPD
并设为开机自启：

```
systemctl restart mpd
systemctl enable mpd
```

## 9. 设置防火墙

Debian 12 自带的防火墙工具是 ‵nftables‵，`vi /etc/nftables.conf`
打开配置文件并将其中 `chain input` 部分修改为如下内容：

```
chain input {
	type filter hook input priority 0; policy drop;

	# established/related connections
	ct state established,related accept

	# invalid connections
	ct state invalid drop

	# loopback interface
	iif lo accept

	# SSH (port 22)
	tcp dport ssh accept

	# HTTP (ports 80 & 443)
	tcp dport { http, https } accept

	# SAMBA (ports 445)
	tcp dport 445 accept

	# MPD (ports 6600)
	tcp dport 6600 accept
}
```

重启并查看：

```
systemctl restart nftables
nft list ruleset
```

## 10. Docker

参见 https://docs.docker.com/engine/install/debian

## 11. Shairport Sync

### 11.1 从软件源安装

Shairport Sync 是 Linux 版本的 Airplay
应用，可直接从软件源下载安装，但此方式在我的机器上无法播放声音，推荐采用 Docker
方式。

```
apt install shairport-sync
```

用 `shairport-sync -h` 命令查看输出设备名，找到如下行：

```
hardware output devices:
      "hw:PCH"
```

`vi /etc/shairport-sync.conf` 打开配置文件，修改如下内容：

```
general =
{
    name = "%H Shairport";
    volume_range_db = 100;
};

alsa =
{
    output_device = "hw:PCH";
    mixer_control_name = "PCM";
};
```

启动和自启动：

```
systemctl restart shairport-sync
systemctl enable shairport-sync
```

### 11.2 Docker 方式（推荐）

```
docker run -d --restart unless-stopped --net host --device /dev/snd mikebrady/shairport-sync -a "Datacenter Shairport"
```

## 12 连接无线网络

安装以下两个无线网络工具包，前者是替代iwconfig的应用程序，后者是设置WPA密码的应用程序。

```
apt install iw wpa_supplicant
```

用`iw dev`或`ifconfig -a`查看无线网卡名称，假设为`wlp0s20f3`。此时应该未连接任何无线网络，执行以下命令启动无线网卡。

```
ifconfig wlp0s20f3 up
```

执行以下命令搜索可用的无线网络，并连接某个SSID（无密码的情况）：

```
iw wlp0s20f3 scan | grep SSID
iw wlp0s20f3 connect -w <SSID>
```

对于通常WPA/WPA2型密码，需改用以下连接方式：

```
wpa_supplicant -B -i wlp0s20f3 -c <(wpa_passphrase "<SSID>" "<PASSWORD>")
```

如果需要重启后自动连接指定SSID，需编辑`/etc/wpa_supplicant.conf`配置文件，或执行以下命令写入配置：

```
wpa_passphrase <SSID> >>/etc/wpa_supplicant.conf
# reading passphrase from stdin
```

然后改用以下命令进行连接：

```
wpa_supplicant -B -i wlp0s20f3 -c /etc/wpa_supplicant.conf
```

释放并启动DHCP自动分配IP地址：

```
dhclient -r wlp0s20f3
dhclient wlp0s20f3
```

查看IP地址和连接状态：

```
ifconfig wlp0s20f3
iw wlp0s20f3 link
```

断开连接和关闭网卡：

```
iw wlp0s20f3 disconnect
ifconfig wlp0s20f3 down
```

## 13. 创建有线中继+无线热点

这个小破功能足足折腾了我一天。基本思路是先创建一个网桥，将有线网和无线网连接起来，然后在无线网卡处创建WIFI热点，并通过DHCP动态分配地址。网桥模式下，被分配地址的客户端处于同一子网。

### 13.1 创建并连接网桥

据说新版本的Linux用systemd-networkd代替了networking来管理网络，我在networking（其配置文件为`/etc/network/interfaces`）上踩坑无数也没搞出来，最后万能的谷歌告诉我systemd-networkd更牛逼，但也更麻烦。首先进入`/etc/systemd/network/`目录，创建三个文件，文件名及内容分别如下，其作用依次是创建网桥、配置网桥、连接网桥（将名为`br0`的网桥连接到名为`enp1s0`的有线网卡设备）：

```
# br0.netdev
[NetDev]
Name=br0
Kind=bridge
```

```
# br0.network
[Match]
Name=br0

[Network]
Address=10.0.0.2/24
Gateway=10.0.0.1
DNS=10.0.0.1
#DHCPServer=true
```

```
# br0-enp.network
[Match]
Name=enp1s0

[Network]
Bridge=br0
```

执行命令`systemctl restart systemd-networkd`重启网络。此时如果连接服务器，是网桥作为虚拟设备提供网络，真正的物理网卡可以没有IP
地址。`systemd-networkd`无需考虑网桥、网卡、连接的启动时机、顺序等问题，在`networking`上因为这些花了大量时间；同时自带DHCP服务，无需安装`udhcpd`或`dnsmasq`等第三方服务。

### 13.2 创建WIFI热点

`systemd-networkd`目前还不支持无线网络管理，需配合`hostapd`使用。`apt install hostapd`安装之后，打开配置文件`/etc/hostapd/hostapd.conf`，主要参数修改如下：

```
# /etc/hostapd/hostapd.conf
# 无线网卡名称，可通过ifconfig查看
interface=wlp0s20f3
# 上一步创建的网桥名称（此两项设置即表示将“以太网-网桥-局域网”关联起来）
bridge=br0
# Wifi显示名称
ssid=metacloud
# 接口驱动类型 (hostap/wired/none/nl80211/bsd)
driver=nl80211
# 国家代码 (ISO/IEC 3166-1)
country_code=CN
# 网络模式 (a = IEEE 802.11a (5 GHz), b = IEEE 802.11b (2.4 GHz)
hw_mode=g
# 通道数
channel=6
# 最大连接数
max_num_sta=30
# 不限制MAC地址
macaddr_acl=0
# 身份验证算法: 1=wpa, 2=wep, 3=both
auth_algs=1
# WPA版本: 0=WPA, 1=WPA2, 2=both
wpa=3
# 加密算法：CCMP=AES（推荐）、TKIP
wpa_pairwise=CCMP
# 密钥管理算法：WPA-PSK（推荐）、WAP-EAP
wpa_key_mgmt=WPA-PSK
# Wifi连接密码
wpa_passphrase=12345678
# 启用 802.11n
ieee80211n=1
# 启用 WMM
wmm_enabled=1
# 日志输出级别
logger_stdout=-1
logger_stdout_level=2
```

执行`systemctl restart hostapd`重启，用手机搜索上面的SSID无线网络进行测试。最后，别忘了将`systemd-networkd`、‵hostapd`设为开机启动，并禁用`networking`服务。

## 99. 常用命令

```
# 查看服务及其状态
service --status-all

# 统计当前目录下所有文件的大小
du -s

# 统计当前目录下文件的个数（包括子目录）
ls -lR | grep "^-" | wc -l

# 统计当前目录下文件夹的个数（包括子目录）
ls -lR | grep "^d" | wc -l
```
