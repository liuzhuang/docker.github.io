---
description: Best practices for using Docker Hub for CI/CD
keywords: CI/CD, GitHub Actions,
title: 将 Docker Hub 用于 CI/CD 的最佳实践
---

根据 [2020 Jetbrains developer survey](https://www.jetbrains.com/lp/devecosystem-2020/){:target="_blank" rel="noopener" class="_"}，44% 的开发人员现在正在使用某种形式的 Docker 容器持续集成和部署。我们了解到，大量开发人员已经使用 Docker Hub 作为其部分工作流程的容器注册表进行了设置。本指南包含执行此操作的一些最佳实践，并提供有关如何开始的指南。

我们还听到反馈，考虑到 [Docker introduced](https://www.docker.com/blog/scaling-docker-to-serve-millions-more-developers-network-egress/){:target="_blank" rel="noopener" class="_"}  的与网络出口和免费用户拉取数量相关的更改，关于在不达到这些限制的情况下使用 Docker Hub 作为 CI/CD 工作流的一部分的最佳方式存在问题。本指南涵盖了改善您的体验的最佳实践，并使用 Docker Hub 的合理消耗来降低达到这些限制的风险，并包含有关如何根据您的用例增加限制的提示。

## 内循环和外循环

首先，使用 Docker 和任何 CI/CD 时最重要的事情之一是了解何时需要使用 CI 进行测试，以及何时可以在本地执行此操作。在 Docker，我们从内循环（代码、构建、运行、测试）和外循环（推送更改、CI 构建、CI 测试、部署）的角度考虑开发人员如何工作。

![CI/CD inner and outer loop](images/inner-outer-loop.png)

在考虑优化 CI/CD 之前，重要的是要考虑您的内循环以及它与外循环（CI）的关系。我们知道大多数用户不喜欢“通过 CI 调试”。因此，您的内循环和外循环越相似越好。我们建议您docker build通过在 Dockerfile 中为它们添加目标来将单元测试作为命令的一部分运行。这样，当您在本地进行更改和重建时，您可以使用简单的命令运行与在本地计算机上的 CI 中运行相同的单元测试。

博客文章 [Go development with Docker](https://www.docker.com/blog/tag/go-env-series/){:target="_blank" rel="noopener" class="_"} 是一个很好的例子，说明如何在 Docker 项目中使用测试并在 CI 中重用它们。这也为问题创建了一个更短的反馈循环，并减少了拉动和构建您的 CI 需要做的工作量。

## 优化 CI/CD 部署

进入实际的外循环和 Docker Hub 后，您可以做一些事情来充分利用 CI 并提供最快的 Docker 体验。

首先，保持安全。设置 CI 时，请确保使用 Docker Hub 访问令牌，而不是密码。


  > **Note**
  >
  > 您可以从 Docker Hub 上的 [Security](https://hub.docker.com/settings/security){:target="_blank" rel="noopener" class="_"} 页面创建新的访问令牌。

创建访问令牌并将其添加到平台上的机密存储后，您需要考虑何时推送和拉入 CI/CD，以及从哪里开始，具体取决于您所做的更改。

为了减少构建时间和减少调用次数，您可以做的第一件事是利用构建缓存来重用您已经拉取的层。您可以通过使用 buildX (buildkits) 缓存功能和您的平台提供的任何缓存在许多平台上执行此操作。例如，请参阅[使用构建缓存优化 GitHub 操作工作流](../github-actions#optimizing-the-workflow)。

您可能想要进行的另一个更改是仅将您的发布映像转到 Docker Hub。这意味着设置功能将您的 PR 图像推送到更本地的图像存储，以便快速提取和测试，而不是将它们一直推广到生产。

## 下一步

我们知道在 CI 中使用 Docker 有很多技巧和窍门，但是，考虑到最近的 Docker Hub 速率限制更新，我们认为这些是一些重要的事情。

  > **Note**
  >
  > 如果您在通过身份验证后仍然遇到拉取限制问题，您可以考虑升级到Docker 订阅。

有关如何配置 GitHub 操作 CI/CD 管道的信息，请参阅 [Configure GitHub Actions](github-actions.md)。