# 如何在 CentOS 上增加交换分区

交换分区是Linux系统中当物理内存（RAM）容量不足时中使用的空间。如果系统需要更多的内存资源但RAM已满，则内存中的非活动页面将移至交换分区。本指南提供在 CentOS7 上添加交换分区所需的步骤。

## 一. 检查交换分区

通常，虚拟机默认情况下都没有启用交换分区，可以使用以下命令进行检查，如果没有任何输出，则表明没有任何交换分区。

```text
swapon --show
```

## 二. 创建交换文件

以下命令可以创建一个 1GB 的交换文件。交换文件不宜过大，通常在物理内存的2倍以内。创建完成后需设置正确的权限。

```text
# if: 输入文件；of: 输出文件；bs: 块大小；count: 块数
dd if=/dev/zero of=/swapfile bs=1024 count=1048572

chmod 600 /swapfile
```

## 三. 设置交换分区

使用`mkswap`命令将文件标记为交换分区。

```text
mkswap /swapfile
```

然后将收到与此类似的输出：

```text
Setting up swapspace version 1, size = 1048568 KiB
no label, UUID=83500984-b857-4ed3-b46c-c3b68c0e5272
```

## 四. 启用交换分区

```text
swapon /swapfile
```

键入以下命令验证交换分区是否可用：

```text
swapon --show
```

将收到与此类似的输出：

```text
NAME      TYPE  SIZE USED PRIO
/swapfile file 1024M   0B   -2
```

可以通过`free -h`命令再次检查输出。

```text
        total    used    free    shared    buff/cache  available
Mem:    587M     126M    48M     4.3M      412M        339M
Swap:   1.0G     0B      1.0G
```

## 五. 开机自动挂载分区

以上操作仅为当前会话创建交换分区，如果重启服务器，当前更改将消失。因此，还可以通过将交换文件添加到`/etc/fstab`中使设置永久生效。

```text
vi /etc/fstab
```

在文件末尾添加以下内容：

```text
/swapfile swap swap defaults 0 0
```

## 六. 设置交换参数

### 1. swappiness

`swappiness`参数允许配置系统内核将 RAM 数据交换到交换分区的频率，取值范围为0到100，值越小，内核使用交换分区的频率越低，值越高，则会尝试将更多数据进行交换。告诉系统不要过多依赖交换分区通常会系统运行更快。通过以下命令查看当前的可交换性值：

```text
cat /proc/sys/vm/swappiness
```

使用以下命令将值设置为10。

```text
sysctl vm.swappiness=10
```

可以通过将条目添加到`/etc/sysctl.conf`文件中来使该值永久生效。

```text
vi /etc/sysctl.conf
```

在末尾添加以下行：

```text
vm.swappiness=10
```

### 2. vfs\_cache\_pressure

`vfs_cache_pressure`设置内核回收用于 directory 和 inode cache 内存的倾向, 默认为100, 我们设为50。

```text
sysctl vm.vfs_cache_pressure=50
```

以上为临时设置, 重启失效。编辑`/etc/sysctl.conf`，在行尾添加以下内容可永久生效：

```text
vm.vfs_cache_pressure=50
```

## 七. 删除交换分区

删除交换分区是以上步骤的逆操作，首先将对应的分区条目从`/etc/fstab`文件中删除，再通过以下命令卸载交换分区：

```text
swapoff -v /swapfile
```

最后删除交换文件：

```text
rm /swapfile
```

