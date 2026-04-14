---
title: 初识RabbitMQ
tags:
  - 同步调用
  - 异步调用
  - 交换机
  - 消息队列
categories:
  - Java
  - 消息队列
date: 2024-08-06 21:52:42
---

<meta name="referrer" content="no-referrer"/>

微服务一旦拆分，必然涉及到服务之间的相互调用，目前我们服务之间调用采用的都是基于OpenFeign的调用。这种调用中，调用者发起请求后需要**等待**服务提供者执行业务返回结果后，才能继续执行后面的业务。也就是说调用者在调用过程中处于阻塞状态，因此我们成这种调用方式为**同步调用**，也可以叫**同步通讯**。但在很多场景下，我们可能需要采用**异步通讯**的方式，为什么呢？

我们先来看看什么是同步通讯和异步通讯。如图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1686983181054-f2bcce85-1fce-412f-95cd-1ae829f8406f.png#averageHue=%239dce6d&clientId=uf9c47826-2719-4&from=paste&height=613&id=u84c8f02e&originHeight=760&originWidth=1695&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=112976&status=done&style=none&taskId=u779e4d59-c9a8-4b1f-a49f-59c578c4ccd&title=&width=1367.3949141495996)

<!--more-->

解读：

- 同步通讯：就如同打视频电话，双方的交互都是实时的。因此同一时刻你只能跟一个人打视频电话。
- 异步通讯：就如同发微信聊天，双方的交互不是实时的，你不需要立刻给对方回应。因此你可以多线操作，同时跟多人聊天。

两种方式各有优劣，打电话可以立即得到响应，但是你却不能跟多个人同时通话。发微信可以同时与多个人收发微信，但是往往响应会有延迟。

所以，如果我们的业务需要实时得到服务提供方的响应，则应该选择同步通讯（同步调用）。而如果我们追求更高的效率，并且不需要实时响应，则应该选择异步通讯（异步调用）。

同步调用的方式我们已经学过了，之前的OpenFeign调用就是。但是：

- 异步调用又该如何实现？
- 哪些业务适合用异步调用来实现呢？

通过今天的学习你就能明白这些问题了。

# 1.初识MQ

## 1.1.同步调用

之前说过，我们现在基于OpenFeign的调用都属于是同步调用，那么这种方式存在哪些问题呢？
举个例子，我们以昨天留给大家作为作业的**余额支付功能**为例来分析，首先看下整个流程：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686989758652-29a64761-c029-4ec4-91aa-f1fc85de086c.jpeg)
目前我们采用的是基于OpenFeign的同步调用，也就是说业务执行流程是这样的：

- 支付服务需要先调用用户服务完成余额扣减
- 然后支付服务自己要更新支付流水单的状态
- 然后支付服务调用交易服务，更新业务订单状态为已支付

三个步骤依次执行。
这其中就存在3个问题：
**第一**，**拓展性差**
我们目前的业务相对简单，但是随着业务规模扩大，产品的功能也在不断完善。
在大多数电商业务中，用户支付成功后都会以短信或者其它方式通知用户，告知支付成功。假如后期产品经理提出这样新的需求，你怎么办？是不是要在上述业务中再加入通知用户的业务？
某些电商项目中，还会有积分或金币的概念。假如产品经理提出需求，用户支付成功后，给用户以积分奖励或者返还金币，你怎么办？是不是要在上述业务中再加入积分业务、返还金币业务？
。。。
最终你的支付业务会越来越臃肿：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686984472076-c05b2155-3346-40f5-b85e-5961caa998ab.jpeg)
也就是说每次有新的需求，现有支付逻辑都要跟着变化，代码经常变动，不符合开闭原则，拓展性不好。

**第二**，**性能下降**
由于我们采用了同步调用，调用者需要等待服务提供者执行完返回结果后，才能继续向下执行，也就是说每次远程调用，调用者都是阻塞等待状态。最终整个业务的响应时长就是每次远程调用的执行时长之和：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686989760653-42e1ae3e-677b-4f27-b55a-eaa259f03ad3.jpeg)
假如每个微服务的执行时长都是50ms，则最终整个业务的耗时可能高达300ms，性能太差了。

**第三，级联失败**
由于我们是基于OpenFeign调用交易服务、通知服务。当交易服务、通知服务出现故障时，整个事务都会回滚，交易失败。
这其实就是同步调用的**级联失败**问题。

