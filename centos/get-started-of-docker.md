# Docker 安装与使用

## 一. 安装 Docker

```bash
yum -y install docker
docker version
```

## 二. 安装 Jar 程序镜像

首先，将 springboot 的源码打包成可执行性 jar 文件并上传到服务器，在 jar 的同级目录下创建 dockerfile 文件，内容如下：

```bash
# 拉取对应版本的 JDK
FROM openjdk:12
# 作者签名
MAINTAINER cloudseat
# 创建临时数据目录
VOLUME /tmp
# 复制 jar 包到 docker 根目录
ADD fallboot-1.0.0.jar /fallboot.jar
# 指定容器启动脚本
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/fallboot.jar"]
# 设置对外端口（已可以在启动时覆盖）
EXPOSE 9090
```

然后构建 Jar 的镜像

```bash
docker build -t fallboot .
```

最后创建并运行容器，命令中的 `-d` 表示守护进程，运行之后，其它主机通过宿主机IP访问容器提供的服务。

```bash
docker run -d -p <宿主机端口>:<容器端口> --name <容器名称> <镜像名称>
docker run -d -p 8080:8080 --name fallboot fallboot
```

或者：容器中运行程序和在宿主机中运行一样，访问宿主机IP就可以访问容器服务。

```bash
docker run -d --net=host --name <容器名称> <镜像名称>
docker run -d --net=host --name fallboot fallboot
```

## 三. 安装 Memcached 镜像

```bash
docker pull memcached
docker run -d -p 11211:11211 --name memcached memcached
# docker run -d --net=host --name memcached memcached
```

## 四. 常用命令

```bash
# 搜索镜像
docker search <镜像名称>
# 显示已安装镜像列表
docker images
# 删除镜像
docker rmi <镜像ID>
# 显示容器列表 -a表示所有（默认只显示正在运行的容器）
docker ps -a
# 自启动容器
docker update --restart=always <容器ID>|<容器名称>
# 取消自启动
docker update --restart=no <容器ID>|<容器名称>
# 查看容器信息
docker inspect <容器ID>|<容器名称>
# 查看容器日志
docker container logs <容器id>|<容器名称>
# 运行容器
docker start <容器ID>|<容器名称>
# 停止容器
docker stop <容器ID>|<容器名称>
# 删除容器
docker rm <容器ID>|<容器名称>
# 启动 docker
systemctl start docker
```

