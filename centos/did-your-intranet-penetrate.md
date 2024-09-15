# 内网穿透技术简介

自从搞起ITX主机，就无时不在设法穿透内网。用过一个叫 [Serveo](http://serveo.net)
的神器，号称无需注册无需安装无需公网IP，仅通过系统自带的ssh进行反向代理，确实能用，但即便采用autossh，稳定性也比较差，或许是我打开方式不对。

## 一. Serveo 神器 + SSH

### 1. 基本用法

```bash
# 自动分配二级域名
ssh -R 80:localhost:3000 serveo.net

# 指定二级域名
ssh -R myname:80:localhost:8888 serveo.net
```

### 2. 自定义域名

首先需要一个SSH密钥对，可以使用`ssh-keygen -l`命令生成，输出结果如下：

```text
2048 SHA256:pmc7ZRv7ymCmghUwHoJWEm5ToSTd33ryeDeps5RnfRY no comment (RSA)
```

然后将域名A记录解析至159.89.214.31，同时增加TXT记录，内容为（其中等号右边是密钥指纹）：

```text
authkeyfp=SHA256:pmc7ZRv7ymCmghUwHoJWEm5ToSTd33ryeDeps5RnfRY
```

### 3. 启动SSH

为便于使用，可创建服务脚本：

```text
vi /usr/lib/systemd/system/serveo.service
```

内容如下：

```text
[Unit]
Description=Auto SSH Local Tunnel
After=network-online.target sshd.service

[Service]
Type=simple
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -R customdomain.com:80:localhost:3000 serveo.net
ExecReload=/bin/kill $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

保存后启动：

```text
systemctl enable serveo
systemctl start serveo
```

## 二. 公网服务器 + SSH

如果自己有一台公网服务器，可以按如下步骤进行端口转发穿透内网，理论上应该比
serveo 更稳定和可靠。

### 1. 准备工作

为了实现SSH免登录，先来设置证书连接公网服务器，在内网机执行以下命令生成非对称密钥：

```text
ssh-keygen -t rsa
```

命令完成后会生成几个文件，将其中`~/.ssh/id_rsa.pub`文件中的内容复制出来，添加到公网服务器的`/home/{username}/.ssh/authorized_keys`文件（不存在则新建，有内容则附加），此时执行SSH命令就可以不需要密码。

### 2. 端口转发

SSH命令的大致形式如下：

```text
ssh -fCNR 0.0.0.0:<远程映射端口>:localhost:<本地监听端口> <远程主机用户名>@<远程主机名>
ssh -fCNR 0.0.0.0:7777:localhost:80 root@8.8.8.8
```

参数包括：

```text
-f 登录成功后即转为后台任务执行
-n 重定向stdin为/dev/null，用于配合-f
-q 安静模式，忽视大部分的警告和诊断信息
-C 允许压缩传输数据
-N 不执行远程指令（仅做端口转发）
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口
-T 不需要创建虚拟终端
-p 指定远程主机的SSH端口
```

该命令执行后，通过`netstat`可以查看到公网服务器已经监听`7777`端口。默认情况下，监听的是`127.0.0.1`，即只允许本机访问；如果允许所有IP访问（通常没必要），则需要修改公网服务器的`/etc/ssh/sshd_config`配置文件，将以下行取消注释修改为`yes`，重启`sshd`：

```text
GatewayPorts yes
```

再次执行上述命令可监听`0.0.0.0`。如果SSH以后台模式运行，需要用`ps -ef | grep ssh`找到`pid`再`kill`的方式停止。
看似很复杂，其实仅一条命令，即可实现公网到内网的穿透，只是为了更稳定更便捷更正常地使用，后续还有一些工作要做。

### 3. AutoSSH

AutoSSH可以对SSH发生异常时进行重连，一定程度上保障了转发通道的稳定性。其使用方法和SSH大致相同，只是多加了几个参数，以下是两者的比较：

```text
ssh -fCNR 0.0.0.0:7777:localhost:80 root@8.8.8.8
autossh -M 2222 -fCNR 0.0.0.0:7777:localhost:80 root@8.8.8.8
```

`M`参数指定用于监听SSH隧道是否正常的端口，它会启动两个端口来回发送数据（另一个在指定的基础上+1），但在新版本的AutoSSH中已经不推荐这种用法，改为将其设为0禁用内置监听端口，增加`ServerAliveInterval`和`ServerAliveCountMax`进行心跳检测，完整的命令如下（每隔30秒检测一次，3次失败后断开连接）：

```text
autossh -M 0 -o "ServerAliveInterval=30" -o "ServerAliveCountMax=3" -fCNR 0.0.0.0:7777:localhost:80 root@8.8.8.8
```

### 4. 开机启动

创建文件`/usr/lib/systemd/system/ssh-tunnel.service`，内容如下：

```text
[Unit]
Description=Auto SSH Local Tunnel
After=network-online.target sshd.service

[Service]
Type=simple
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ServerAliveInterval=30" -o "ServerAliveCountMax=3" -CNR 0.0.0.0:7777:localhost:80 root@8.8.8.8
ExecStop=/bin/kill $MAINPID
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

其中`AUTOSSH_GATETIME`的意思是：SSH必须启动多长时间才能被认为是成功的连接，默认值30秒，0表示禁用此行为，并且即使在首次尝试运行SSH失败时，AutoSSH也将重试，在开机运行AutoSSH时，设置为0最为有用。此外，AutoSSH必须为全路径，且需去掉`-f`参数。

然后就是启动：

```text
systemctl enable ssh-tunnel
systemctl start ssh-tunnel
```

### 5. Nginx 反向代理

以上步骤的结果是将公网服务器的`7777`端口映射到了本地内网机器的`80`端口，如果提供网站服务显然不太合适，需要再利用Nginx将公网服务器的`80`或`443`代理到`7777`。如此一来，用户访问公网IP或域名会转到自身的`7777`，再转到内网的`80`。

## 三. 公网服务器 + FRP

[FRP](https://github.com/fatedier/frp)
是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp 协议，为 http 和 https
应用协议提供了额外的能力，且尝试性支持了点对点穿透，但前提也是必须有一个公网IP，例如云服务器IP。以下对通过自定义域名访问内网
web
服务、以及简单的文件访问服务进行部署，更详细的文档参见[这里](https://github.com/fatedier/frp/blob/master/README_zh.md)。

首先根据对应的操作系统及架构，从
[Release](https://github.com/fatedier/frp/releases)
页面下载最新版本的程序，分别放置在具有公网IP的服务器上和处于内网环境的本地机上。

### 1. 服务器端

服务器端修改 frps.ini 文件，设置 http 访问端口为 8000：

```text
[common]
bind_port = 7000
vhost_http_port = 8000
```

为 frps 设置执行权限并启动 frps：

```bash
chmod +x frps
./frps -c ./frps.ini
```

### 2. 本地端

本地端修改 frpc.ini 文件，假设 frps 所在服务器 IP 为 x.x.x.x，local\_port 为本地
web 服务对应的端口, 绑定自定义域名 www.domain.com：

```text
[common]
server_addr = x.x.x.x
server_port = 7000

[web]
type = http
local_port = 80
custom_domains = www.domain.com

[file]
type = tcp
remote_port = 8001
plugin = static_file
plugin_local_path = /home/files
plugin_strip_prefix =
plugin_http_user = test
plugin_http_passwd = test
```

为 frpc 设置执行权限并启动 frpc：

```bash
chmod +x frpc
./frpc -c ./frpc.ini
```

### 3. 域名解析

将自定义域名解析到服务器公网IP，通过浏览器访问
[http://www.domain.com:8000](http://www.domain.com:8000) 即可访问处于内网的 web
服务，[http://x.x.x.x:8001](http://x.x.x.x:8001) 即可访问处于内网的 /home/files
目录。值得一提的是，部署过程中应尽量避免使用非安全端口，同时开通云服务器的安全组限制。

### 4. 开机自启动

将 frp 服务设置为后台启动和开机自启动，服务端和本地操作一致，只是注意 frps 和
frpc 的区别。

```bash
vi /lib/systemd/system/frp.service
```

写入如下内容：

```text
[Unit]
Description=frp server
After=network.target network-online.target syslog.target
Wants=network.target network-online.target

[Service]
Type=simple
ExecStart=/path/frps -c /path/frps.ini

[Install]
WantedBy=multi-user.target
```

保存退出后执行：

```bash
systemctl enable frp
systemctl start frp
```

## 四. 终极大法

然而，FRP
毕竟还是需要一个具有公网IP的服务器，于是继续研究。人说家里的路由器可以做端口映射，但路由器是连在光猫上的，光猫不放话，内网再怎么折腾也无济于事。为了获取光猫的超级权限，又在网上寻找各种破解之术，但没一个管用。

最后，抱着弱弱的态度，我拨打了两个电话。一个给电信客服，说家里需要分配公网IP，客服居然二话没说就答应了，上光猫一查，果然显示的广域网IP和其他方式查到的一样。第二个电话给负责小区宽带的安装师傅，说我需要光猫超级管理员，师傅居然也爽快地给了，只是一再强调改出问题来可别怪他。激动之情难以言表，赶紧登录一看，我去，功能之多，要啥有啥，先设它几个端口映射，立马就穿透，居然连80端口都开放，你说惊不惊喜，神不神奇？！

剩下的，无非是DDNS了，加入定时任务搞定。从此高枕无忧矣，家就是云，云就是家，还要什么腾讯阿里服务器，怕只怕，电信会出幺蛾子，工信部会请喝茶。