但是大家思考一下，我们假设用户余额充足，扣款已经成功，此时我们应该确保支付流水单更新为已支付，确保交易成功。毕竟收到手里的钱没道理再退回去吧![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1686986652875-9e2924a9-e0f3-4de2-ae41-8b39ef6345bc.png#averageHue=%23d2c088&clientId=uf9c47826-2719-4&from=paste&height=22&id=u1eecfdc1&originHeight=143&originWidth=150&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=42199&status=done&style=none&taskId=u80811df5-062e-457a-a9a8-0173e00f6b1&title=&width=22.999998092651367)。

因此，这里不能因为短信通知、更新订单状态失败而回滚整个事务。


综上，同步调用的方式存在下列问题：

- 拓展性差
- 性能下降
- 级联失败

而要解决这些问题，我们就必须用**异步调用**的方式来代替**同步调用**。


## 1.2.异步调用

异步调用方式其实就是基于消息通知的方式，一般包含三个角色：

- 消息发送者：投递消息的人，就是原来的调用方
- 消息Broker：管理、暂存、转发消息，你可以把它理解成微信服务器
- 消息接收者：接收和处理消息的人，就是原来的服务提供方

![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686990662733-65b0eac8-f65f-4024-a581-6d5761c4c5a4.jpeg)

在异步调用中，发送者不再直接同步调用接收者的业务接口，而是发送一条消息投递给消息Broker。然后接收者根据自己的需求从消息Broker那里订阅消息。每当发送方发送消息后，接受者都能获取消息并处理。
这样，发送消息的人和接收消息的人就完全解耦了。

还是以余额支付业务为例：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686990257816-4f0b5ddd-7618-4095-b797-25b92f0bf2a5.jpeg)
除了扣减余额、更新支付流水单状态以外，其它调用逻辑全部取消。而是改为发送一条消息到Broker。而相关的微服务都可以订阅消息通知，一旦消息到达Broker，则会分发给每一个订阅了的微服务，处理各自的业务。

