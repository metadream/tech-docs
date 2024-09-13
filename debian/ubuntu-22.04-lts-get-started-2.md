# Ubuntu 22.04 LTS 新手上路定制篇

> 定制篇主要包括图形界面的美化与使用习惯上的调整。

## 1. Gnome 桌面系统扩展

### 1.1 gnome-tweaks

如果仅需对桌面系统进行少量调整，可安装优化工具：

```
sudo apt install gnome-tweaks
```

### 1.2 gnome-shell-extensions

如果希望进行更加全面的调整，可安装扩展程序。

```
sudo apt install gnome-shell-extensions
```

Ubuntu 本身自带了三个扩展
appindicator、desktop-icons、dash-to-dock，可在扩展程序中查看并设置。

### 1.3 gnome-shell-extension-manager

扩展管理程序用于搜索、添加、删除扩展。

```
sudo apt install gnome-shell-extension-manager
```

### 1.4 浏览器扩展管理插件

Chrome 浏览器扩展管理插件也可以管理扩展。首先访问 https://extensions.gnome.org
网站根据提示安装插件，然后打开终端安装本地主机连接器（用于连接浏览器插件）。

```
sudo apt install chrome-gnome-shell
```

### 1.5 下载扩展安装包

遗憾的是，1.3 和 1.4
在我的系统都存在问题，前者搜索扩展时报错并关闭，后者提示本机连接器版本不够，所以只能采用第三种方式：在
https://extensions.gnome.org
网站上下载安装包，手动安装扩展。下载安装包并解压后，打开 `metadata.json`
文件查看 uuid：

```
"uuid": "dash-to-panel@jderose9.github.com"
```

将 `metadata.json` 所在文件夹命名为 uuid 的值，然后将该文件夹复制到
`/home/xehu/.local/share/gnome-shell/extensions`
目录下，即可在扩展程序中找到并设置该扩展。

### 1.6 dconf-editor

还有一种更为晦涩的调整方式：安装 `dconf-editor` 工具（类似于 Windows
注册表），需要对每个参数有足够的了解。

```
sudo apt install dconf-editor
```

无论哪种方式，均可能造成系统的不稳定，慎用！

## 1.7 dock-to-panel

`dock-to-panel`
包含非常丰富的界面设置，尤其是将顶部栏和程序坞合并功能对我来说至关重要，因此尽管在我的系统上切换到某个设置时屡屡司机也必须安装（只要不切换到这个设置就没事）。

## 1.8 custom-hot-corner

用于设置四个热角触发的动作。可惜在我的系统中无法使用。

## 1.9 修改登录界面背景

22.04
版本的登录界面为深灰色，以上所有扩展工具似乎都无法修改，网上找到一个独立的小工具，安装和使用方式如下。

```
wget -q https://raw.githubusercontent.com/PRATAP-KUMAR/ubuntu-gdm-set-background/main/ubuntu-gdm-set-background && chmod +x ubuntu-gdm-set-background
sudo ./ubuntu-gdm-set-background --image /home/xehu/Pictures/Wallpapers/Yosemite-Color-Blur.png
```

重启即可生效。更多设置方法参见
https://github.com/PRATAP-KUMAR/ubuntu-gdm-set-background

## 1.10 修改默认文件列表栏目

文件管理器以列表形式展示时，默认只显示两列：大小和修改时间，如果想显示更多栏目需要每个文件夹单独设置，非常耗时。前面提到的
`dconf-editor` 可一次性修改系统默认显示栏。打开 dconf-editor，定位到
`org/gnome/nautilus/list-view/default-visible-columns`，将 `Custom value`
修改为：

```
['name', 'size', 'type', 'date_created', 'date_modified', 'starred']
```

列的可能名称为 name, size, type, owner, group, permissions, detailed_type,
where, date_modified_with_time, date_modified, date_accessed, date_created,
recency, starred。

## 1.11 暂未找到解决方法的调整

- 修改应用程序列表背景（22.04 同样将背景改为深灰色且无法调整）
- 隐藏用户登录列表，允许直接输密码（默认情况下需要点击一个用户再输密码）
