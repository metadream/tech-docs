# Cron 定时服务和数据库备份

CentOS的定时服务名叫crontab，首先在命令行输入 `crontab` 检查是否已安装，如果提 示`command not found` 说明未安装。

## 一. 安装 Crontab

```bash
yum install vixie-cron
yum install crontabs
```

## 二. 日常启停

```bash
systemctl start crond
systemctl stop crond
systemctl restart crond
```

## 三. 创建备份脚本

例如创建脚本文件 backup.sh，内容如下：

```bash
#!/bin/bash
#This is a comment: The first line is required

db_user="xxx"
db_pass="xxx"
db_name="xxx"
path="/data/backup"
time="$(date +"%Y%m%d%H%M%S")"

mysqldump -u$db_user -p$db_pass $db_name > "$path/$db_name"_"$time.sql"
```

修改backup.sh文件属性为可执行（以下任选其一）

```bash
chmod +x backup.sh
chmod 777 backup.sh
```

执行 `./backup.sh` 测试脚本是否运行正常，若出现以下错误，大多是因为文件格式有误：

```bash
-bash: ./backup.sh: /bin/bash^M: bad interpreter: No such file or directory
```

使用以下命令进行修改：

```bash
vi backup.sh
:set fileformat=unix
:wq
```

## 四. 添加或修改定时任务

命令行中输入 `crontab -e`，在文件内添加或修改以下内容：

```text
30 7 * * * /data/backup/backup.sh
```

保存退出即可生效，无需重启 crontab 服务。crontab 文件格式如下：

| 列数 | 说明 |
| :--- | :--- |
| 1 | 分钟，值为1~59，每分钟用 \* 或者 \*/1 表示，整点分钟为00或0，下同 |
| 2 | 小时，值为0~23 |
| 3 | 日，值为1~31 |
| 4 | 月，值为1~12 |
| 5 | 星期，值为0~6（0表示星期天） |
| 6 | 要执行的脚本 |

