---
title: ActiveMQ基础与运用
date: 2018-07-06
tags: 
categories: 框架

---



### ActiveMQ基础与运用 ###

[TOC]

#### 1.概念 ####

消息队列:**即时消息通信**和**延时消息通信**



ActiveMQ底层基于java的JMS实现,在没有JMS之前的系统存在很多缺陷:

1. 前后端同步问题,如果后台没有响应,则前段会一直阻塞等待
2. 前后端生命周期耦合性太强,一方崩了则另一方也会崩
3. 点对点通信,前段一次只能发送给某一个单独的服务对象,无法群发

**JMS:** (Java Message Service ) 通过消息中间件(MOM：Message Oriented Middleware )

将消息发送给单独的消息服务器中,消息服务器会将消息存放在若干的队列/主题中,在合适的时候将消息发送给接收者.**发送和接收是异步的,无需阻塞等待** 在pub/sub的模式下,可以将消息发送给多个接收者

**JMS类中定义了java访问中间件的接口,除此之外都是异常定义**

1. Provider/MessageProvider：生产者 
2. Consumer/MessageConsumer：消费者 
3. PTP：Point To Point，点对点通信消息模型 
4. Pub/Sub：Publish/Subscribe，发布订阅消息模型 
5. **Queue**：队列，目标类型之一，和PTP结合 
6. **Topic**：主题，目标类型之一，和Pub/Sub结合 
7. ConnectionFactory：连接工厂，JMS用它创建连接 
8. Connnection：JMS Client到JMS Provider的连接 
9. Destination：消息目的地，由Session创建 
10. **Session**：会话，由Connection创建，实质上就是发送、接受消息的一个线程，因此生产者、消费者都是Session创建的 

#### 2.应用 ####

**| 异步处理 | 应用解耦 | 流量削锋 | 消息通讯 |**

**详情参考:https://blog.csdn.net/kingcat666/article/details/78660535**

#### 3.消息模式 ####
- **P2P模式(点对点)**
- **Pub/Sub模式(发布订阅)**
- **Push模式(推拉模式,消息更新C/S中)**



#### 4.java中与Solr结合 ####

ActiveMQ以监视器的方式将信息与Solr结合使用...待更新