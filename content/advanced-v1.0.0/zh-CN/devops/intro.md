---
title: "DevOps 工程概述"
---

当今互联网产品研发的世界里唯快不破，迭代速度往往很快，有些企业甚至在一天时间内有多次迭代。在产品快速发展与迭代中，如何让项目产品平稳的落地，一套完善且可靠的持续集成 CI/CD 和 DevOps 解决方案就是多数企业和 IT 团队所期待的。

### KubeSphere DevOps 特点

相较于 Express Edition (易捷版)，DevOps 工程是 Advanced Edition (高级版) 独有的功能，针对企业实际的快速迭代和快速交付业务需求和场景，可以发现很多企业和 IT 团队都有持续集成和持续交付的需求。DevOps 工程提供从仓库 (SVN/Git)、代码编译、镜像制作、镜像安全、推送到仓库、应用版本、到定时构建的端到端流水线设置，支持用户在开发、测试等环境下的端到端高效流水线能力，支持用户成员管理，同时提供完整的日志功能，记录 CI/CD 流水线的每个过程。

对用户而言，KubeSphere DevOps 工程在高级版的 v1.0.0 已经具有这些功能亮点：

- 开箱即用的DevOps功能，无需对 Jenkins 进行复杂的插件配置
- 独立 DevOps 工程，提供访问可控、安全隔离的 CI/CD 操作空间
- 兼容 Jenkinsfile in & out of SCM（Source Code Management）两种模式
- 可视化流水线编辑工具，降低 CI/CD 学习成本
- 使用 KubeSphere 基于 Kubernetes 提供弹性、干净、可定制的构建环境

值得期待的是，KubeSphere DevOps 工程将在下一个版本 v2.0.0 中集成更多新的功能：

- 支持 source to image，快速交付容器镜像
- 多语言代码静态检查，持续提升代码质量

### 为什么要落地 DevOps

由于软件开发复杂度的增高和更多的协同工作，团队开发成员间如何更好地在协同工作中确保软件开发和交付质量，逐渐成为研发过程中不可回避的问题。众所周知，敏捷开发 (Agile) 在业内日趋流行，团队如何在不断变化的需求中快速适应和保证软件质量就变得极其重要了。而 CI/CD，就是专门为解决上述需求的软件开发实践。CI/CD 要求每次的集成都是通过自动化的构建来验证，包括自动编译、发布和测试，从而尽快地发现集成错误，让团队能够更快的开发内聚的软件，减轻了软件发布时的压力。

对企业而言，高效可靠的 CI/CD 流水线是一个 IT 组织实现软件应用与服务快速交付的基础，同时也是 DevOps 在企业落地的最佳实践。虽然业界流行的 CI/CD 工具很多，比如 Travis CI、Teamcity 和 Jenkins 等，但如今大量企业选择采用 Jenkins 来搭建其交付流水线。谈到 Jenkins，大部分人肯定耳熟能详，持续集成/持续交付，自动化部署工具，测试工具，以及丰富的插件支持，配合目前大多数开发工具, 简直是万能的瑞士军刀。

### 理解 KubeSphere DevOps

因此，在经过调研之后，KubeSphere 的 DevOps 工程选择基于 Jenkins 提供可视化的 CI/CD 流水线构建，或基于代码仓库已有的 Jenkinfile 构建流水线，正是因为 Jenkins 易配置、支持 Unit/TestNG 测试报告、支持分布式构建、支持众多第三方插件等特点。

软件开发的生命周期中，持续构建和发布是 IT 团队在日常工作中必不可少的步骤，目前很多公司也都选择 Jenkins 集群来搭建符合企业需求的 CI/CD 流程。但是，相比较传统的 Jenkins 集群一主多从的方式必然存在一些痛点：

- Master 一旦发生单点故障，那么整个 CI/CD 流水线就崩溃了
- 资源分配不均衡，有的 Slave 要运行的 job 出现排队等待，而有的 Slave 处于空闲状态
- 不同的 Slave 的配置环境可能不一样，需要完成不同语言的编译打包，这类差异化的配置进一步导致管理不便，维护起来也不是一件易事

KubeSphere 的 CI/CD 是基于底层 Kubernetes 的动态 Jenkins Slave，也就是说 Jenkins Slave 具有动态伸缩的能力，能够根据任务的执行状态进行动态创建或自动注销释放资源。实际上，Jenkins Master 和 Jenkins Slave 以 Pod 形式运行在 KubeSphere 集群的 Node 上，Master 运行在其中一个节点，并且将其配置数据存储到一个 Volume 中，Slave 运行在各个节点上，并且它不是一直处于运行状态，它会按照需求动态的创建并自动删除。

上述的工作流程可以理解为：当 Jenkins Master 接受到 Build 请求时，会根据配置的 Label 动态创建运行在 Pod 中的 Jenkins Slave 并注册到 Master 上，当这些 Slave 运行完任务后，就会被注销，并且相关的 Pod 也会自动删除，恢复到最初状态。

所以，这种动态的 Jenkins Slave 优势就显而易见了：

- 动态伸缩，合理使用资源，每次运行任务时，会自动创建一个 Jenkins Slave，任务完成后，Slave 自动注销并删除容器，资源自动释放，而且 KubeSphere 会根据每个资源的使用情况，动态分配 Slave 到空闲的节点上创建，降低出现因某节点资源利用率高，还排队等待在该节点的情况。
- 扩展性好，当 KubeSphere 集群的资源严重不足而导致任务排队等待时，可以很容易的添加一个 KubeSphere Node 到集群中，从而实现扩展。
- 高可用，当 Jenkins Master 出现故障时，KubeSphere 会自动创建一个新的 Jenkins Master 容器，并且将 Volume 分配给新创建的容器，保证数据不丢失，从而达到集群服务高可用。

### 快速上手 CI/CD

我们提供了两个具有代表性的示例和文档，帮助您快速上手 CI/CD。

- [示例一 - Jenkinsfile in SCM](../../devops/jenkinsfile-in-scm)

本示例以文档和视频演示如何通过 GitHub 仓库中的 Jenkinsfile 来创建 CI/CD 流水线，最终将一个文档网站部署到 KubeSphere 集群中的开发环境和产品环境，并且能够通过公网访问。

- [示例二 - Jenkinsfile out of SCM](../../devops/jenkinsfile-out-of-scm)

本示例以文档和视频演示如何基于 [示例一 - Jenkinsfile in SCM](../jenkinsfile-in-scm)，以可视化的方式构建 CI/CD 流水线 (包含示例一的前六个阶段)，最终将本文档网站部署到 KubeSphere 集群中的开发环境且能够通过公网访问。

### Jenkins Agent 说明

在 DevOps 工程中，KubeSphere 使用 Kubernetes Jenkins agent 来执行具体的构建。Agent 部分指定整个 Pipeline 或特定阶段将在 Jenkins 环境中执行的位置，具体取决于该 Agent 部分的放置位置。该部分必须在 Pipeline 块内的顶层或 stage 内部定义，详见 [Jenkins agent 说明](../jenkins-agent)。

### 高级设置

KubeSphere 使用了 Configuration-as-Code 进行 Jenkins 的系统设置，详见 [Jenkins 系统设置](../jenkins-setting)。


