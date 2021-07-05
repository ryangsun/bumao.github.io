---
title: Harbor的安装与配置
date: 2021-07-03 15:19:11
tags: [harbor,docker,持续集成,ci/cd,devops]
categories: [CI/CD]
comments: false
---

## Harbor简介

Harbor 是由 VMware 公司中国团队为企业用户设计的 Registry server 开源项目，包括了权限管理(RBAC)、LDAP、审计、管理界面、自我注册、HA 等企业必需的功能，同时针对中国用户的特点，设计镜像复制和中文支持等功能。

![](https://static.oschina.net/uploads/space/2016/1117/163439_2f9X_2671340.png)

作为一个企业级私有 Registry 服务器，Harbor 提供了更好的性能和安全。提升用户使用 Registry 构建和运行环境传输镜像的效率。Harbor 支持安装在多个 Registry 节点的镜像资源复制，镜像全部保存在私有 Registry 中， 确保数据和知识产权在公司内部网络中管控。另外，Harbor 也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

### Harbor的官网

```
https://goharbor.io/
```

### Harbor的github地址

```
https://github.com/goharbor/harbor
```

(已经有15.2K的star了)

## 安装Harbor

### 前置条件

宿主机已安装好docker和docker-compose，具体安装方案这里不不展开。

### 下载&解压

Harbor有两个大版本，一是1.x，另外是2.x，据说2.x优化了很多，所以这里采用2.3.0这个版本，下载离线安装包。

```
wget https://github.com/goharbor/harbor/releases/download/v2.3.0/harbor-offline-installer-v2.3.0.tgz
tar zxvf harbor-offline-installer-v2.3.0.tgz
cd harbor
```

### 修改配置文件

将配置模板copy一份

```
cp harbor.yml.tmpl harbor.yml
```

编辑配置模板

```
vim harbor.yml
```

主要修改以下几个地方

```
# The IP address or hostname to access admin UI and registry service.
# DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname: <你的harbor的域名或者IP(不加端口号)>

# http related config
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: <http协议监听的端口>

# https related config
https:
  # https port for harbor, default is 443
  port: <https协议监听的端口>
  # The path of cert and key files for nginx
  certificate: <.crt证书文件的路径>
  private_key: <.key证书文件的路径>
```

保存后执行配置脚本，使刚才的配置生效

### 执行配置脚本

```
./prepare
```

### 安装

```
./install.sh
```

如果之后修改配置再次启动，此步骤可以省略。

安装成功后，即可在 https://youip 上访问harbor了，初始帐号为 admin 初始密码为 Harbor12345 。

## 一些常用命令

### 启动Harbor

```
docker-compose up -d
```

### 关闭Harbor

```
docker-compose down -v
```

### 修改配制后需要重新关停和开启Harbor

```
./prepare #执行配置脚本
docker-compose down -v #关停Harbor
docker-compose up -d #启动Harbor
```