---
title: Docker 简单使用
date: 2022-9-12 10:32	
categories:
  - Linux
  - Docker
---



## Linux防火墙

```sh
systemctl status firewalld #查看状态
systemctl stop firewalld #关闭
systemclt disable firewalld #关闭开机自启
```

## docker常用操作

### docker安装

[server安装](https://docs.docker.com/engine/install/#server)

### docker开机自启

```sh
systemctl enable docker.service
```

### 容器开机自启

```sh
--restart=always #开机自启
```

### 安装RabbitMQ

> rabbitmq:management有管理界面

```sh
docker pull rabbitmq:management
```

```sh
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq1 rabbitmq:management
```