# Nginx+Apache+PHP 服务安装

## 一. 安装 Nginx

```bash
yum install -y nginx
systemctl enable nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
```

在 `nginx.conf` 文件中设置允许发布内容的最大许可（即上传文件大小限制）

```text
client_max_body_size 2M
```

## 二. 安装 Apache

```bash
yum install -y httpd
systemctl enable httpd
systemctl start httpd
systemctl stop httpd
systemctl restart httpd
```

在 `httpd.conf` 中设置设置文件上传限制：

```text
<IfModule mod_fcgid.c>
  MaxRequestLen 10485760   # 10M = 10 * 1024 * 1024
  LimitRequestBody 10485760
</IfModule>
```

## 三. 安装 PHP

查看当前已安装的 PHP 包，如果存在则先卸载：

```bash
yum list installed | grep php
yum remove php56w.x86_64 php56w-cli.x86_64 php56w-common.x86_64 php56w-fpm.x86_64 php56w-gd.x86_64 php56w-ldap.x86_64 php56w-mbstring.x86_64 php56w-mcrypt.x86_64 php56w-mysqlnd.x86_64 php56w-pdo.x86_64 php56w-pear.noarch php56w-pecl-memcache.x86_64 php56w-process.x86_64 php56w-xml.x86_64
```

查看可安装的 PHP 包（以 PHP7 为例），如果没有则先更换rpm源：

```bash
yum list php7*
rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```

安装 PHP7 及其扩展：

```bash
yum install php72w-fpm.x86_64 php72w-common.x86_64 php72w-gd.x86_64 php72w-mbstring.x86_64 php72w-pdo.x86_64 php72w-mysqlnd.x86_64 php72w-pecl-memcached.x86_64
# 单独卸载某php扩展
yum remove php72w-mysql.x86_64
# 单独安装某php扩展（mysqlnd是mysql的原生驱动，官方推荐）
yum install php72w-mysqlnd.x86_64
# 查看已安装的php版本
php -v
# 查看已安装的php扩展
php -m
```

`php.ini` 文件中可能有用的配置

```text
max_execution_time = 150   # 每个脚本运行最长时间（秒）
max_input_time = 60        # 每个脚本可以消耗的时间（秒）
memory_limit = 128M        # 脚本最大消耗内存
post_max_size = 8M         # 表单提交的最大数据
upload_max_filesize = 10M  # 上传文件的最大许可
always_populate_raw_post_data = -1
extension = php_openssl.dll
```

设置开机启动及日常启停

```bash
systemctl enable php-fpm
systemctl start php-fpm
systemctl stop php-fpm
systemctl restart php-fpm
```

### 1. 修改启动权限

在某些特殊情况下，例如 PHP 代码调用较高权限的 Shell 命令时，需要以 root 身份启动 php-fpm，打开配置文件：

```bash
vi /etc/php-fpm.d/www.conf
```

将以下内容中的 `apache` 改为 `root`：

```text
user = apache
group = apache
```

由于 php 默认不允许以 root 身份运行，此时重启 php-fpm 应该会报错，所以还需修改启动脚本：

```bash
vi /usr/lib/systemd/system/php-fpm.service
```

将其中 `ExecStart` 行增加 `-R` 参数，如下：

```bash
ExecStart=/usr/sbin/php-fpm --nodaemonize -R --fpm-config /etc/php-fpm.conf
```

### 2. 与 Nginx 集成

查看 php-fpm 默认配置`cat /etc/php-fpm.d/www.conf | grep -i 'listen ='`，若返回结果为`listen = 127.0.0.1:9000`，表明监听端口为9000，Nginx配置中的PHP解析请求转发到 127.0.0.0:9000 处理即可，通常无需特别处理。

### 3. 多个 php-fpm 主进程

找到`php-fpm.d/www.conf`，复制一份改名为例如`api.conf`，将其中`[www]`及`listen = 127.0.0.1:9000`改为`[api]`及`listen = 127.0.0.1:9001`，配置 nginx 中 proxy\_pass 的端口为9001，重启 php-fpm 和 nginx。

在 Windows 下，同一 Nginx 的不同 Server 之间通过 curl 请求时会阻塞超时，原因是各 Server 共享 php-fpm 的 9000 端口，解决方法就是上述开启新端口供 Server 使用，但 CentOS 下不会出现这种情况，所以无需启动多个端口。

