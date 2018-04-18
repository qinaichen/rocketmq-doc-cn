---
title: 快速入门
---

这里我们将详细的介绍如何在本机安装部署`RocketMQ`，然后进行发送和接收消息

### 系统要求

环境要求：

1. 64位操作系统，推荐使用`Linux`、`Unix`、`Mac`
2. 64位 Jdk 1.8以上版本
3. Maven3.2.x
4. Git


#### 下载编译发行版

点击 [这里](https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.2.0/rocketmq-all-4.2.0-source-release.zip)
下载`4.2.0`发行版源码，或者也可以[下载](http://rocketmq.apache.org/release_notes/release-notes-4.2.0/)发布包

执行下面的命令解压源码包，然后进行编译

``` shell
unzip rocketmq-all-4.2.0-source-release.zip
cd rocketmq-all-4.2.0/
mvn -Prelease-all -DskipTests clean install -U
cd distribution/target/apache-rocketmq
```

#### 启动Name Server

```
nohup sh bin/mqnamesrv &
```

查看日志

```
tail -f ~/logs/rocketmqlogs/namesrv.log
```

> The Name Server boot success...

#### 启动Broker
```
nohup sh bin/mqbroker -n localhost:9876 &
```

查看日志

```
tail -f ~/logs/rocketmqlogs/broker.log
```

> The broker[%s, 172.30.30.233:10911] boot success...

#### 发送、接收消息

在发送和接收消息之前，我们需要告诉客户端 name server 的位置。`RocketMQ`提供了多种实现方式，简单起见，我们设置环境变量`NAMESRV_ADDR`

```shell
> export NAMESRV_ADDR=localhost:9876
> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer

 SendResult [sendStatus=SEND_OK, msgId= ...

> sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer

 ConsumeMessageThread_%d Receive New Messages: [MessageExt...
```

### 停止服务

```shell
> sh bin/mqshutdown broker
The mqbroker(36695) is running...
Send shutdown request to mqbroker(36695) OK

> sh bin/mqshutdown namesrv
The mqnamesrv(36664) is running...
Send shutdown request to mqnamesrv(36664) OK
```
