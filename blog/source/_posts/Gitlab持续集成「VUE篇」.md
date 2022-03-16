---
title: Gitlab持续集成「VUE篇」
date: 2022-03-16 16:22:11
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


java、vue等项目需要编译，所以，至少有两个job，一个是用 nodejs 镜像打dist包，一个是发布到对应位置。

发布节点用到的镜像`centos7-sshpass:1.0.0`详见 《[为Gitlab持续集成打造一个部署用的docker](/2022/03/16/为Gitlab持续集成打造一个部署用的docker/)》

### 案例
```
stages:
  - prepare
  - deploy

prepare:
  stage: prepare
  image: 'node:16.14.0'
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
      - node_modules/
      - dist/
  script:
    - npm install
    - npm run build:prod
    - ls dist/ -lah
  tags:
    - runner_test
  only:
    changes:
      - dev_ver
      - test_ver
deploy_dev:
  stage: deploy
  image: 'centos7-sshpass:1.0.0'
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
      - dist/
  script:
    - ls dist/
    # 清除原有
    - sshpass -p abc ssh user@localhost -o StrictHostKeychecking=no "rm -fr /path/to/pub/*"
    # 拷贝dist包
    - sshpass -p abcc scp -o StrictHostKeyChecking=no -p -r dist/*.* user@localhost:/path/to/pub/
  tags:
    - runner_test
  only:
    changes:
      - dev_ver

deploy_test:
  stage: deploy
  image: 'centos7-sshpass:1.0.0'
  cache:
    key: "$CI_COMMIT_REF_NAME"
    paths:
      - dist/
  script:
    - ls dist/
    # 清除原有
    - sshpass -p abc ssh user@localhost -o StrictHostKeychecking=no "rm -fr /path/to/pub/*"
    # 拷贝dist包
    - sshpass -p abcc scp -o StrictHostKeyChecking=no -p -r dist/*.* user@localhost:/path/to/pub/
  tags:
    - runner_test
  only:
    changes:
      - test_ver
```

以上这个案例，定义了两个环节 `prepare`和`deploy`，有三个job：
- `prepare:` 在dev_ver或test_ver被修改时会被触发
- `deploy_dev:` 在dev_ver被修改时会被触发
- `deploy_test:` 在test_ver被修改时会被触发

那么，如果：
- 项目根目录下的 `dev_ver`被修改，则会触发 `prepare:` 和 `deploy_dev:`
- 项目根目录下的 `test_ver`被修改，则会触发 `prepare:` 和 `deploy_test:`