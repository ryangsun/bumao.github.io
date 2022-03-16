---
title: Gitlab持续集成详细配置与工作原理
date: 2022-03-16 16:19:11
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

## Gitlab Runner 窍门
### 解决gitlab Runner 执行 job时反复拉取镜像的问题

编辑 `/etc/gitlab-runner/config.toml`

找到 shm_size = 0 段
添加
```
pull_policy = "if-not-present"
```

### 每次都要cache的内容可直接放到配置文件中
因为在docker方式中，每次job拉起镜像，都是崭新的，所以在构建Java项目时，`.m2`文件夹不存在，导致每次都需要重新下载全量jar包。
的确可以通过 gitlab-ci.yml中的cache段来解决，但每次写实在太麻烦，所以这个配置，也可以通过编辑 `/etc/gitlab-runner/config.toml`来解决。

找到 `volumes = ["/cache"]`段，修改为
```
volumes = ["/cache","/dockerdata/gitlab-runner-java01/root/.m2:/root/.m2"]
```
其中`"/dockerdata/gitlab-runner-java01/root/.m2:/root/.m2"` 与docker -v参数的传值类似，冒号前面是宿主机路径，后面是容器内路径。




## 基本原理
- 不管我们用什么方式安装的gitlab，本身并不具备cicd功能，需要安装配套的`Runner`来具体做事。
- Docker方式部署的runner也不直接做打包发布的事情，而是通过定义不同环节用哪个docker容器来完成任务，这种方式下的runner，其实就是个管控器。
- 何时触发构建动作、由谁执行构建动作、如何构建，需要在项目根目录下的 `gitlab-ci.yml` 来定义。

## 运行方式
### 触发
gitlab的 pipeline既可以自动触发，也可以通过gitlab界面的cicd相关页面手动触发，这取决于在gitlab-ci.yml中如何配置。

- 手动触发：`gitlab项目页面` -> `左侧"CI/CD"` -> `右侧按钮"Run Pipeline"`
- 自动触发：`gitlab-ci.yml` 中，每个job的 `only` 段，方式非常灵活，可按照 分支提交动作、文件修改动作 等等方式。

#### only案例，修改文件触发构建

作为开发人员，最方便的莫过于修改本地文件然后提交了，以下方式，只需要在项目根目录创建两个文件`dev_ver`和`test_ver`，分别对应自动部署到开发版和测试版。
当gitlab获取到提交请求时，会自动扫描gitlab-ci.yml文件，如果对应的文件有修改，则进行对应的构建和发布操作。
```
  only:
    changes:
      - dev_ver
      - test_ver
```
#### only案例，手动触发

如果想通在gitlab网页界面上操作，可采用这种变量传值方式：
```
  only:
    variables:
      - $RELEASE == "dev"
      - $RELEASE == "test"
```
此时，`gitlab项目页面` -> `左侧"CI/CD"` -> `右侧按钮"Run Pipeline"`，在对应的页面中变量栏key输入 `RELEASE`,value栏输入`dev`，则会触发此规则。

### 阶段
以下信息描述了两个阶段 `prepare`和`deploy`阶段
```
stages:
- prepare
- deploy
```
当然，可以制定更多的阶段（Job），对于docker方式的Runner而言，每个阶段都会临时启动一个docker来执行对应的任务。

### 执行
在每个执行阶段中，我们需要定义：
- 触发规则 only
- 采用的镜像 image
- 运行的脚本 script
- 缓冲（为了下一环节能用到上一环节的作业成果） cache

#### 执行案例
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