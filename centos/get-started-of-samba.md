# Samba 安装与使用

Samba 服务可以实现 Linux 和 Windows 之间的数据共享。

## 一. 安装

```bash
yum -y install samba
```

## 二. 创建用户

在创建Samba用户之前，必须先创建系统用户，再由系统用户转为Samba用户。此例中，用户组为SAMBA，用户名为admin。

```bash
groupadd SAMBA
useradd admin -b /usr/home -s /sbin/nologin -g SAMBA
pdbedit -a admin
```

其他相关命令如下：

```bash
pdbedit -L        # 查看Samba用户
pdbedit -x admin  # 删除Samba用户
cat /etc/passwd   # 查看系统用户
userdel -r admin  # 删除系统用户
```

## 三. 创建共享目录

```bash
mkdir /mnt/nas/MEDIA
chown -R admin:SAMBA /mnt/nas/MEDIA
# 如果该目录允许所有人读写，还需设置777权限
chmod 777 /mnt/nas/MEDIA
```

## 四. 配置文件

备份并打开 Samba 配置文件：

```bash
cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
vi /etc/samba/smb.conf
```

文件中`[global]`部分为全局设置；`[homes]`部分为用户目录，当用户登录到 samba 时实际上是进入到了该用户的家目录，共享名会由 homes 变成用户自己的标识，对于单纯的文件共享来说，该部分可以注释掉；`[printers]`部分为共享打印机设置，通常也可以忽略。

security 字段有以下四种选项：

* share：用户不需要账户及密码即可登陆samba服务器（新版本中已弃用）
* user：由samba服务器负责验证账户及密码（默认）
* server：由另一台windows或samba服务器负责验证账户及密码
* domain：指定windows域控制服务器来验证账户及密码。

最终 smb.conf 的完整内容大致如下：

```text
[global]
workgroup = SAMBA
security = user
passdb backend = tdbsam
# 允许匿名
map to guest = bad user

[Test]
# 共享目录的描述
comment = Test
# 共享目录的完整路径
path = /home/Test
# 是否无需身份验证公开共享，同 guest ok
public = yes
# 是否在浏览资源中显示共享目录，若为否则必须指定共享路径
browsable = yes
# 是否以只读方式共享
read only = yes
# 是否以可写入方式共享，当与read only冲突时，无视read only
writable = yes
# 设定只有此名单内的用户才能访问共享资源（用户名/@组名）
valid users = admin
# 若设定为只读，则只有此设定的名单内的成员才可写入
write list = admin
# 建立文件时所设权限
create mask = 0644
# 建立目录时所设权限
directory mask = 0755
```

如果一块硬盘空间不足，又不想破坏共享目录结构的完整性，可在空间不足的硬盘目录内创建另一块硬盘的软链接，例如在`/mnt/nas/MEDIA/Video`中链接`/mnt/usb/纪录片`：

```bash
ln -s /mnt/usb/纪录片 /mnt/nas/MEDIA/Video
chown -R admin:SAMBA /mnt/nas/MEDIA/Video/纪录片
```

创建链接后需在 smb.conf 的`[global]`中增加以下配置：

```text
follow symlinks = yes
wide links = yes
unix extensions = no
```

## 五. 开机启动及日常启停

```bash
systemctl enable smb
systemctl start smb
systemctl stop smb
systemctl restart smb
systemctl status smb
```

## 六. 防火墙设置

设置防火墙允许Samba服务通过：

```bash
firewall-cmd --zone=public --add-service=samba --permanent
firewall-cmd --reload
```

防火墙的其他相关命令如下：

```bash
# 查看防火墙所有已开放的服务或端口
firewall-cmd --list-all
# 查看防火墙所有已开放的端口
firewall-cmd --zone=public --list-ports
# 查看防火墙所有已开放的服务
firewall-cmd --zone=public --list-services
# 开放防火墙 TCP:80 端口
firewall-cmd --zone=public --add-port=80/tcp --permanent
# 移除防火墙 TCP:80 端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent
# 移除防火墙 samba 服务
firewall-cmd --zone=public --remove-service=samba --permanent
```

## 七. 测试

至此，可以在windows通过`\\内网IP`访问共享目录。如果只能看到目录不能看到文件，可通过命令`setenforce 0`临时解决（重启后失效），或通过修改配置长期解决：

```bash
vi /etc/selinux/config
```

找到`SELINUX=enforcing`改为`SELINUX=disabled`，保存退出。

如果 windows 下因多次登录出现“不允许用户使用一个以上用户名与一个服务器或共享资源的多重连接”提示，可进入cmd输入如下命令清除连接：

```bash
net use * /del /y
```

如果 Samba 服务器配置正确，但 Windows 始终登录失败，可运行`secpol.msc`依次打开“安全设置-&gt;本地策略-&gt;安全选项”，找到“网络安全：LAN管理器身份验证级别”，修改为“仅发送NTLMv2响应，拒绝LM和NTLM”。

