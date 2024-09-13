# Win10 系统字体更换

## 一. 准备新字体

1. [下载新字体](https://pan.baidu.com/s/12WaKtZio8QsfIMCkF-LmLQ)（提取码：qt8w）并复制到 `C:\Fonts` 备用；
2. 可选步骤：将 `C:\Windows\Fonts` 系统字体中的微软雅黑 `msyh.ttc、msyhbd.ttc、msyhl.ttc` 和宋体 `simsun.ttc、simsunb.ttf` 备份；

## 二. 替换旧字体

1. 打开 Windows 10 设置，依次点击 “更新和安全 - 恢复 - 高级启动 - 立即重新启动”，进入界面后选择 “疑难解答 - 高级选项 - 命令提示符”；
2. 在命令行输入 `xcopy c:\fonts c:\windows\fonts`，出现是否替换字体提示时按 “A”；成功后，关闭命令提示符界面选择 “继续”，系统将会重启。

