# Ubuntu 22.04 LTS 新手上路软件篇

## 1. Google Chrome

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i ./google-chrome-stable_current_amd64.deb
```

## 2. Qv2ray

到`https://github.com/Qv2ray/Qv2ray/releases`下载 Qvray
客户端，`https://github.com/v2fly/v2ray-core/releases`下载 v2ray-core。前者为
AppImage 文件；后者为压缩文件，解压后随便放在哪个目录。注意 Qvray 最新版本为
v2.7.0，v2ray-core 版本必须选择 v4.45.2（截至 2023-06-23
的最新稳定版，更高的预览版本会报错）。

将 Qv2ray、v2ray-core 设为可执行，运行
Qv2ray，进入“首选项->内核设置”，选择上一步解压的 v2ray-core
路径；切换到“入站设置”，将监听地址由 127.0.0.1 改为
0.0.0.0，以便局域网内其他设备可连接此代理；切换到“常规设置”，勾选“登录时启动”，并选择“记忆上次链接”。添加订阅后即可使用。

### 2.1 设置快捷方式

创建快捷方式配置文件：

```
vi ~/.local/share/applications/qv2ray.desktop
```

输入以下内容：

```
[Desktop Entry]
Type=Application
Name=Qv2ray
Exec=/home/xehu/Apps/qv2ray/Qv2ray-v2.7.0-linux-x64.AppImage
Icon=/home/xehu/Apps/qv2ray/icon.png
```

## 3. Bomi Video Player

```
sudo add-apt-repository ppa:nemonein/bomi
sudo apt install bomi
```

## 4. Sublime Text 3

Sublime Text 虽然不是开源免费软件，但其试用版几乎很少弹出购买提示，且永久可用。

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo add-apt-repository "deb https://download.sublimetext.com/ apt/stable/"
sudo apt update
sudo apt install sublime-text
```

## 5. Github Desktop

参见：[https://github.com/shiftkey/desktop](https://github.com/shiftkey/desktop)

```
wget https://github.com/shiftkey/desktop/releases/download/release-3.2.1-linux1/GitHubDesktop-linux-3.2.1-linux1.deb
sudo dpkg -i ./GitHubDesktop-linux-3.2.1-linux1.deb
rm -rf GitHubDesktop-linux-3.2.1-linux1.deb
```

## 6. NodeJS & NPM

```
sudo apt install nodejs
sudo apt install npm
node -v
npm -v
sudo npm install n -g
sudo n stable
```

## 7. Deno & Deployctl

```
curl -fsSL https://deno.land/x/install/install.sh | sh
```

`vi ~/.bashrc`在末尾添加以下内容，然后`source ~/.bashrc`。

```
export DENO_INSTALL="/home/xehu/.deno"
export PATH="$DENO_INSTALL/bin:$PATH"
```

安装Deployctl

```
deno install --allow-all --no-check -r -f https://deno.land/x/deploy/deployctl.ts
```

## 8. VS Code

到官网 https://code.visualstudio.com/download 下载deb安装包，执行以下命令

```
sudo dpkg -i code_1.79.2-1686734195_amd64.deb
rm -rf code_1.79.2-1686734195_amd64.deb
```

## 9. Open JDK

```
sudo apt install openjdk-17-jdk
java --version
```

## 10. Eclipse

到官网 https://www.eclipse.org/downloads/
下载安装程序，解压后执行`eclipse-inst`文件，安装完成后删除压缩包和解压后的目录。

## 11. WPS Office

官网 https://www.wps.com/download/ 下载 deb 安装包，执行以下命令：

```
sudo dpkg -i ./wps-office_11.1.0.11698.XA_amd64.deb
rm -rf wps-office_11.1.0.11698.XA_amd64.deb
```

## 12. [Deluge](https://dev.deluge-torrent.org/wiki/Download)

```
sudo apt install deluge
```

## 13. Flawless-Cut

```
wget https://github.com/metadream/app-flawless-cut/releases/download/1.0.0/flawless-cut_1.0.0_amd64.deb
sudo dpkg -i flawless-cut_1.0.0_amd64.deb
```

## 14. 其他软件

- Inkscape
- Dbeaver
- Remmina
- Audacity
- Rclone
- Clonezilla

## 99. 软件卸载说明

- `apt list --installed` 查看所有已安装软件
- `apt remove <package>` 卸载软件及其下属依赖
- `apt purge <package>` 卸载软件及其下属依赖及配置文件
- `apt autoremove [<package>]` 删除软件及所有依赖，慎用！！！
- `apt clean`
  使用`apt`安装软件时，包管理器首先会以.deb格式下载软件及其依赖到`/var/cache/apt/archives/partial`目录，当
  deb
  包完全下载完毕后，它会被移到`/var/cache/apt/archives`目录下，安装或卸载后重新安装时，系统会从该目录读取缓存包。clean的作用就是一次性删除该目录下所有缓存包。
- `apt autoclean` 与 clean 不同的是，autoclean
  只删除那些无法从仓库中下载的包。比如某个包出现了新版本，原来的包无法再下载。
