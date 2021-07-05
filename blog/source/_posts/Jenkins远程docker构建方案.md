---
title: Jenkins远程docker构建方案
date: 2021-07-05 10:19:11
tags: [docker,持续集成,ci/cd,devops,dind,jenkins]
categories: [CI/CD]
comments: false
---

## 背景介绍

Jenkins，包括idea，有很多集成方案构建docker，但是因为网络结构、构建环境等客观原因，集成性的方案并不能满足需求。如：

- Jenkins权限限制；
- Jenkins已经是docker化部署，无法嵌套集成方案；
- 构建环境不唯一，需要多个docker环境分别构建；
- Jenkins端无私仓权限，无法把构建好的docker部署到指定位置。

这时，基于DinD的构建方案就非常方便了。



## 安装与部署DinD容器

容器的Dockerfile见这里：

https://github.com/ryangsun/centos7-dind

安装和部署介绍见

[用DinD方式构建docker]: https://blog.bumao.com/2021/07/05/%E7%94%A8DinD%E6%96%B9%E5%BC%8F%E6%9E%84%E5%BB%BAdocker/



## docker构建环境搭建

### 免密登录

1、将jenkins对应使用的私钥copy到dind容器的对应文件中：

```
scp -P <DinD容器端口> <Jenkins私钥文件> <DinD容器>:/home/jenkinsbuild/.ssh/....
```

2、Jenkins创建新的远程主机连接

3、Jenkins SSH Publishers配置：

![image-20210705115301268](https://i.loli.net/2021/07/05/sJ3YfLiAhFHDrzN.png)

第一步：针对一般的java项目，需要将jenkins打包好的jar文件和对应的Dockerfile拷贝到DinD容器。上图中，我们开了两个Transer Set，一个拷贝了.jar文件，一个拷贝了项目对应的Dockerfile。

第二步：远程执行命令构建docker:

```
#进入构建目录并构建Docker
cd ~/ci-jenkins/${JOB_NAME} && docker build -t <私仓IP>:<私仓端口>/mytest/${JOB_NAME}:${BUILD_ID} .
#将构建好的docker推到私仓
docker push <私仓IP>:<私仓端口>/mytest/${JOB_NAME}:${BUILD_ID}
#推私仓成功后，删除本地的docker image
docker rmi <私仓IP>:<私仓端口>/mytest/${JOB_NAME}:${BUILD_ID}
```

第三步：触发后续操作，如Rancher构建等。
