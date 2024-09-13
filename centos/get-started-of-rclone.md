# Rclone 安装与使用

### 一. 安装 Rclone

```bash
yum install p7zip unzip -y
curl https://rclone.org/install.sh | bash
```

### 二. 配置 Google Drive

执行`rclone config`命令，会出现以下看似复杂其实很容易操作的信息，需要填写或选择的部分见注释：

```bash
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n # 创建一个新的远程配置
name> gd # 随便取一个配置名称
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Provider (AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, etc)
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Dropbox
   \ "dropbox"
 9 / Encrypt/Decrypt a remote
   \ "crypt"
10 / FTP Connection
   \ "ftp"
11 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
12 / Google Drive
   \ "drive"
13 / Google Photos
   \ "google photos"
14 / Hubic
   \ "hubic"
15 / JottaCloud
   \ "jottacloud"
16 / Koofr
   \ "koofr"
17 / Local Disk
   \ "local"
18 / Mega
   \ "mega"
19 / Microsoft Azure Blob Storage
   \ "azureblob"
20 / Microsoft OneDrive
   \ "onedrive"
21 / OpenDrive
   \ "opendrive"
22 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
23 / Pcloud
   \ "pcloud"
24 / Put.io
   \ "putio"
25 / QingCloud Object Storage
   \ "qingstor"
26 / SSH/SFTP Connection
   \ "sftp"
27 / Union merges the contents of several remotes
   \ "union"
28 / Webdav
   \ "webdav"
29 / Yandex Disk
   \ "yandex"
30 / http Connection
   \ "http"
31 / premiumize.me
   \ "premiumizeme"
Storage> 12    # 选择 Google Drive，注意该序列号可能会变化，看清楚自己所需
client_id>     # 在启用谷歌 API 页面上生成的客户端 ID
client_secret> # 在启用谷歌 API 页面上生成的客户端 Secret
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 2 # 选择授权范围：全部、只读、读写等等
ID of the root folder
Leave blank normally.
Fill in to access "Computers" folders. (see docs).
Enter a string value. Press Enter for the default ("").
root_folder_id> # 根目录 ID，置空即可
Service Account Credentials JSON file path 
Leave blank normally.
Needed only if you want use SA instead of interactive login.
Enter a string value. Press Enter for the default ("").
service_account_file> # 服务账号 JSON 文件路径，置空即可
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n # 是否进入高级配置，否即可
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n # 是否使用自动配置，否即可
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/  #打开该地址获取code
Log in and authorize rclone for access
Enter verification code>  # 此步骤会出现回调 URL 链接，将此链接复制到浏览器会请求登录谷歌，并询问是否同意授权，同意后会得到 code
Configure this as a team drive?
y) Yes
n) No
y/n> n # 将配置设为团队云盘的，否即可
--------------------
[gd]
type = drive
client_id = 
client_secret = 
scope = drive.readonly
token = {}  # 此步骤会显示最终结果，复制其中的 refresh_token 供 API 使用
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y # 确认结果，是
Current remotes:

Name                 Type
====                 ====
gd                   drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q # 退出
```

使用以下命令可以查看配置文件路径：

```bash
rclone config file | grep rclone.conf
```

里面储存了读写 GoogleDrive 所需参数，其中配置名称（此例中为`gd`）是 Rclone 后续操作的重要标识。

### 三. 配置 Tencent COS

腾讯对象存储（COS）默认不包含在配置过程的选项中，可直接在配置文件中添加以下内容（去掉注释）：

```bash
[cos]
type = s3           # 对象存储似乎必须是s3类型
provider = Other
acl = public-read
env_auth = false    # 不从环境变量中获取密钥
access_key_id =     # 腾讯云的 SecretId
secret_access_key = # 腾讯云的 SecretKey
endpoint = cos.ap-hongkong.myqcloud.com # COS域名
```

其中`acl`参数需注意，同步时会根据此设置覆盖原有权限。对于腾讯COS来说，可选值有：`private`私有读写（默认）；`public-read`公有读私有写；`public-read-write`公有读写。

### 四. Rclone 常用操作

#### 1. 列出路径下的目录或存储桶

```bash
rclone lsd [远程配置名称]:[远程目录绝对路径]

# 例如
rclone lsd gd:/image
rclone lsd cos:/image
```

#### 2. 列出路径下的文件

```text
rclone ls [远程配置名称]:[远程目录绝对路径]
```

#### 3. 查看路径下文件数目和总大小

```text
rclone size [远程配置名称]:[远程目录绝对路径]
```

#### 4. 将源目录同步到目标目录

同步命令与`rclone copy`不同，会删除目标目录中多余的文件，`--dry-run`参数可预先查看哪些文件将被删除和变更，正常使用时去掉。如果是本地路径，冒号及前面的名称可去掉。

```bash
rclone sync [源名称]:[源绝对路径] [目标名称]:[目标绝对路径] --dry-run
```

#### 5. 挂载远程云盘到本地文件系统

先要安装 FUSE（用户空间文件系统）：

```text
yum install fuse -y
```

再执行挂载命令：

```text
rclone mount [源名称]:[源绝对路径] [本地绝对路径]
rclone mount gd:/ /mnt/gd
```

该命令会占用控制台，需持续运行才能正常工作，因此通常设置为后台运行方式，或服务方式。

新建服务启动脚本：

```text
vi /etc/systemd/system/rclone.service
```

粘贴以下内容：

```text
[Unit]
Description=Rclone Mount
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount gd:/ /mnt/gd --allow-other --cache-dir /tmp/rclone.cache --vfs-cache-mode full --vfs-cache-max-age 72h
Restart=on-abort

[Install]
WantedBy=default.target
```

启动、重启、开机自启动、停止、查看服务状态：

```text
systemctl start rclone
systemctl restart rclone
systemctl enable rclone
systemctl stop rclone
systemctl status rclone
```

`df -h`查看是否挂载成功。

#### 6. 卸载云盘

```text
umount /mnt/gd
# or
fusermount -qzu /mnt/gd
```

#### 7. 排除目录或文件

新建简单文本文件：

```text
vi ~/.config/rclone/excludes.txt
```

内容如下：

```text
node_modules/**
_site/**
.git/**
.settings/**
.debris/**
.stfolder/**
.stignore
Thumbs.db
*.tmp
*.class
*~.*
~*
```

使用以下命令饮用上述文件：

```text
sudo rclone sync ~/Cloud/ /mnt/nas/Cloud/ --exclude-from ~/.config/rclone/excludes.txt -v
```

