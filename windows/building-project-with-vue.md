# Vue 项目搭建

## 一. 安装 Nodejs

略。

## 二. 安装 Vue 命令

```bash
npm install -g @vue/cli
```

## 三. 创建 Vue 项目

```bash
# 创建过程中选择 babel 可兼容更多浏览器
vue create webapp
```

## 四. 手动安装插件

```bash
cd webapp
# 安装 vue-router
npm install vue-router --save
# 安装 vuex
npm install vuex --save
# 安装 axios
npm install axios --save
```

## 五. 常用命令

```bash
# 运行 Vue 服务
npm run serve
# 编译源文件
npm run build
# 清除缓存
npm cache clean --force
```

## 六. 修改 Vue 配置

在 `node_modules\@vue\cli-service\lib\options.js` 文件中，找到如下选项：

```javascript
# 修改编译目录
outputDir: dist
# 编译时不产生.map文件
productionSourceMap: false
# 修改启动端口
devServer: {}
```

## 七. 单一入口配置

```text
# ngnix.conf
location / {
  try_files $uri /index.html;
}
```

