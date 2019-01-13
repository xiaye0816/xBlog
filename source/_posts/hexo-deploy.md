---
title: 博客搭建
date: 2019-01-13 18:53:45
description: 本hexo博客的搭建部署流程
tags: hexo
categories: hexo
---





## 准备工作

### 环境
```shell
CentOS 7.2 64位
```

### node.js

```bash
#下载安装包
wget https://nodejs.org/dist/v7.0.0/node-v7.0.0-linux-x64.tar.xz

#解压缩
xz -d node-v7.0.0-linux-x64.tar.xz
tar -xvf node-v7.0.0-linux-x64.tar

#部署bin文件
ln -s /usr/local/node-v7.0.0-linux-x64/bin/node /usr/bin/node
ln -s /usr/local/node-v7.0.0-linux-x64/bin/npm /usr/bin/npm

#测试安装成功
node -v
npm -v

#切换国内npm数据源
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm config set registry https://registry.npm.taobao.org

ln -s /usr/local/node-v7.0.0-linux-x64/bin/cnpm /usr/bin/cnpm

#测试
cnpm -v
```

### git

```bash
#移除老版本
yum remove git

#安装git
yum install -y git

#测试安装成功
git --version
```

## 安装hexo

```bash
#hexo安装
npm install -g hexo-cli
ln -s /usr/local/node-v7.0.0-linux-x64/bin/hexo /usr/bin/hexo
hexo -v

#初始化hexo
mkdir xBlog
cd xBlog
hexo init

#git仓库
git init
git remote add origin https://github.com/xiaye0816/xBlog.git
git fetch --all
git reset --hard origin/master

#启动
hexo server -p 80

#修改主题
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
cnpm install hexo-renderer-pug --save
cnpm install hexo-renderer-sass --save

编辑_config.yml，将theme的值改为maupassant
```


## hexo操作

```bash
#创建新文章
hexo new [name]

#创建草稿
hexo new draft [name]

#发布草稿
hexo publish [name]
```

