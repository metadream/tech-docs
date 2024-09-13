# Syncthing 安装与使用

## 一. Windows Syncthing

### 1. 安装

到官网 [https://syncthing.net](https://syncthing.net) 选择与本机操作系统相符的版本，下载后解压到任意目录，双击该目录下的 `syncthing.exe` 启动，程序将自动打开浏览器 [http://127.0.0.1:8384](http://127.0.0.1:8384) 地址，如果出现管理界面即表示成功。

### 2. 开机自启动

在 `syncthing.exe` 同级目录下创建 `syncthing.bat` 文件，输入如下内容保存，然后创建该文件的快捷方式并复制到 “开始-&gt;所有程序-&gt;启动” 下。

```bash
start "Syncthing" syncthing.exe -no-console -no-browser
```

## 二. CentOS Syncthing

### 1. 安装

登录 CentOS 后，输入如下命令（根据系统自行选择版本）下载和解压，完成后可删除压缩包：

```bash
wget https://github.com/syncthing/syncthing/releases/download/v1.2.1/syncthing-linux-amd64-v1.2.1.tar.gz
tar xzvf syncthing-linux-amd64-v1.2.1.tar.gz
rm -rf syncthing-linux-amd64-v1.2.1.tar.gz
```

进入解压后的目录，将可执行文件复制到系统PATH，然后直接输入 `syncthing` 启动，待输出信息结束后按 `CTRL-C` 退出：

```bash
cd syncthing-linux-amd64-v1.2.1
cp syncthing /usr/bin
chmod +x /usr/bin/syncthing
syncthing
```

首次运行的目的是让程序自动创建配置文件，以便修改其内容使外网可以通过IP访问。

```bash
vi ~/.config/syncthing/config.xml
```

找到 `127.0.0.1:8384` 改为 `0.0.0.0:8384`，保存退出后再次执行 `syncthing`，可在 [http://IP:8384](http://IP:8384) 看到和 Windows 同样的界面（需开放防火墙对应端口）。

### 2. 开机自启动

Syncthing 自带系统服务和用户服务两种脚本，此为第一种：将脚本复制到 CentOS 系统服务目录下，然后启动服务、查看服务状态：

```bash
cp ~/syncthing-linux-amd64-v1.2.1/etc/linux-systemd/system/syncthing@.service /usr/lib/systemd/system
systemctl enable syncthing@root.service
systemctl start syncthing@root.service
systemctl status syncthing@root.service
```

## 三. 设置同步目录

假设 Windows 为本地端，CentOS 为服务器端，实现各自的目录之间双向同步。

1. 服务器端的管理界面即 [http://IP:8384](http://IP:8384) 第一次运行时会要求创建密码，创建完后到本地端即 [http://127.0.0.1:8384](http://127.0.0.1:8384) “添加远程设备”，除设备ID从服务器端复制粘贴外，其他设置可以默认。
2. 继续在本地端“添加文件夹”（默认的文件夹可删除），将路径指向本机某个需要同步的目录，并共享给服务器端；不出意外的话，服务器端界面会分别弹出“是否添加新设备”和“是否添加新文件夹”的提示，均点击确定，并将文件夹指向服务器的某个目录。
3. 至此，同步设置完成，可在两端的共享目录中分别放置文件测试同步效果，并分别重启本机和服务器测试自启动。需要注意的是：如果服务器有端口限制，需开放 TCP:8384 和 TCP:22000 端口，前者用于外网访问 Web 界面，后者用于 Socket 同步。

