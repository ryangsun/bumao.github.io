---
title: ARM芯片的node-sass支持
date: 2022-04-08 16:19:11
tags: [ARM,M1,node-sass,vue]
categories: [VUE]
comments: false
---

用M1芯片的博友越来越多了，但node-sass，在高版本的nodejs上一直不能正常使用，下面来说说如何在ARM芯片上把node-sass跑起来。

直奔主题： 自打M1问世，node-sass的作者就一直没有想着更好的支持arm，所以：降级！

我是在mac M1芯片的docker中进行测试的，因为宿主机的环境有些复杂。

### 开docker容器

```
docker run -ti \
-p 20081:8080 \
-v /Users/ryan/dockerdata/uni_node14:/root/uni \
--name uni_node14 centos:7 \
bash
```

映射了8080端口和一个路径，这样方便文件操作和访问。

### 在docker中安装nodejs14

```
curl -sL https://rpm.nodesource.com/setup_14.x | bash -
yum list nodejs --showduplicates | sort -r   #列出所有版本
# 默认安装最新版本,安装指定版本
yum install nodejs-14.16.0 -y
```

这里我用了nodejs 14.16.0

### 安装node-sass

**因为sass的安装需要编译环境，所以先yum装对应的包**

```
yum install gcc cmake automake autoconf libtool make gcc-c++ kernel-devel
```

**安装node-sass**

```
npm install node-sass@4.14.0 --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
```

注意这里的版本 `4.14.0` 。



至此，ARM上node-sass安装完毕。
