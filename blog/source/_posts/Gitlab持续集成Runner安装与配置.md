---
title: Gitlib持续集成Runner的安装与配置
date: 2022-03-15 16:19:11
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


首先，gitlab不做持续集成的事儿，只是发送指令和接收结果，实际做cicd的是Runner。安装和配置runner有集中方式，这里记录两种：

## docker方式，非常轻便灵活

展开容器
```
docker run -d \
--name gitlab-runner-docker \
--restart always \
-v /dockerdata/gitlab-runner-docker/etc/gitlab-runner:/etc/gitlab-runner \
-v /var/run/docker.sock:/var/run/docker.sock \
gitlab/gitlab-runner:latest
```
接入对应的gitlab
```
docker exec -it gitlab-runner-docker gitlab-runner register
```
接着会询问gitlab的地址、token ，这两个信息，可以在我们自己gitlab的 admin->概览->Runner中查询到。

之后会询问 这个runner的名字和tag，tag这里要注意，会影响到实际cicd中的调用，因为跑自动化的时候是可以通过tag来指定runner的。

交互式提问结束后，在gitlab的Runner页面，应该有对应的Runner注册上来了。

## Centos中通过ssh连接

添加yum源
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
```

安装runner
```
yum install gitlab-ci-multi-runner
```
向GitLab-CI注册runner
```
gitlab-ci-multi-runner register
```
后面的步骤跟docker方式类似，gitlab的路径、token，连接方式选ssh，然后把本机的IP、ssh端口、账号密码录入进去。

## 配置过程（以docker方式为例）
```
# docker exec -it gitlab-runner-java01 gitlab-runner register
Runtime platform                                    arch=amd64 os=linux pid=24 revision=5316d4ac version=14.6.0
Running in system-mode.                            
                                                   
Enter the GitLab instance URL (for example, https://gitlab.com/):
<这里输入gitlab私仓的url>
Enter the registration token:
<这里输入runner的token>
Enter a description for the runner:
[007fa02670d1]: <给这个runner起个名字，会显示在gitlab中>
Enter tags for the runner (comma-separated):
<这里输入tag，跑任务的时候可以通过 tags 来指定>
Registering runner... succeeded                     runner=rANP_dLs
Enter an executor: docker, docker-ssh, parallels, shell, virtualbox, docker+machine, custom, ssh, docker-ssh+machine, kubernetes:
<运行方式，这里写 docker>
Enter the default Docker image (for example, ruby:2.6):
<默认运行容器，如果在job中不指定容器，默认采用的运行容器，这里我添了 tico/docker>
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded! 
```
