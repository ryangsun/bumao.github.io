---
title: Gitlab持续集成「springboot篇」
date: 2022-03-16 16:21:11
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


java、vue等项目需要编译，所以，至少有两个job，一个是用 maven镜像打jar包，一个是发布到对应位置。

发布节点用到的镜像`centos7-sshpass:1.0.0`详见 《[为Gitlab持续集成打造一个部署用的docker](/2022/03/16/为Gitlab持续集成打造一个部署用的docker/)》

### 案例
```
# 定义了两个流程
stages:
  - prepare
  - deploy

prepare:    # job的名称，可以随便起
  stage: prepare    #这里要跟 stages 里面定义的保持一致
  image: 'maven:3.6.3-openjdk-8'    #用哪个镜像
  cache:    # 缓存
    policy: pull-push   #缓存方式 有 pull push pull-push
    key: ${CI_COMMIT_REF_SLUG}  #按照分支进行缓冲
    paths:
      - target/     #把target目录缓冲下来给下一步用
  script:   #执行脚本
    - mvn -v && java -version
    - mvn clean install
  tags:
    - java01    #对应的Runner的TAG，多个runner可以用同样的tag，到时候gitlab来分配具体用哪个runner来执行
  only:
    changes:    此步骤（job）通过文件修改来触发，下面定义的意思是，这两个文件其中一个有修改即触发执行
      - dev_ver
      - test_ver


deploy_dev:     #往dev版发布
  stage: deploy     #这里要跟 stages 里面定义的保持一致
  image: 'centos7-sshpass:1.0.0'
  cache:
    policy: pull  #只读不写
    key: ${CI_COMMIT_REF_SLUG}  #按照分支进行缓冲
    paths:
      - target/
  script:
    - ls ./target/
    # 拷贝jar包
    - sshpass -p abcd scp -o StrictHostKeyChecking=no -P 1234 target/demo-0.0.1-SNAPSHOT.jar root@localhost:/app/demo-0.0.1-SNAPSHOT.jar
    # 运行脚本
    - sshpass -p abcd ssh -p 1234 root@localhost "/app/jenkins_restart.sh demo-0.0.1-SNAPSHOT >> /app/jenkins_restart.log"
  tags:
    - java01
  only:
    changes:    #这里仅针对dev_ver的修改触发
      - dev_ver
    

deploy_test:    #往test版发包
  stage: deploy
  image: 'centos7-sshpass:1.0.0'
  cache:
    policy: pull  #只读不写
    key: ${CI_COMMIT_REF_SLUG}  #按照分支进行缓冲
    paths:
      - target/
  script:
    - 。。。。。。
  tags:
    - java01
  only:
    changes:
      - test_ver    #仅 文件 test_ver修改时候触发
```

以上这个案例，定义了两个环节 `prepare`和`deploy`，有三个job：
- `prepare:` 在dev_ver或test_ver被修改时会被触发
- `deploy_dev:` 在dev_ver被修改时会被触发
- `deploy_test:` 在test_ver被修改时会被触发

那么，如果：
- 项目根目录下的 `dev_ver`被修改，则会触发 `prepare:` 和 `deploy_dev:`
- 项目根目录下的 `test_ver`被修改，则会触发 `prepare:` 和 `deploy_test:`