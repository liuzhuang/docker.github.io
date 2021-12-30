<!-- This text will be included as part of the Deploy your app section in the Language-specific guides -->

现在，我们已经配置了 CI/CD 管道，让我们看看如何部署应用程序。Docker 支持在 Azure ACI 和 AWS ECS 上部署容器。如果您在 Docker Desktop 中启用了 Kubernetes，您还可以将您的应用程序部署到 Kubernetes。

## Docker 和 Azure ACI

Docker Azure 集成使开发人员能够在构建云原生应用程序时使用原生 Docker 命令在 Azure 容器实例 (ACI) 中运行应用程序。新体验提供了 Docker Desktop 和 Microsoft Azure 之间的紧密集成，允许开发人员使用 Docker CLI 或 VS Code 扩展快速运行应用程序，从本地开发无缝切换到云部署。

有关详细说明，请参阅 [Deploying Docker containers on Azure](/cloud/aci-integration/)。

## Docker 和 AWS ECS

Docker ECS 集成使开发人员能够在构建云原生应用程序时使用 Docker Compose CLI 中的原生 Docker 命令在 Amazon EC2 Container Service (ECS) 中运行应用程序。

Docker 和 Amazon ECS 之间的集成允许开发人员使用 Docker Compose CLI 在一个 Docker 命令中设置 AWS 上下文，允许您从本地上下文切换到云上下文并快速轻松地运行应用程序，简化多容器应用程序开发在 Amazon ECS 上使用 Compose 文件。

有关详细说明，请参阅 [Deploying Docker containers on ECS](/cloud/ecs-integration/)。

## Kubernetes


Docker Desktop 包括一个独立的 Kubernetes 服务器和客户端，以及在您的机器上运行的 Docker CLI 集成。启用 Kubernetes 后，您可以在 Kubernetes 上测试您的工作负载。

要启用 Kubernetes：

1. 从 Docker 菜单中，选择 **Preferences** （**Settings**）。
2。 选择 **Kubernetes** 并单击 **Enable Kubernetes**。

这会在 Docker Desktop 启动时启动 Kubernetes 单节点集群。

有关详细信息，请参阅 [Deploy on Kubernetes](/desktop/kubernetes/) 和[Describing apps using Kubernetes YAML](/get-started/kube-deploy/#describing-apps-using-kubernetes-yaml)。