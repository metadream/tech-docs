# V2ray 安装与使用

## 一. 安装服务端

```bash
curl -O https://install.direct/go.sh
chmod +x go.sh
./go.sh

# or
bash <(curl -L -s https://install.direct/go.sh)
```

完成后会显示入站端口和UUID信息，可先复制下来备用也可以后续在`/etc/v2ray/config.json`配置文件中查看。启动 v2ray：

```bash
systemctl start v2ray
```

## 二. 安装客户端

进入 [https://github.com/2dust/v2rayN/releases](https://github.com/2dust/v2rayN/releases) 页面下载 v2rayN-Core.zip，解压至本地后运行`v2rayN.exe`。

### 1. 添加 VMess 服务器

在客户端界面上点击服务器图标-&gt;添加 VMess 服务器，基本上只需填写以下四个值，其他默认：

* 地址：服务器IP
* 端口：服务端配置文件中的入站端口
* 用户ID：服务端配置文件中的 UUID
* 额外ID：服务端配置文件中的 alterId

### 2. 参数设置

点击界面中的参数设置图标，记住第一个Tab页“基础设置”的本地监听端口；点开第二个Tab页“路由设置”，路由模式选择“绕过局域网及大陆地址”。

### 3. 启动HTTP代理

右键点击托盘图标，点击“启用HTTP代理”，代理模式设为“开启PAC”。

### 4. 设置浏览器代理

如果使用 Chrome 浏览器，可以安装 Proxy SwitchyOmega 扩展程序以设置代理，也可以采用如下方式：

新建 chrome.exe 快捷方式，右键点击属性，在目标栏后面增加启动参数（与原值之间有空格），其中端口号是“基础设置”中的本地监听端口。

```bash
--proxy-server="socks5://127.0.0.1:10808"
```

设置完毕之后，点击该快捷方式即可科学上网。

## 三. Websocket, TLS 和 CDN

为尽可能降低被墙的风险，还可以在以上基础上增加 Websocket, TLS 和 CDN 部署，其原理是将服务器伪装成 Web 站点，通过 TLS 和 Websocket 建立连接，同时利用 CDN 隐藏服务器的真实IP。

### 1. 增加 Websocket 配置

编辑 v2ray 配置文件 `/etc/v2ray/config.json`，在 `inbounds` 中增加以下内容（与 `settings` 并列，注意格式中的逗号）：

```javascript
"streamSettings": {
  "network": "ws",
  "wsSettings": {
    "path": "/etc/nginx/html"
  }
}
```

### 2. 增加 Nginx 配置

`vi /etc/nginx/conf.d/v2ray.conf`创建新的配置文件，内容如下：

```text
server {
  listen 80;
  server_name warn.ga;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2 default_server;
  server_name warn.ga;

  ssl_certificate /etc/nginx/ssl/warn.ga.cer;
  ssl_certificate_key /etc/nginx/ssl/warn.ga.key;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
  ssl_prefer_server_ciphers on;

  root /etc/nginx/html;
  index index.html;

  location /etc/nginx/html {
    proxy_redirect off;
    proxy_pass http://127.0.0.1:34180;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $http_host;
  }
}
```

其中与 SSL 证书相关的配置参见 [/centos/get-started-of-letsencrypt.md](/centos/get-started-of-letsencrypt.md)。

### 3. 增加 CDN 设置

以 Cloudflare 为例，给域名增加A记录指向服务器IP，并点亮橙色云朵图标。

### 4. 重新配置客户端

重启 v2ray 服务端、重启 Nginx、CDN 生效之后，回到客户端修改服务器信息。

* 地址：由服务器IP改为域名
* 端口：由v2ray端口改为web端口443
* 用户ID：不变
* 额外ID：不变
* 传输协议：由tcp改为ws
* 路径：/etc/nginx/html
* 底层安全传输：tls

当然，还可以在`/etc/nginx/html`目录下创建一个真实的网站，让伪装来得更逼真一点。

