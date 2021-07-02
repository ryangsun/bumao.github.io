---
title: 当mysql遇到ssl
date: 2021-05-17 15:19:11
tags: [mysql,java]
categories: [未解之谜]
keywords: KeyCastOW 录屏技巧 按键显示
comments: true
---
SpringBoot 连接提示 Communications link failure
```
com.mysql.cj.jdbc.exceptions.CommunicationsException: Communications link failure

The last packet sent successfully to the server was 0 milliseconds ago. The driver has not received any packets from the server.
	at com.mysql.cj.jdbc.exceptions.SQLError.createCommunicationsException(SQLError.java:174) ~[mysql-connector-java-8.0.20.jar:8.0.20]
	at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:64) ~[mysql-connector-java-8.0.20.jar:8.0.20]
	at com.mysql.cj.jdbc.ConnectionImpl.createNewIO(ConnectionImpl.java:836) ~[mysql-connector-
```
具体的错误原因没找到，只从网上搜到了这个：
原因是MySQL在高版本需要指明是否进行SSL连接
````
Establishing SSL connection without server’s identity verification is not recommended. According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn’t set. For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
不建议在没有服务器身份验证的情况下建立SSL连接。根据MySQL 5.5.45+、5.6.26+和5.7.6+的要求，如果不设置显式选项，则必须建立默认的SSL连接。需要通过设置useSSL=false来显式禁用SSL，或者设置useSSL=true并为服务器证书验证提供信任存储。
````
用参数 useSSL=true 进行尝试，发现还是报一样的错误，当使用 useSSL=false 时，就可以进行连接了。也就是
jdbc:mysql://192.168.221.201:3306/jdbc?useSSL=false
不过为什么要这样连接，暂时没弄清楚，只知道时版本和兼容的原因。

