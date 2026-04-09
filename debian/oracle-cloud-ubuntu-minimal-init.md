# Oracle Cloud Ubuntu Minimal 服务器优化

## 1. 卸载 fwupd

```bash
sudo systemctl stop fwupd
sudo systemctl disable fwupd
sudo systemctl mask fwupd
sudo apt purge -y fwupd
```

## 2. 卸载 unattended-upgr

```bash
sudo systemctl stop unattended-upgrades
sudo systemctl disable unattended-upgrades
sudo apt purge -y unattended-upgrades
```

## 3. 卸载 oracle-cloud-agent

```bash
sudo snap remove oracle-cloud-agent
```

## 4. 卸载 snapd

```bash
sudo systemctl stop snapd.service
sudo systemctl stop snapd.socket
sudo apt purge -y snapd
sudo rm -rf ~/snap /snap /var/snap /var/lib/snapd

sudo tee /etc/apt/preferences.d/nosnap.pref <<EOF
Package: snapd
Pin: release a=*
Pin-Priority: -10
EOF

sudo apt autoremove -y
```

## 5. 安装必要的工具

```bash
sudo apt update
sudo apt install -y vim
sudo apt install -y iptables-persistent
```

## 6. 设置root密码

```bash
sudo -i
passwd root
```

## 7. 修改 SSH 配置文件

```bash
vi /etc/ssh/sshd_config
```

修改以下属性：

```
Port 20022
PermitRootLogin yes
PasswordAuthentication yes
```

删除 sshd_config.d 目录下的文件（文件名可能不一样）：

```bash
rm -rf /etc/ssh/sshd_config.d/50-cloud-init.conf
```

## 8. 删除默认的 ubuntu 用户

```bash
pkill -u ubuntu
userdel -r ubuntu
```

Oracle 默认在 root 的授权 key 文件里写了一段脚本，专门用来拦截 root 登录。删掉 authorized_keys 开头那段 no-port-forwarding,no-agent-forwarding... 以及报错提示文字。

```bash
sed -i 's/^.*ssh-rsa/ssh-rsa/' /root/.ssh/authorized_keys
```

Oracle Cloud 的 cloud-init 服务有时会在系统重启时尝试重新创建默认用户。为了彻底杜绝，建议执行：

```bash
touch /etc/cloud/cloud-init.disabled
```

## 9. 开放SSH新端口

首先需要在Oracle网站端网络安全组中开放新端口，然后开放服务器防火墙端口。先查看防火墙优先级顺序，在最后一条记录之前插入新端口，删除原端口，然后保存、重载。

```bash
# 1. 查看原规则
iptables -L INPUT -n --line-numbers

# 2. 在第5行插入规则
iptables -I INPUT 5 -p tcp --dport 20022 -j ACCEPT
iptables -I INPUT 5 -p tcp --dport 443 -j ACCEPT

# 3. 删除第4行规则
iptables -D INPUT 4

# 4. 保存规则
netfilter-persistent save

# 5. 重载规则
iptables -F INPUT # 如果出现重复先清空再加载
netfilter-persistent reload
```

重启SSH服务：

```bash
# 1. 刷新 systemd 配置
systemctl daemon-reload

# 2. 停止所有 SSH 相关组件
systemctl stop ssh.socket
systemctl stop ssh

# 3. 启动新配置的 SSH
systemctl start ssh
```
