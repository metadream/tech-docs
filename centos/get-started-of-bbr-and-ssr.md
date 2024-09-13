# 安装 BBR & SSR

## 一. 安装 Wget

```bash
sudo -i
yum install -y wget
```

## 二. 安装 BBR

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

## 三. 安装 SSR

安装过程中需要进行几次选择，其中混淆方式推荐`tls1.2_ticket_auth`，协议推荐使用`auth_sha1_v4`、`auth_aes128_md5`或`auth_aes128_sha1`。

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log

systemctl start shadowsocks-r
```

