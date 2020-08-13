---
title: easyswoole 微服务-Rpc
meta:
  - name: description
    content: EasySwoole中用RPC实现分布式微服务架构
  - name: keywords
    content: swoole|swoole 拓展|swoole 框架|easyswoole|Rpc服务端|swoole RPC|swoole微服务|swoole分布式|PHP 分布式
---

# EasySwoole RPC

很多传统的`Phper`并不懂`RPC`是什么，`RPC`全称`Remote Procedure Call`，中文译为远程过程调用,其实你可以把它理解为是一种架构性上的设计，或者是一种解决方案。

例如在某庞大商场系统中，你可以把整个商场拆分为`N`个微服务（理解为`N`个独立的小模块也行），例如：
    
- 订单系统
- 用户管理系统
- 商品管理系统
- 等等 

那么在这样的架构中，就会存在一个Api网关的概念，或者是叫服务集成者。我的Api网关的职责，就是把一个请求
，拆分成N个小请求，分发到各个小服务里面，再整合各个小服务的结果，返回给用户。例如在某次下单请求中，那么大概
发送的逻辑如下：
- Api网关接受请求
- Api网关提取用户参数，请求用户管理系统，获取用户余额等信息，等待结果
- Api网关提取商品参数，请求商品管理系统，获取商品剩余库存和价格等信息，等待结果。
- Api网关融合用户管理系统、商品管理系统的返回结果，进行下一步调用（假设满足购买条件）
- Api网关调用用户管理信息系统进行扣款，调用商品管理系统进行库存扣减，调用订单系统进行下单（事务逻辑和撤回可以用请求id保证，或者自己实现其他逻辑调度）
- APi网关返回综合信息给用户

而在以上发生的行为，就称为远程过程调用。而调用过程实现的通讯协议可以有很多，比如常见的`HTTP`协议。而`EasySwoole RPC`采用自定义短链接的`TCP`协议实现，每个请求包，都是一个`JSON`，从而方便实现跨平台调用。

## 全新特性
 - 协程调度
 - 服务自动发现
 - 服务熔断
 - 服务降级
 - Openssl加密
 - 跨平台，跨语言支持
 - 支持接入第三方注册中心

## 安装

> composer require easyswoole/rpc=4.x

## 执行流程

服务端：  
注册RPC服务，创建相应的服务swoole table表（ps:记录调用成功和失败的次数） 
注册worker,tick进程  
  
woker进程监听：   
客户端发送请求->解包成相对应的格式->执行对应的服务->返回结果->客户端  

tick进程：  
注册定时器发送心跳包到本节点管理器   
启用广播：每隔几秒发送本节点各个服务信息到其他节点  
启用监听：监听其他节点发送的信息，发送相对应的命令（心跳|下线）到节点管理器处理  
进程关闭：主动删除本节点的信息，发送下线广播到其他节点  

![](/Images/Passage/rpcDesign.png)