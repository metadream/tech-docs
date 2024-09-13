# MinIO 安装

MinIO 是在 GNU Affero 通用公共许可证 v3.0 下发布的高性能对象存储。 它是与 Amazon
S3 云存储服务兼容的 API。 使用 MinIO
为机器学习、分析和应用程序数据工作负载构建高性能基础架构。

### 方法一：容器安装

```shell
sudo docker run -p 9000:9000 --name minio \
  -d --restart=always \
  -e "MINIO_ACCESS_KEY=testuser" \
  -e "MINIO_SECRET_KEY=testpass" \
  -v /mnt/cos:/data \
  minio/minio server /data

sudo docker images
sudo docker ps
sudo docker start minio
sudo docker stop minio
sudo docker rm minio
```

### 方法二：二进制安装

下载二进制文件并设为可执行：

```shell
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
```

`vi minio.sh` 创建Shell文件并设为可执行`chmod +x minio.sh`，内容为：

```shell
#!/bin/bash
export MINIO_ACCESS_KEY=testuser
export MINIO_SECRET_KEY=testpass
/root/minio server /mnt/cos
```

创建系统服务：

```shell
vi /etc/systemd/system/minio.service
```

内容为：

```shell
[Unit]
Description=MinIO object storage server
After=network.target

[Service]
Type=simple
ExecStart=/root/minio.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

启动服务：

```shell
systemctl start minio
systemctl enable minio
```

访问终端：

```
http://localhost:9000
```
