---
title: Harbor SSL避坑指南
date: 2021-07-03 15:19:11
tags: [harbor,docker,持续集成,ci/cd,devops]
categories: [CI/CD]
comments: false
---

Harbor的安装与配置请参考这里

[Harbor的安装与配置]: https://blog.bumao.com/2021/07/03/Harbor%E7%9A%84%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE/

Docker默认情况下，push和pull对应的仓库均需要ssl加密，如果我们在内网采用Harbor作为私仓，有两种方案可以解决：

- Harbor开启ssl
- docker侧修改配置文件，并且需重启docker服务

实际情况是，docker因为承载的服务较多，无法选择重启方案，那么就必须采用第一种方案。这种方案：

- harbor侧开启ssl
- 对应的docker服务侧需要拷贝对应的签名证书

## Harbor开启ssl

### 创建ssl证书

```
创建一个证书文件夹
mkdir harbor_cert && cd harbor_cert
# 创建ca时，根据提示输入 IP地址
openssl req  -newkey rsa:4096 -nodes -sha256 -keyout ca.key -x509 -days 365 -out ca.crt
openssl req  -newkey rsa:4096 -nodes -sha256 -keyout server.key -out server.csr
# 把IP地址写入到文件中
echo subjectAltName = IP:xx.xx.xx.xx > extfile.cnf
# 生成 cr
openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out server.crt
```

### 配置Harbor

```
cd harbor2.3/
vim harbor.yml

#修改对应的配置项目：
# https related config
https:
  # https port for harbor, default is 443
  port: 10443
  # The path of cert and key files for nginx
  certificate: /root/harbor_cert/server.crt
  private_key: /root/harbor_cert/server.key
#保存

#使配置生效
./prepare

#重启Harbor
docker-compose down -v
docker-compose up -v
```

### 拷贝对应签名文件到docker服务侧

```/
scp server.crt  root@<docker侧主机>:/etc/docker/certs.d/<Harbor主机IP>:<harbor主机端口>/
scp ca.crt  root@<docker侧主机>:/etc/docker/certs.d/<Harbor主机IP>:<harbor主机端口>/
scp server.key root@<docker侧主机>:/etc/docker/certs.d/<Harbor主机IP>:<harbor主机端口>/
```

这里注意的是：如果Harbor侧ssl的监听端口为443，则docker侧的路径为

```
/etc/docker/certs.d/<Harbor主机IP>/
```

否则为

```
/etc/docker/certs.d/<Harbor主机IP>:<harbor主机端口>/
```

### 测试

```
docker login <Harbor主机IP>:<harbor主机端口>
```

这样，无需重启docker服务，即可实现内网harbor私仓的正常使用。



## 相关的docker对应的官方文档 

- https://docs.docker.com/registry/insecure/
- https://docs.docker.com/engine/security/certificates/