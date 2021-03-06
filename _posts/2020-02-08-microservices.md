---
title: 微服务概念学习笔记
tags:
    - 微服务
    - Microservices
---

**[参考链接](https://martinfowler.com/articles/microservices.html)**

# 微服务概念学习笔记

## 微服务架构风格

简单来说，微服务架构风格是一种将一个单一应用程序开发为**一组小型服务**的方法，每个服务**运行在自己的进程中**，服务间通信采用轻量级通信机制(通常用HTTP资源API)。这些服务**围绕业务能力构建**并且可通过全自动部署机制**独立部署**。这些服务使用不同的编程语言书写，以及不同数据存储技术，并且仅保持**最低限度的集中式管理**。

## 和单体风格的对比

单体应用程序被构建为单一单元。系统的任何改变都将牵涉到重新构建和部署服务端的一个新版本。该单体的水平扩展可以通过在负载均衡器后面运行多个实例来实现。

### 单体风格存在的问题

变更周期被捆绑在一起 —— 即使只变更应用程序的一部分，也需要重新构建并部署整个单体。

### 微服务架构风格

构建应用程序为服务套件。除了服务是可独立部署、可独立扩展的之外，每个服务都提供一个固定的模块边界。甚至允许不同的服务用不同的的语言开发，由不同的团队管理。

![单体和微服务](/assets/posts/Monoliths%20and%20Microservices.png)

微服务可按需伸缩。

## 微服务架构的特征

### 通过服务组件化

组件是一个可独立替换和独立升级的软件单元。

**微服务架构组件化软件的主要方式：**
分解成服务，而服务是进程外的组件，它通过Web服务请求或RPC(远程过程调用)机制通信。

>![单体应用](/assets/posts/monolithic-architecture-app.jpg)
![微服务应用](/assets/posts/micro-services-app.jpg)

**使用服务作为组件而不是使用库的主要原因：**

- 服务可以独立布署：
每个服务的变更仅需要发布相应的服务就可以
- 更加明确的接口：
服务通过明确的远程调用机制可以避免组件间的紧耦合

**缺点：**

- 远程调用比进程内部调用更加消耗性能
- 远程的API往往是粗粒度的，用起来不是很友好
- 需要更改组件间的责任(功能)分配时更困难

### 围绕“业务功能”组织团队

人员组织结构决定了产品结构。
然而，当我们想把一个大的应用拆分成小的部分时，我们的注意力主要集中在技术层面，拆分成UI团队、服务端的逻辑团队和数据库团队。当使用这种标准对团队进行划分时，甚至一个非常小的更变都将导致跨团队间项目协作，从而消耗时间和预算审批。
![Conway定律](/assets/posts/Conway's%20Law%20in%20action.png)

微服务采用不同的分割方法，它更倾向于围绕**业务功能**对服务结构进行划分、拆解。因此，团队都是**跨职能**的，包括开发需要的全方位技能：用户体验、数据库、项目管理。
![产品结构反应了人员组织结构](/assets/posts/Service%20boundaries%20reinforced%20by%20team%20boundaries.png)

### “做产品”而不是“做项目”

大多数项目模式：目标是交付将要完成的一些软件。完成后的软件被交接给维护组织，然后它的构建团队就解散了。
**微服务支持者倾向于避免这种模式**，而是认为一个团队应该负责产品的**整个生命周期**。
> 亚马逊："You build, you run it"

### “智能端点”与“傻瓜管道”

要实现不同进程间通信，很多产品和方法把“智能”强加进通信机制本身（如ESB）。
> SOA(Service Oriented Architecture, 面向服务的架构)专注于ESB，来集成各个单块应用，不同于本微服务风格。
![ESB](/assets/posts/ESB-exp.png)
![微服务-服务间调用](/assets/posts/micro-services-exp.jpg.png)

**微服务社区主张另一种方法**：尽可能的解耦和尽可能的内聚。
接收请求->应用适当的逻辑->产生响应。使用**简单**的REST风格的协议，如：

- REST API的HTTP请求-响应和轻量级消息传送
- 在轻量级消息总线上传递消息（如Protobufs）

把单体变成微服务,最大的问题在于**通信模式的改变**。用**粗粒度通信**代替细粒度通信。

### 去中心化治理技术

使用正确的工具来完成工作。把单体的组件分裂成服务，在构建这些服务时可以有自己的选择。
去中心化治理的最高境界就是"build it / run it"理念：团队要对他们构建的软件的各方面负责，包括7x24小时的运营。

### 去中心化数据管理

微服务更倾向于让每个服务管理自己的数据库，或者同一数据库技术的不同实例，或完全不同的数据库系统。
![去中心化数据管理](/assets/posts/decentralised-data.png)
**分布式事务：**
微服务架构强调服务间的无事务协作，关注最终一致性和事后补偿。
**权衡：**
修复错误的代价是否小于一致性下业务损失的代价？

### 基础设施自动化

基于持续部署(和它的前身持续集成)，构建管线图：
![基础构建管道](/assets/posts/basic-pipeline.png)
关键特性：

- 自动化测试
- 自动化部署
- 在生产环境中管理微服务

单体与微服务的模块部署有差异：
![微服务部署](/assets/posts/micro-deployment.png)

### 容错设计

使用服务作为组件的一个**后果**：
应用程序需要被设计成能够**容忍服务失效**。任何服务调用都可能因为服务提供者不可用而失败，客户端必须尽可能优雅地应对这种失败(消息方式)。
为应对随时可能失败的服务，微服务：

1. 快速检测故障
2. 自动恢复服务
3. 应用程序的实时监测：包括性能指标(如TPS)和业务指标(如订单数)
4. 告警系统：通知开发团队跟进和调查

微服务希望看到为每个单独的服务设置完善的监控和日志记录：

1. 控制面板上显示启动/关闭状态
2. 各种各样的运营和业务相关指标
3. 断路器（熔断器）状态
4. 当前吞吐量和时延

### 演进式设计

演进式设计承认难以对边界进行正确定位，所以它将工作的重点放到了易于对边界进行重构之上。
微服务将服务的分解，使得在软件中**让变化发生得频繁、快速**且经过了**良好的控制**。
分割应用的原则：**组件可独立地更换和升级**。
> 如开发一个页面用于报道一个体育赛事：快速整合、赛事结束可被删除。

微服务强调**可替代性**，通过**变更模式**来驱动模块化：**同时变化的东西保持在同一模块中**。系统中很少变更的部分应该和正在经历大量扰动的部分放在不同的服务里。如果你发现自己不断地一起改变两个服务，这是它们应该被合并的一个标志。

**优点：**

- 更加精细的软件发布计划
- 简化并加快发布过程：只需要重新部署修改过的那些服务就够了

**缺点:**

- 需考虑服务发生变化时，依赖它的其他服务可能无法工作

**解决方法：**

- 把各个服务设计得尽量能够容错
- 版本控制

## 展望

技术大会圈子充满了各种各样的**正在转向微服务**的公司案例。

也有人觉得微服务或许很难成熟起来：

- 组件化成功与否取决于软件与组件的匹配程度。准确地搞清楚某个组件的**边界**的位置应该出现在哪里，是一件困难的工作。
- 当各个组件成为各个进行远程通信的服务后，比起在单一进程内进行各个软件库之间的调用，此时的**重构就变得更加困难**。跨越服务边界的代码移动就变得困难起来：接口变化需要在其各个参与者之间进行协调，向后兼容的层次也需要被添加进来，测试也会变得更加复杂。
- 如果这些组件不能干净利落地组合成一个系统，那么所做的一切工作，仅仅是将组件内的复杂性**转移**到组件之间的连接之上，导致服务之间杂乱的连接。
- 不一定适用于一个技术略逊一筹的**团队**。一个糟糕的团队，会构建一个糟糕的系统。

**一个合理的说法:**
不要一上来就以微服务架构做为起点。相反，要用一个单块系统做为起点,并保持其模块化。当这个单块系统出现了问题后，再将其分解为微服务。
