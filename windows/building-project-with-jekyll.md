# Jekyll 环境搭建

- 强烈建议所有步骤均在管理员身份下进行
- 强烈建议不要更改默认安装路径

## 一. 安装 Ruby

到 https://rubyinstaller.org/downloads 网站下载最新 Ruby 安装程序及开发包，例如
Ruby+Devkit 2.6.3-1 (x64).exe；安装过程中需选择 MSYS2 development
toolchain；安装完成后勾选 Run 'ridk
install'，在弹出界面选第3项，结束后会提示再做一次123选择，此时直接退出即可。

使用 `ruby -v` 和 `gem -v` 命令确认相关程序是否安装成功。

## 二. 安装 Jekyll

打开命令行输入 `gem install jekyll`，安装完成后输入 `jekyll -v`
确认是否成功。如果上一步骤更改了 Ruby 的默认安装路径，则此步骤很有可能失败。

命令行继续输入 `gem install jekyll-paginate` 安装分页插件。
