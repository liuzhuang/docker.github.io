---
title: "下一步是什么"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Making sure you have more ideas of what you could do next with your application
---

尽管我们的讲习班已经完成，但关于容器的知识还有很多东西要学! 我们不打算在这里深入研究，但是接下来还有一些其他的领域要看!

## 容器编排

在生产中运行容器很困难。您不想登录到一台机器并简单地运行一个 `docker run` 或 `docker-compose up`. 为什么不？那么，如果容器死了会发生什么？您如何跨多台机器进行扩展？容器编排解决了这个问题。Kubernetes、Swarm、Nomad 和 ECS 等工具都有助于解决这个问题，但方式略有不同。

一般的想法是你有“经理”谁接收预期的状态。此状态可能是“我想运行我的 Web 应用程序的两个实例并公开端口 80”。然后管理人员查看集群中的所有机器并将工作委托给“工作”节点。管理人员观察变化（例如容器退出），然后努力使实际状态反映预期状态。

## 云原生计算基础项目

CNCF 是各种开源项目的供应商中立之家，包括 Kubernetes、Prometheus、Envoy、Linkerd、NATS 等！您可以在此处查看已毕业和孵化的项目 以及[在此处]((https://landscape.cncf.io/){:target="_blank" rel="noopener" class="_"})查看整个CNCF 景观。有很多项目可以帮助解决监控、日志记录、安全、镜像注册、消息传递等方面的问题！

因此，如果您不熟悉容器领域和云原生应用程序开发，欢迎您！请与社区联系，提出问题，并继续学习！我们很高兴有你！