---
title: 用DinD方式构建docke用DinD方式构建dockev
date: 2021-07-05 15:19:11
tags: [docker,持续集成,ci/cd,devops,dind]
categories: [CI/CD]
comments: false
---



## Docker in Docker(dind)

Docker in Docker(dind) image可以用于Jenkins做build， 可以在一台机器上起多个docker image，每个image里面安装不同的第三方，形成不同的build环境，然后可以将待编译的代码SCP或GIT过去，用指定账号SSH来进行编译，达到一个机器多种编译环境的效果，提高了效率。Docker in Docker(dind) 镜像基于centos7，可用于jenkins打包编译：

- 基础镜像: centos7 
- 内置用户：jenkinsbuild, 密码: jenkinsbuild, 构建文件夹: /home/jenkinsbuild/ci-jenkins
- 采用ssh login方式登录

## 构建和使用镜像

### 构建镜像

```
./build-centos7-dind
```

### 展开容器

```
docker run -d -p 22 \
--name=centos7-dind \
-e TZ=Asia/Shanghai \
-e LANG=en_US.UTF-8  \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
centos7-dind:1.0.0
```

### 将打代码copy到容器

```
scp -P \<port\> -r \<buildsourcecode\> jenkinsbuild@localhost:/home/jenkinsbuild/ci-jenkins/
```

### ssh 登录进去

```
ssh -p \<port\> jenkinsbuild@localhost  
```

### 免密登录

```
将对对应公钥copy到 /home/jenkinsbuild/ 对应位置即可。
```



## git地址

https://github.com/ryangsun/centos7-dind
