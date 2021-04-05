---
layout: post
title: 2021年04月05日 ElasticSearch Head插件安装
date: 2021-04-05
description: "ElasticSearch Head插件安装"
tags: ElasticSearch 插件 Head ES
---   

### ES head插件安装

#### 下载 head插件
```sh
# 进入安装目录
cd /run/media/duanbin/programs/opt/elastic_head
# 下载
wget https://github.com/mobz/elasticsearch-head/archive/refs/heads/master.zip
# 解压
unzip master.zip
# 进入解压目录
cd elasticsearch-head-master
# 安装（确保安装node）
npm install
# 启动
npm run start
```

#### 浏览器打开 http://127.0.0.1:9100

![Head插件启动](https://gitee.com/dbin0123/picgo/raw/master/image/20210405122441.png)


#### ES配置跨域访问

在elasticsearch中启用CORS
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```

需要身份认证链接
```
http:/127.0.0.1:9100?auth_user=elastic&auth_password=pwd
```

- base_uri 强制elasticsearch-head连接到特定节点。
- dashboard实验性功能，以适合仪表板/散热器的模式打开elasticsearch-head。接受一个参数dashboard=cluster
- auth_user向http请求添加基本身份验证凭据（需要elasticsearch-http-basic插件或反向代理）
- auth_password如上所述的基本身份验证密码（注意：没有其他安全层，密码将通过网络以明文形式发送）
- lang 强制elasticsearch-head使用指定的ui语言（例如：en，fr，pt，zh，​​zh-TW，tr，ja）