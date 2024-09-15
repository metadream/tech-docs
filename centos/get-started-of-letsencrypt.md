# Let's Encrypt 域名证书安装

[Let's Encrypt](https://letsencrypt.org) 致力于清除 Web 安全障碍，宣布 ACME v2 正式支持[免费通配符证书](https://www.sslforfree.com)。详细安装及配置过程参见：[https://github.com/Neilpang/acme.sh/wiki/说明](https://github.com/Neilpang/acme.sh/wiki/说明) ，以下仅针对个人情况作记录：

```bash
curl https://get.acme.sh | sh
source ~/.bashrc
export DP_Id="xxxxxx"
export DP_Key="xxxxxx"
acme.sh --issue --dns dns_dp -d example.com -d *.example.com
acme.sh --installcert -d example.com -d *.example.com   \
        --key-file /etc/nginx/sslcert/example.com.key \
        --fullchain-file /etc/nginx/sslcert/fullchain.cer \
        --reloadcmd  "systemctl force-reload nginx"
```

使用 Cloudflare 解析的域名需修改其中三行语句：

```bash
export CF_Key="xxxxxx"
export CF_Email="xxx@xxx.com"
acme.sh --issue --dns dns_cf -d example.com -d *.example.com
```

如遇如下错误，可能由于域名解析中存在子域为 @ 的 CNAME 记录，暂停解析即可。

```text
Verify error: CAA record for *.example.com prevents issuance
```

Let's Encrypt 免费证书有效期为三个月，acme已自动加入系统定时任务，每天检查过期时间并自动更新证书，也可使用以下命令手动重新生成：

```text
acme.sh --renew --dns dns_dp -d example.com -d *.example.com
```

