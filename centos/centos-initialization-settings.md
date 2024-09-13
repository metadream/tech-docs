# CentOS 初始化设置

## 1. 修改主机名

1.1. 永久生效、下次登录生效，不支持大写
```
hostnamectl set-hostname appserver
```

1.2. 永久生效、下次登录生效，支持大写
```
hostnamectl set-hostname --static APPSERVER
```

## 2. 设置DNS

通过`ifconfig`命令查看网卡名称，在`/etc/sysconfig/network-scripts/ifcfg-xxxxx`找到对应名称的设置文件，添加如下信息：
```
DNS1=211.141.90.68
DNS2=8.8.8.8
```
重启网络：
```
systemctl restart NetworkManager
```

### 3. 设置时区
```
timedatectl set-timezone Asia/Shanghai
```
同时检查数据库时区：`show variables like '%time_zone%';`

启用时间同步服务：
```
systemctl start chronyd
systemctl enable chronyd
```

### 4. 切换 CentOS8 软件源
CentOS8 已停止维护（被CentOS 8 Stream替代），软件源已失效，切换为阿里云镜像源。
```
cd /etc/yum.repos.d/
mkdir repo-bak
mv *.repo repo-bak/

wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
yum clean all
yum makecache
yum update
```

### 5. 开放端口
```
firewall-cmd --zone=public --add-service=mysql --permanent
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

### 6. 允许 httpd 连接网络
例如 Nginx 反向代理 Springboot 服务。
```
setsebool -P httpd_can_network_connect 1
getsebool httpd_can_network_connect
```

### 7. 允许 httpd 使用特殊端口
```
semanage port -l | grep http_port_t
semanage port -a -t http_port_t -p tcp 8008 // 增加
semanage port -d -t ssh_port_t -p tcp 8008 // 删除
```

### 8. SCP 免密上传
1. 客户端上执行：`ssh-keygen -t rsa`，一路回车，将在`C:\Users\Administrator\.ssh`下生成`id_rsa.pub`等文件；
2. 服务器上执行：`vi ~/.ssh/authorized_keys`（不存在则创建），将`id_rsa.pub`的内容追加进去。
