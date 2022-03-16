---
title: Gitlib持续集成Runner的安装与配置
date: 2022-03-15 16:19:11
tags: [gitlab,docker,持续集成,ci/cd,devops]
categories: [CI/CD]
comments: false
---

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

