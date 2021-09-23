<!-- GFM-TOC -->

- [分布式基础理论](#分布式基础理论)
  - [什么是分布式系统](#什么是分布式系统)
  - [发展演变](#发展演变)
    - [单一应用架构](#单一应用架构)
    - [垂直应用架构](#垂直应用架构)
    - [分布式服务架构](#分布式服务架构)
    - [流动计算架构](#流动计算架构)
- [RPC](#RPC)
  - [简介](#简介)
  - [基本原理](#基本原理)

- [Reference](#Reference)

<!-- GFM-TOC -->

# 分布式基础理论

## 什么是分布式系统

> 《分布式系统原理与范型》定义：
>
> “分布式系统是若干独立计算机的集合，这些计算机对于用户来说就像单个相关系统”

分布式系统（distributed system）是建立在网络之上的软件系统。

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需**一个治理系统**确保架构有条不紊的演进

## 发展演变

- 推荐阅读：[分布式架构演变历史](https://blog.csdn.net/yuhaiyang_1/article/details/80862914)

### 单一应用架构

当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键

<center><img src="https://i.im5i.com/2021/05/24/C23Tl.png" alt="C23Tl.png" border="0" /></center>

优点：

- 适用于小型网站，小型管理系统**，将所有功能都部署到一个功能里**，简单易用。

缺点：

- 性能扩展比较难

- 协同开发问题

- 不利于升级维护

### 垂直应用架构

当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。此时，用于加速前端页面开发的**Web框架(MVC)**是关键

<center><img src="https://i.im5i.com/2021/05/24/C2i62.png" alt="C2i62.png" border="0" /></center>

优点：

- 通过切分业务来实现各个模块独立部署，降低了维护和部署的难度，团队各司其职更易管理，性能扩展也更方便，更有针对性。

缺点：

- 公用模块无法重复利用，开发性的浪费

### 分布式服务架构

当垂直应用越来越多，**应用之间交互不可避免**，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的<font color="red">分布式服务框架(RPC)</font>是关键

<center><img src="https://i.im5i.com/2021/05/24/C2mbP.png" alt="C2mbP.png" border="0" /></center>

<center><a href="https://sm.ms/image/wTGMtsO1QhJc8gd" target="_blank"><img src="https://i.loli.net/2021/05/25/wTGMtsO1QhJc8gd.png" ></a></center>

RPC框架是传统单体应用向微服务应用转变过程中必不可少的工具，RPC更有利于服务拆分，将原本耦合度高，扩展性差、复杂度高和稳定性差的单体应用变成一个高内聚低耦合的微服务应用。总结起来，RPC使得开发人员更加专注自己的业务开发、有效的屏蔽了技术细节、提升开发效率、运维方便

### 流动计算架构

当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的<font color="red">资源调度和治理中心(SOA)[ Service Oriented Architecture]</font>是关键

<center><img src="https://i.im5i.com/2021/05/24/C2xiD.png" alt="C2xiD.png" border="0" /></center>

<center><a href="https://sm.ms/image/hAL4ic7yp918eRf" target="_blank"><img src="https://i.loli.net/2021/05/25/hAL4ic7yp918eRf.png" ></a></center>

# RPC

## 简介

RPC（Remote Procedure Call）是指**远程过程调用**，是一种进程间通信方式，是一种技术的思想，而不是规范。它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即程序员无论是调用本地的还是远程的函数，本质上编写的调用代码基本相同

## 基本原理

<center><img src="https://i.im5i.com/2021/05/24/C2AQj.png" alt="C2AQj.png" border="0" /></center>

1. client以本地调用方式调用服务
2. Client stub将调用方法、参数做封装序列化发起网络请求
3. 数据通过网络传输
4. Server stub收到网络消息后进行解码、反序列化
5. server stub根据解码结果调用本地服务
6. 本地服务执行并将结果返回给server stub
7. server stub将返回结果打包成消息并发送至消费方
8. 数据通过网络传输
9. client stub接收到消息，并进行解码
10. 服务消费方得到最终结果

<center><img src="https://i.im5i.com/2021/05/24/C2LBS.png" alt="C2LBS.png" border="0" /></center>

### RPC核心要素

### 序列化和反序列化

序列化和反序列化的常见方案（协议）有：***XML、Json、Thrift、Protobuf***

### 网络传输和IO模型

网络协议：TCP、HTTP

IO模型：BIO、NIO

### 服务的注册和发现

为了实现高可用和服务的实时监控，服务的提供方需要向服务注册中心暴露其提供的接口，当服务发生变更时要及时通知服务调用方也需要通过服务注册中心来实现。服务注册中心主要提供服务注册能力、服务心跳监控和服务的变更通知（***register、subscribe、notify***）

# Reference

- [尚硅谷Dubbo教程](https://www.bilibili.com/video/BV1ns411c7jV)