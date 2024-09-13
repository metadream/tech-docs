# PM2 安装与使用

## 一. 全局安装

```text
npm install -g pm2
```

## 二. 启动 node 应用

指定 js 入口启动，其中 `--name` 为应用名称，`--watch` 监视文件变化（即热部署）。

```text
pm2 start /path/to/main.js --name="appname" --watch
```

也支持 npm 启动，进入项目目录（package.json 所在目录）后，执行以下命令：

```text
pm2 start --name="appname" npm -- start --watch
```

## 三. 设置开机启动

1. 保存当前应用进程列表到文件

   ```text
   pm2 save
   ```

2. 创建开机启动命令（注意根据提示继续操作）

   ```text
   pm2 startup
   ```

3. 取消开机启动

   ```text
   pm2 unstartup
   ```

## 四. 其他命令

1. 显示所有应用程序

   ```text
   pm2 list
   pm2 status
   ```

2. 显示每个应用程序的CPU和内存占用情况

   ```text
   pm2 monit
   ```

3. 显示应用程序详细信息

   ```text
   pm2 show appname
   ```

4. 显示应用程序的日志

   ```text
   pm2 logs appname
   ```

5. 重启/停止/删除（指定或所有）应用程序

   ```text
   pm2 restart appname
   pm2 stop appname
   pm2 delete appname
   pm2 restart all
   pm2 stop all
   pm2 delete all
   ```

6. 负载均衡

   ```text
   # 启动 4 个应用实例并自动负载均衡（cluster mode）
   pm2 start main.js -i 4
   # 重启 cluster mode 下的所有应用
   pm2 reload all
   ```