假如产品经理提出了新的需求，比如要在支付成功后更新用户积分。支付代码完全不用变更，而仅仅是让积分服务也订阅消息即可：
![](https://cdn.nlark.com/yuque/0/2023/jpeg/27967491/1686989956210-7c1f451c-0368-4602-b02e-a66f2c0f6deb.jpeg)
不管后期增加了多少消息订阅者，作为支付服务来讲，执行问扣减余额、更新支付流水状态后，发送消息即可。业务耗时仅仅是这三部分业务耗时，仅仅100ms，大大提高了业务性能。

另外，不管是交易服务、通知服务，还是积分服务，他们的业务与支付关联度低。现在采用了异步调用，解除了耦合，他们即便执行过程中出现了故障，也不会影响到支付服务。

综上，异步调用的优势包括：

- 耦合度更低
- 性能更好
- 业务拓展性强
- 故障隔离，避免级联失败

当然，异步通信也并非完美无缺，它存在下列缺点：

- 完全依赖于Broker的可靠性、安全性和性能
- 架构复杂，后期维护和调试麻烦

## 1.3.技术选型

消息Broker，目前常见的实现方案就是消息队列（MessageQueue），简称为MQ.
目比较常见的MQ实现：

- ActiveMQ
- RabbitMQ
- RocketMQ
- Kafka

几种常见MQ的对比：

|            | **RabbitMQ**            | **ActiveMQ**                   | **RocketMQ** | **Kafka**  |
| ---------- | ----------------------- | ------------------------------ | ------------ | ---------- |
| 公司/社区  | Rabbit                  | Apache                         | 阿里         | Apache     |
| 开发语言   | Erlang                  | Java                           | Java         | Scala&Java |
| 协议支持   | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议   | 自定义协议 |
| 可用性     | 高                      | 一般                           | 高           | 高         |
| 单机吞吐量 | 一般                    | 差                             | 高           | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                         | 毫秒级       | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                           | 高           | 一般       |


追求可用性：Kafka、 RocketMQ 、RabbitMQ
追求可靠性：RabbitMQ、RocketMQ
追求吞吐能力：RocketMQ、Kafka
追求消息低延迟：RabbitMQ、Kafka

据统计，目前国内消息队列使用最多的还是RabbitMQ，再加上其各方面都比较均衡，稳定性也好，因此我们课堂上选择RabbitMQ来学习。

# 2.RabbitMQ

RabbitMQ是基于Erlang语言开发的开源消息通信中间件，官网地址：
[Messaging that just works — RabbitMQ](https://www.rabbitmq.com/)
接下来，我们就学习它的基本概念和基础用法。

## 2.1.安装

我们同样基于Docker来安装RabbitMQ，使用下面的命令即可：

```shell
docker run \
 -e RABBITMQ_DEFAULT_USER=itheima \
 -e RABBITMQ_DEFAULT_PASS=123321 \
 -v mq-plugins:/plugins \
 --name mq \
 --hostname mq \
 -p 15672:15672 \
 -p 5672:5672 \
 --network hmall \
 -d \
 rabbitmq:3.8-management
```

可以看到在安装命令中有两个映射的端口：

- 15672：RabbitMQ提供的管理控制台的端口
- 5672：RabbitMQ的消息发送处理接口

安装完成后，我们访问 http://192.168.150.101:15672即可看到管理控制台。首次访问需要登录，默认的用户名和密码在配置文件中已经指定了。
登录后即可看到管理控制台总览页面：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687137883587-56417f79-a649-43a5-be88-2ff777d3cd25.png#averageHue=%23f7f6f6&clientId=u6a529863-cf4b-4&from=paste&height=707&id=u7d848ee1&originHeight=876&originWidth=1572&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=83168&status=done&style=none&taskId=ub505f8cf-075f-462b-bce3-e0df935715d&title=&width=1268.168026574142)


RabbitMQ对应的架构如图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687136827222-52374724-79c9-4738-b53f-653cc0805d22.png#averageHue=%23e8d7b3&clientId=u6a529863-cf4b-4&from=paste&height=495&id=ub8dd8df6&originHeight=614&originWidth=1458&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=104273&status=done&style=none&taskId=uc0c132a5-73a3-4024-819f-61241da2511&title=&width=1176.2016429676203)
其中包含几个概念：

- `**publisher**`：生产者，也就是发送消息的一方
- `**consumer**`：消费者，也就是消费消息的一方
- `**queue**`：队列，存储消息。生产者投递的消息会暂存在消息队列中，等待消费者处理
- `**exchange**`：交换机，负责消息路由。生产者发送的消息由交换机决定投递到哪个队列。
- `**virtual host**`：虚拟主机，起到数据隔离的作用。每个虚拟主机相互独立，有各自的exchange、queue

上述这些东西都可以在RabbitMQ的管理控制台来管理，下一节我们就一起来学习控制台的使用。

## 2.2.收发消息

### 2.2.1.交换机

我们打开Exchanges选项卡，可以看到已经存在很多交换机：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687137953880-08aa9694-6a1e-4337-8bde-5757ec3c33f8.png#averageHue=%23f7f6f6&clientId=u6a529863-cf4b-4&from=paste&height=605&id=u413741e2&originHeight=750&originWidth=1264&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=60217&status=done&style=none&taskId=u8611b86c-aa50-46d9-855f-8307a318079&title=&width=1019.6974463038903)
我们点击任意交换机，即可进入交换机详情页面。仍然会利用控制台中的publish message 发送一条消息：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687138031622-ccce4612-954f-42c0-9291-73cf19915e39.png#averageHue=%23f9f8f7&clientId=u6a529863-cf4b-4&from=paste&height=487&id=u9d211d96&originHeight=604&originWidth=947&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=38263&status=done&style=none&taskId=ue134ec0e-ad83-465f-a1b2-97cb7667d75&title=&width=763.9663620647026)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687138163403-839087fe-66f7-4710-a866-210aa0282be8.png#averageHue=%23f9f6f6&clientId=u6a529863-cf4b-4&from=paste&height=616&id=ubca84480&originHeight=763&originWidth=1092&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=46085&status=done&style=none&taskId=u5f176fff-eda8-457c-94cd-bb7d6bbd997&title=&width=880.9411482308925)
这里是由控制台模拟了生产者发送的消息。由于没有消费者存在，最终消息丢失了，这样说明交换机没有存储消息的能力。

### 2.2.2.队列

我们打开`Queues`选项卡，新建一个队列,命名为`hello.queue1`：
再以相同的方式，创建一个队列，命名为`hello.queue2`，最终队列列表如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255204405-523f8053-e414-45f3-99c3-b66de152f79e.png#averageHue=%23f6f5f4&clientId=u1711eaf3-9387-4&from=paste&height=359&id=u956d1947&originHeight=445&originWidth=1074&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=39049&status=done&style=none&taskId=u1eb8bf9f-f74b-4238-a33e-796c4280e78&title=&width=866.4201402930207)
此时，我们再次向`amq.fanout`交换机发送一条消息。会发现消息依然没有到达队列！！
怎么回事呢？
发送到交换机的消息，只会路由到与其绑定的队列，因此仅仅创建队列是不够的，我们还需要将其与交换机绑定。

### 2.2.3.绑定关系

点击`Exchanges`选项卡，点击`amq.fanout`交换机，进入交换机详情页，然后点击`Bindings`菜单，在表单中填写要绑定的队列名称：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255547460-d87943cd-4309-4778-8e9e-374167a97e45.png#averageHue=%23f9f7f7&clientId=u1711eaf3-9387-4&from=paste&height=481&id=u04a61731&originHeight=596&originWidth=1022&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=34676&status=done&style=none&taskId=u0ce69958-400b-4c37-89ea-adf0b369080&title=&width=824.4705618058354)
相同的方式，将hello.queue2也绑定到改交换机。
最终，绑定结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255624712-7bd850b1-95fd-4d98-8243-57d1779de935.png#averageHue=%23f7f4f4&clientId=u1711eaf3-9387-4&from=paste&height=385&id=u82198db4&originHeight=477&originWidth=978&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=28098&status=done&style=none&taskId=u1394f18f-c109-4688-9eb1-effec6a43fb&title=&width=788.9747646243708)

### 2.2.4.发送消息

再次回到exchange页面，找到刚刚绑定的`amq.fanout`，点击进入详情页，再次发送一条消息：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687138163403-839087fe-66f7-4710-a866-210aa0282be8.png#averageHue=%23f9f6f6&clientId=u6a529863-cf4b-4&from=paste&height=616&id=GyhjT&originHeight=763&originWidth=1092&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=46085&status=done&style=none&taskId=u5f176fff-eda8-457c-94cd-bb7d6bbd997&title=&width=880.9411482308925)
回到`Queues`页面，可以发现`hello.queue`中已经有一条消息了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255725782-fd5e2550-3572-48c0-9ec0-60786e33a3b1.png#averageHue=%23f5f4f3&clientId=u1711eaf3-9387-4&from=paste&height=319&id=u97a4707c&originHeight=395&originWidth=1051&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=38484&status=done&style=none&taskId=u4d68c013-3032-4d2b-83a0-571c3335780&title=&width=847.8655190390733)
点击队列名称，进入详情页，查看队列详情，这次我们点击get message：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255765034-69e67460-1535-48b3-8537-da383c498141.png#averageHue=%23f8f7f7&clientId=u1711eaf3-9387-4&from=paste&height=473&id=ua850c29b&originHeight=586&originWidth=974&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=35281&status=done&style=none&taskId=u668d2c9f-54a9-4427-adc4-e121a960025&title=&width=785.7478739715103)
可以看到消息到达队列了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687255798153-dda9b729-a3a0-415c-9167-48c525c75800.png#averageHue=%23f9f7f7&clientId=u1711eaf3-9387-4&from=paste&height=466&id=u66fa5450&originHeight=578&originWidth=762&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=33500&status=done&style=none&taskId=u665361c6-23b2-4fc4-b1a9-fdf6c880545&title=&width=614.7226693699085)
这个时候如果有消费者监听了MQ的`hello.queue1`或`hello.queue2`队列，自然就能接收到消息了。

## 2.3.数据隔离

### 2.3.1.用户管理

点击`Admin`选项卡，首先会看到RabbitMQ控制台的用户管理界面：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687151143347-f7e2aaff-0a14-4022-8d50-582ee75e2998.png#averageHue=%23f7f5f5&clientId=uc5430584-57f9-4&from=paste&height=450&id=u2a51a990&originHeight=558&originWidth=1580&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=55212&status=done&style=none&taskId=u18a12c4e-be8d-4ccb-a14a-415a21db44a&title=&width=1274.621807879863)
这里的用户都是RabbitMQ的管理或运维人员。目前只有安装RabbitMQ时添加的`itheima`这个用户。仔细观察用户表格中的字段，如下：

- `Name`：`itheima`，也就是用户名
- `Tags`：`administrator`，说明`itheima`用户是超级管理员，拥有所有权限
- `Can access virtual host`： `/`，可以访问的`virtual host`，这里的`/`是默认的`virtual host`

对于小型企业而言，出于成本考虑，我们通常只会搭建一套MQ集群，公司内的多个不同项目同时使用。这个时候为了避免互相干扰， 我们会利用`virtual host`的隔离特性，将不同项目隔离。一般会做两件事情：

- 给每个项目创建独立的运维账号，将管理权限分离。
- 给每个项目创建不同的`virtual host`，将每个项目的数据隔离。

比如，我们给黑马商城创建一个新的用户，命名为`hmall`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687151725993-05fe9bd1-8f8b-468d-8456-eac36278bea2.png#averageHue=%23f7f5f5&clientId=uc5430584-57f9-4&from=paste&height=609&id=ua32ca0ae&originHeight=755&originWidth=1569&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=70298&status=done&style=none&taskId=u4f4ed00c-b8dd-4ffd-8a83-75d03c11fb5&title=&width=1265.7478585844967)
你会发现此时hmall用户没有任何`virtual host`的访问权限：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687151853554-e671a696-e9c0-4ff5-9caf-31b39e1a17f5.png#averageHue=%23f7f5f4&clientId=uc5430584-57f9-4&from=paste&height=353&id=ueeaf90c6&originHeight=437&originWidth=927&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=31497&status=done&style=none&taskId=u74d79385-5602-447d-8beb-ed20ec36022&title=&width=747.8319088004005)
别急，接下来我们就来授权。


### 2.3.2.virtual host

我们先退出登录：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687152245922-8438490f-d094-4db1-88fa-a2d916d46a97.png#averageHue=%23f6f5f5&clientId=uc5430584-57f9-4&from=paste&height=374&id=u12c0492e&originHeight=463&originWidth=1571&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=50699&status=done&style=none&taskId=u830f1745-a0ed-4202-9849-7653ebae4c2&title=&width=1267.3613039109268)
切换到刚刚创建的hmall用户登录，然后点击`Virtual Hosts`菜单，进入`virtual host`管理页：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687152310566-2531b1c8-b362-47c7-ba81-1b7c1880c18b.png#averageHue=%23f5f4f3&clientId=uc5430584-57f9-4&from=paste&height=409&id=uf51820c2&originHeight=507&originWidth=1565&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=60462&status=done&style=none&taskId=ud7655191-c9d5-4801-9669-55b47348861&title=&width=1262.5209679316363)
可以看到目前只有一个默认的`virtual host`，名字为 `/`。
 我们可以给黑马商城项目创建一个单独的`virtual host`，而不是使用默认的`/`。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687152363999-edb47263-f303-4ee8-a80d-be55d6b0ed37.png#averageHue=%23f6f5f4&clientId=uc5430584-57f9-4&from=paste&height=553&id=ufc5bd4a7&originHeight=685&originWidth=1555&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=67199&status=done&style=none&taskId=u38a5fe38-fcb5-4163-bee4-ddffaba416b&title=&width=1254.4537412994853)
创建完成后如图：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687152448758-d0a05827-10ac-459b-a92f-495304dddf89.png#averageHue=%23f5f5f4&clientId=uc5430584-57f9-4&from=paste&height=232&id=ue38b9ba4&originHeight=287&originWidth=990&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=24622&status=done&style=none&taskId=ue5f326dc-340c-46e0-83d5-84a19fef1d9&title=&width=798.655436582952)
由于我们是登录`hmall`账户后创建的`virtual host`，因此回到`users`菜单，你会发现当前用户已经具备了对`/hmall`这个`virtual host`的访问权限了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687152695194-6c2dda94-43c4-4ee9-b95c-ca9d8504cd0c.png#averageHue=%23f7f4f4&clientId=ud5bd9b1f-141b-4&from=paste&height=349&id=u0cf22cf3&originHeight=432&originWidth=890&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=30925&status=done&style=none&taskId=u1b04bdb9-ab59-41d9-b2bb-e5f5a0cca59&title=&width=717.9831702614417)

此时，点击页面右上角的`virtual host`下拉菜单，切换`virtual host`为 `/hmall`：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687153236457-ca138f25-b351-4095-8855-aa0df42fae65.png#averageHue=%23f7f5f4&clientId=ud5bd9b1f-141b-4&from=paste&height=223&id=u0989d284&originHeight=277&originWidth=1448&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=35060&status=done&style=none&taskId=u6ab4f38a-ad0d-48bd-ace7-7f0281755d1&title=&width=1168.1344163354693)
然后再次查看queues选项卡，会发现之前的队列已经看不到了：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/27967491/1687153307085-0157ac47-2d89-4f32-ab9a-d513b0e19f25.png#averageHue=%23f9f6f6&clientId=ud5bd9b1f-141b-4&from=paste&height=431&id=u151b88b9&originHeight=534&originWidth=1443&originalType=binary&ratio=1.2395833730697632&rotation=0&showTitle=false&size=48526&status=done&style=none&taskId=u7ef6af14-7e3c-4988-8721-80965a310f6&title=&width=1164.1008030193937)
这就是基于`virtual host `的隔离效果。
