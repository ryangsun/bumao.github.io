---
title: 为Gitlab持续集成打造一个部署用的docker
date: 2022-03-16 16:20:11
tags: [gitlab,docker,持续集成,ci/cd,devops]
categories: [CI/CD]
comments: false
---
《[Gitlab持续集成Runner的安装与配置](/2022/03/15/Gitlab持续集成Runner安装与配置/)》

《[Gitlab持续集成详细配置与工作原理](/2022/03/16/Gitlab持续集成详细配置与工作原理/)》

《[为Gitlab持续集成打造一个部署用的docker](/2022/03/16/为Gitlab持续集成打造一个部署用的docker/)》

《[Gitlab持续集成「springboot篇」](/2022/03/16/Gitlab持续集成「springboot篇」/)》

《[Gitlab持续集成「PHP篇」](/2022/03/16/Gitlab持续集成「PHP篇」/)》

《[Gitlab持续集成「VUE篇」](/2022/03/16/Gitlab持续集成「VUE篇」/)》


因为docker化部署的Runner，每一部都需要在容器中完成，针对没有完全使用k8等容器化运维的项目或者项目构建过程中需要做额外操作的ci/cd流程，使用官方容器就比较尴尬：

- 打包好的目标文件，在maven、nodejs容器因为没有对应的命令，无法上传到指定位置
- ci/cd过程复杂，需要跟其他服务交互

这种情况，咱们就得自己包docker image来给Runner用。

## ceontos-sshpass

为了应对以上问题，笔者自己包了个发版的docker，地址在 [https://github.com/ryangsun/centos7-sshpass](https://github.com/ryangsun/centos7-sshpass)

### 构建镜像
```
git clone https://github.com/ryangsun/centos7-sshpass.git
chmod 700 build-image.sh
./build-image.sh
```
默认创建的docker镜像为 `centos7-sshpass:1.0.0`，233M大小。