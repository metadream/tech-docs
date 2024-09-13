# Win10 初始化偏好设置

## 一. 系统设置

1. 系统设置：通知，电源，隐私，Cleartype...
2. 设置资源管理器启动时打开“此电脑”：文件夹选项 - 常规
3. 隐藏“此电脑”中的7个文件夹：执行 `hide-folders-in-this-computer.reg`
4. 删除多余的快速访问
5. 修改桌面路径指向
6. 映射网络驱动器NAS目录
7. 删除时显示确认提示：回收站属性

## 二. 软件设置

1. 安装常用字体：Inconsolata, YaHei-Consolas-Hybrid, YaHei-Mono
2. 设置默认浏览器：CentBrowser
3. 设置默认图片查看器：MassiGra
4. 设置默认播放器：PotPlayer
5. 设置默认解压工具：Bandizip

## 三. 安装其他软件

1. 安装迅雷极速版（防升级）

   1.1 将 `Data/ThunderPush` 目录设为只读，并在安全选项拒绝所有权限访问

   1.2 删除 `Program/XLLiveUD.exe`，复制一份 `thunder.exe` 更名为 `XLLiveUD.exe`

2. 安装 Office
3. 安装 JDK
4. 安装 Github Desktop
5. 设置 Eclipse
6. 关联 SublimeText：执行 sublime\_text\_settings.bat

## 四. 禁用不必要的服务

* Connected User Experiences and Telemetry
* Contact Data\_37c1e5
* Human Interface Device Service
* IP Helper
* Microsoft Store 安装服务
* Network Connection Broker
* Print Spooler
* PrintWorkflow\_37c1e5
* Program Compatibility Assistant Service
* Quality Windows Audio Video Experience
* Shell Hardware Detection
* Touch Keyboard and Handwriting Panel Service
* Web 帐户管理器
* Windows 推送通知系统服务
* Windows 许可证管理器服务
* Xbox Live 网络服务
* 剪贴板用户服务\_37c1e5
* 蓝牙音频网关服务
* 连接设备平台用户服务\_37c1e5
* 数据使用量
* 同步主机\_37c1e5
* 无线电管理服务

## 五. 删除临时文件

Windows.old, $SysReset, UserTemp

## 六. 系统备份

设置 - 更新与安全 - 备份

