# Ubuntu 22.04 LTS 新手上路系统篇

> 系统篇主要包括最小安装下的系统优化和增强

## 0. Ubuntu 下载与安装

到官网 [https://ubuntu.com/#download](https://ubuntu.com/#download) 下载 Ubuntu
22.04 LTS (长期支持版) 系统镜像，用 [Rufus](https://rufus.ie/) 工具将其刻录成 U
盘启动盘，刻录时使用默认设置即可。刻录完成后重启电脑，狂按 F2 或 F12 进入
BIOS，选择 U 盘启动，然后会进入 Ubuntu
安装界面，基本上一路默认前行，其中两个步骤可以视情况调整：一是是否选择最小安装，二是硬盘分区。对于前者，最小化安装不包含
Office、多媒体等办公娱乐软件；对于后者，官方认为无需分区，但传统习惯使然，还是可以考虑如下简单的分区方式：

|   | Mount | FS   | Size      |
| - | ----- | ---- | --------- |
| 1 | /     | ext4 | 40G       |
| 2 | /efi  |      | 500M      |
| 3 | /swap |      | 8G        |
| 4 | /home | ext4 | Remaining |

安装结束后会自动重启电脑，并提示你拔掉 U 盘正式进入 Ubuntu
系统。最小安装模式自带的软件有限，因此可能需要继续下述章节的优化与增强。

## 1. 卸载自带的软件中心

```
// 0. Review snap packages installed
snap list

// 1. Stop snapd (snap daemon) services:
sudo systemctl disable snapd.service
sudo systemctl disable snapd.socket
sudo systemctl disable snapd.seeded.service

// 2. Remove each snap. It’s best to do so one-by-one, rather than all in one apt remove line.
sudo snap remove firefox
sudo snap remove snap-store
sudo snap remove gtk-common-themes
sudo snap remove gnome-3-38-2004
sudo snap remove snapd-desktop-integration
sudo snap remove core20
sudo snap remove bare

// 3. Now, let’s delete any leftover snap cached data:
sudo rm -rf /var/cache/snapd/

// 4. Then purge or remove completely snapd using the following command:
sudo apt autoremove --purge snapd

// 5. Finally, using purge doesn’t touch your home directory, so you can optionally delete any files previously created in ~/snap.
rm -rf ~/snap

// 6. Refresh apt cache
sudo apt update
```

## 2. 修改个人文件夹名称

如果安装时选择中文语言，则家目录下文档、图片、音乐、视频等个人文件夹名称均为中文，在使用时多有不便，可将其改为英文。当然，如果整个系统都改为英文界面则无此烦恼。进入设置->区域与语言->选择英语->注销，系统将提示是否更改个人文件夹名称，选择本次更改且下次不再提示，再回到设置->区域与语言->选择中文->注销。此方式比通过命令行修改更安全。

## 3. 修改命令行提示符

`vi ~/.bashrc` 打开终端配置文件，在末尾添加如下内容：

```
// 此处提示符为绿色
export PS1="\[\e[32m\][\u@\h:\W]# \[\e[m\]"

// 顺便添加两个别名（如果存在则修改）
alias ll='ls -lF'
alias la='ls -AF'
```

保存关闭后，执行 `source ~/.bashrc` 命令使其生效。针对 root 用户，可 `sudo -i`
登录后按照上述步骤执行一遍，内容替换为：

```
// 此处提示符为紫色
export PS1="\[\e[35m\][\u@\h:\W]$ \[\e[m\]"
```

## 4. 免密 sudo

用 `sudo visudo` 命令打开 `/etc/sudoers` 配置文件，将其中：

```
%sudo ALL=(ALL:ALL) ALL
```

修改为

```
%sudo ALL=(ALL:ALL) NOPASSWD:ALL
```

## 5. 安装必备工具

```
// ifconfig、netstat 等网络工具
sudo apt install net-tools

// 运行 AppImage 文件所需依赖
sudo apt install libfuse2

// 增强默认 vi 编辑器
sudo apt install vim
// 将 vim 设为默认，执行以下命令后选择 /usr/bin/vim.basic
sudo update-alternatives --config editor

// 使自带归档管理器支持更多格式
sudo apt install rar unrar p7zip-full

// 使自带图片查看器支持 webp
sudo add-apt-repository ppa:helkaluin/webp-pixbuf-loader
sudo apt install webp-pixbuf-loader

// 自动生成视频文件缩略图
sudo apt install ffmpegthumbnailer
```

## 6. 设置缩略图显示

系统默认缩略图仅针对本机图片和视频，如果要在网络目录显示缩略图，进入文件管理器->首选项，分别设置为“所有位置”、“所有文件”、“所有文件夹”。

## 7. 任务栏点击行为

Ubuntu 点击任务栏图标的默认行为不像 Windows
一样最小化，以下命令可启用最小化点击（或通过系统扩展实现）：

```
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```

## 8. 右键新建文本文档

```
cd ~/Templates
touch Text Document.txt
```

## 9. 启用/禁用触摸板

```
sudo modprobe psmouse
sudo modprobe -r psmouse
```

## 10. SSH 配置

为方便远程连接服务器，可创建 SSH 配置文件 `vi ~/.ssh/config`，输入如下内容：

```
Host nas
  HostName 10.0.0.2
  Port 22
  User root
  IdentityFile ~/.ssh/id_rsa_nas
```

其中 `IdentityFile`
是对称密钥中的私钥，公钥放在服务器上，如服务器无需密钥登录则可省略。然后就可以通过以下命令连接服务器：

```
ssh nas
```

## 11. 取消 Chrome 密码

如果每次开机后启动 chrome 都需要输入密码，可进入 Password and Keys
应用程序，右键单击左侧菜单中的“登录”项，选择“更改密码”，输入旧密码后让新密码置空并保存。

## 12. 安装人脸识别 Howdy

由于网络限制，在国内安装 howdy
的过程相当漫长（1个多小时），甚至可能半途而废。所以，首先学会科学上网，在终端加入网络代理，即便如此，安装过程中
howdy
的内部还会请求境外软件包（一般的代理无法解决此类情况）而导致失败，所以有条件的话需保证绝对的全局代理。

### 12.1 预先下载3个文件

这3个文件在无代理执行安装的过程中通常很难下载。

```
sudo mkdir -p /lib/security/howdy/dlib-data
cd /lib/security/howdy/dlib-data/
sudo wget https://github.com/davisking/dlib-models/raw/master/dlib_face_recognition_resnet_model_v1.dat.bz2
sudo wget https://github.com/davisking/dlib-models/raw/master/mmod_human_face_detector.dat.bz2
sudo wget https://github.com/davisking/dlib-models/raw/master/shape_predictor_5_face_landmarks.dat.bz2
```

### 12.2 安装 howdy

```
sudo add-apt-repository ppa:boltgolt/howdy
sudo apt update
sudo apt install howdy
```

正常情况下执行以上三条命令即可。中途有一个选择，选b（balance）即可：

```
What profile would you like to use? [f/b/s]: b
```

非正常情况下，会出现如下错误：

```
Downloading 3 required data files...
Unpacking...
bzip2: Can't open input file *.bz2: No such file or directory.
```

出现此错误时需要先操作第一步骤。

### 12.3 修改配置

```
sudo howdy config
```

在打开的文件中找到 `device_path` 并修改如下：

```
device_path = /dev/video0
timeout = 10
```

其中 `/dev/video0`
为通常情况下的摄像头地址，如果不是这个地址，则需要另外安装工具来查看。

### 12.4 增加面部模型

```
sudo howdy add
```

执行后会提示输入模型名称，然后开启摄像头扫描面部，完成后重启即可使用人脸登录。建议使用多添加几个面部模型，如戴眼镜、不戴眼镜、正视摄像头、侧对摄像头、近看摄像头、远看摄像头等，以增加识别概率。

### 12.5 其他命令

```
sudo howdy test             // 测试摄像头
sudo howdy snapshot         // 使用摄像头拍摄快照
sudo howdy list             // 列出所有面部模型记录
sudo howdy remove <faceID>  // 删除指定 ID 的面部记录
sudo howdy clear            // 清除所有面部模型记录
sudo howdy disable 1        // 禁用 Howdy
sudo howdy disable 0        // 启用 Howdy
sudo howdy version          // 查看 Howdy 版本
```

## 13. 挂载 Samba 目录

### 13.1 创建挂载点

```
sudo mkdir /mnt/temp
```

### 13.2 创建认证文件

可以在任意位置以任意名称创建该文件，例如 `vi /home/xehu/.smbcreds`，内容如下：

```
username=abc
password=123
```

### 13.3 安装 cifs

```
sudo apt install cifs-utils
```

### 13.4 修改文件系统表

`sudo vi /etc/fstab` 在文件末尾添加如下行：

```
//10.0.0.2/temp /mnt/temp cifs credentials=/home/xehu/.smbcreds,noauto,x-systemd.automount,dir_mode=0777,file_mode=0777 0 0
```

然后执行 `sudo mount -a` 即可挂载（或重启后自动生效）。

值得一提的是 `fstab` 中 `noauto,x-systemd.automount`
参数，其含义是按需（访问时）挂载。以我的机器为例（不知道是不是个例），不加此参数无法实现重启后自动挂载，严格说是有时能挂载有时不能挂载，大概率是网络与
fstab 启动顺序不确定导致。在网上查询了好多文章，有的说增加
`auto`，但不起作用，其实默认就是 auto；有的说增加
`_netdev`，延迟挂载网络共享直到网络启动，但据大神翻看源代码发现 Linux 其实已经将
cifs
等格式自动识别为网络文件系统。更多参数参见：https://zhangguanzhang.github.io/2019/01/30/fstab/

## 14. 终端代理

通常有以下三种方式，但前提是已经安装网络代理服务如 v2ray。

### 14.1 环境变量法

执行如下三组命令之一（仅对当前终端有效）：

```
export http_proxy=http://127.0.0.1:8889
export https_proxy=http://127.0.0.1:8889

// 或者
export http_proxy=socks5://127.0.0.1:8888
export https_proxy=socks5://127.0.0.1:8888

// 或者
export ALL_PROXY=socks5://127.0.0.1:8888
```

或将其写入 `.bashrc` 文件以便持续使用。设置变量对有些软件（如
curl、wget）来说有效，对有些软件（如
git、apt）则无效，它们有自己单独设置代理的方式，不在此展开。

### 14.2 Proxychains (推荐)

首先安装该软件的第四个版本：

```
sudo apt install -y proxychains4
```

然后编辑 `/etc/proxychains4.conf` 文件，在 `[ProxyList]`
一节增加代理设置，例如：

```
socks5 127.0.0.1 8888
```

同时注释该文件的远程 DNS 解析，防止 DNS 污染风险：

```
#proxy_dns
```

该方案几乎可代理所有情况。当需要使用代理时，只需在命令前面增加 `proxychains`
即可，例如：

```
proxychains git clone https://github.com/...
proxychains apt install ...
```
