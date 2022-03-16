---
title: Gitlab持续集成「PHP篇」
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


PHP项目不要要编辑，一个job可以搞定。

发布节点用到的镜像`centos7-sshpass:1.0.0`详见 《[为Gitlab持续集成打造一个部署用的docker](/2022/03/16/为Gitlab持续集成打造一个部署用的docker/)》

### 案例
```
stages:
  - deploy

deploy_dev:
  stage: deploy
  image: 'centos7-sshpass:1.0.0'
  script:
    - ls ./ -lah
    # 清除原有
    - sshpass -p abc ssh -p 1234 -o StrictHostKeychecking=no root@localhost "rm -fr /path/to/pub/*"
    # 拷贝文件包
    - sshpass -p abc scp -o StrictHostKeyChecking=no -P 1234 -r ./* root@localhost:/path/to/pub/
  tags:
    - runner_test
  only:
    changes:
      - dev_ver


deploy_test:
  stage: deploy
  image: 'centos7-sshpass:1.0.0'
  script:
    - ls ./ -lah
    # 清除原有
    - sshpass -p abc ssh -p 1234 -o StrictHostKeychecking=no root@localhost "rm -fr /path/to/pub/*"
    # 拷贝文件包
    - sshpass -p abc scp -o StrictHostKeyChecking=no -P 1234 -r ./* root@localhost:/path/to/pub/
  tags:
    - runner_test
  only:
    changes:
      - dev_ver
```

如果：
- 项目根目录下的 `dev_ver`被修改，则会触发 `deploy_dev:`
- 项目根目录下的 `test_ver`被修改，则会触发 `deploy_test:`