# 使用Pub/Sub API进行消息发布/订阅
## 什么是Pub/Sub API
开发者经常使用消息队列等中间件产品（比如开源的Rocket MQ,Kafka,比如云厂商提供的AWS SNS/SQS）来实现消息的发布、订阅。发布订阅模式可以帮助应用更好的解耦、应对流量洪峰。

不幸的是，这些消息队列产品的API都不一样。当应用想要跨云部署，或者想要移植（比如从阿里云搬到腾讯云），应用需要重构代码。

Layotto Pub/Sub API的设计目标是定义一套统一的消息发布/订阅API，应用只需要关心API、不需要关心具体用的哪个消息队列产品，让应用能够随意移植，让应用足够"云原生"。

## 快速开始

该示例展示了如何通过Layotto调用redis，进行消息发布/订阅。

该示例的架构如下图，启动的进程有：redis、一个监听事件的Subscriber程序、Layotto、一个发布事件的Publisher程序

![img_1.png](../../../img/mq/start/img_1.png)

### 第一步：部署redis

1. 取最新版的 Redis 镜像。
这里我们拉取官方的最新版本的镜像：

```shell
docker pull redis:latest
```

2. 查看本地镜像
   使用以下命令来查看是否已安装了 redis：

```shell
docker images
```
![img.png](../../../img/mq/start/img.png)

3. 运行容器

安装完成后，我们可以使用以下命令来运行 redis 容器：

```shell
docker run -itd --name redis-test -p 6380:6379 redis
```

参数说明：

-p 6380:6379：映射容器服务的 6379 端口到宿主机的 6380 端口。外部可以直接通过宿主机ip:6380 访问到 Redis 的服务。

### 第二步：启动Subscriber程序,订阅事件
```shell
 cd ${project_path}/demo/pubsub/server/
 go build -o subscriber
```

```shell @background
 ./subscriber -s pub_subs_demo
```
打印出如下信息则代表启动成功：

```bash
Start listening on port 9999 ...... 
```

解释：

该程序会启动一个gRPC服务器，开放两个接口：

- ListTopicSubscriptions

调用该接口会返回应用订阅的Topic。本程序会返回"topic1"

- OnTopicEvent

当有新的事件发生时，Layotto会调用该接口，将新事件通知给Subscriber。

本程序接收到新事件后，会将事件打印到命令行。

### 第三步：运行Layotto

将项目代码下载到本地后，切换代码目录：

```shell
cd ${project_path}/cmd/layotto
```

构建:

```shell @if.not.exist layotto
go build -o layotto
```

完成后目录下会生成layotto文件，运行它：

```shell @background
./layotto start -c ../../configs/config_redis.json
```

### 第四步：运行Publisher程序，调用Layotto发布事件

```shell
 cd ${project_path}/demo/pubsub/client/
 go build -o publisher
 ./publisher -s pub_subs_demo
```

打印出如下信息则代表调用成功：

```bash
Published a new event.Topic: topic1 ,Data: value1 
```

### 第五步：检查Subscriber收到的事件消息

回到subscriber的命令行，会看到接收到了新消息：

```bash
Start listening on port 9999 ...... 
Received a new event.Topic: topic1 , Data:value1 
```

### 下一步
#### 这个Publisher程序做了什么？
示例Publisher程序中使用了Layotto提供的golang版本sdk，调用Layotto Pub/Sub API,发布事件到redis。随后Layotto监听到redis有新事件，将新事件回调Subscriber程序开放的接口，通知Subscriber。

sdk位于`sdk`目录下，用户可以通过sdk调用Layotto提供的API。

除了使用sdk，您也可以用任何您喜欢的语言、通过grpc直接和Layotto交互。

其实sdk只是对grpc很薄的封装，用sdk约等于直接用grpc调。


#### 细节以后再说，继续体验其他API
通过左侧的导航栏，继续体验别的API吧！

#### 了解Pub/Sub API实现原理

如果您对实现原理感兴趣，或者想扩展一些功能，可以阅读[Pub/Sub API的设计文档](zh/design/pubsub/pubsub-api-and-compability-with-dapr-component.md)