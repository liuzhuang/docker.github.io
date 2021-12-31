---
title: "部署到 Kubernetes"
keywords: kubernetes, pods, deployments, kubernetes services
description: Learn how to describe and deploy a simple application on Kubernetes.
---

## 先决条件

- 按照 [Orientation and setup](index.md) 的说明下载并安装 Docker 桌面。
- 完成 [Part 2](02_our_app.md) 中的应用程序容器化。
- 确保在您的 Docker 桌面上启用了 Kubernetes：
  -  Mac：单击菜单栏中的 Docker 图标，导航到首选项并确保“Kubernetes”旁边有一个绿灯。
  -  Windows：单击系统托盘中的 Docker 图标并导航到设置并确保“Kubernetes”旁边有一个绿灯。

  如果 Kubernetes 未运行，请按照本教程的[Orchestration](orchestration.md)中的说明完成设置。

## 简介

现在我们已经证明了我们应用程序的各个组件作为独立容器运行，是时候安排它们由 Kubernetes 这样的编排器管理了。Kubernetes 提供了许多用于扩展、联网、保护和维护容器化应用程序的工具，这些工具超出了容器本身的能力。

为了验证我们的容器化应用程序在 Kubernetes 上运行良好，我们将在我们的开发机器上使用 Docker Desktop 的内置 Kubernetes 环境来部署我们的应用程序，然后将其移交给生产中的完整 Kubernetes 集群上运行。Docker Desktop 创建的 Kubernetes 环境功能齐全，这意味着它具有您的应用程序将在真实集群上享受的所有 Kubernetes 功能，可从您的开发机器上方便地访问。

## 使用 Kubernetes YAML 描述应用程序

Kubernetes 中的所有容器都被调度为pods，它们是共享某些资源的并置容器组。此外，在实际应用中，我们几乎从不创建单独的 Pod；相反，我们的大部分工作负载都被安排为部署，这是由 Kubernetes 自动维护的可扩展的 pod 组。最后，所有 Kubernetes 对象都可以而且应该在称为Kubernetes YAML文件的清单中进行描述。这些 YAML 文件描述了 Kubernetes 应用程序的所有组件和配置，可用于在任何 Kubernetes 环境中轻松创建和销毁您的应用程序。

1.  您已经在本教程的编排概述部分编写了一个非常基本的 Kubernetes YAML 文件。现在，让我们编写一个稍微复杂一点的 YAML 文件来运行和管理我们的公告板。将以下内容放入名为 的文件中bb.yaml：

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: bb-demo
      namespace: default
    spec:
      replicas: 1
      selector:
        matchLabels:
          bb: web
      template:
        metadata:
          labels:
            bb: web
        spec:
          containers:
          - name: bb-site
            image: getting-started
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: bb-entrypoint
      namespace: default
    spec:
      type: NodePort
      selector:
        bb: web
      ports:
      - port: 3000
        targetPort: 3000
        nodePort: 30001
    ```

    在这个 Kubernetes YAML 文件中，我们有两个对象，用  `---` 分隔：
    - A Deployment，描述一组可扩展的相同 pod。在这种情况下，您将只获得一个replica或 pod 的副本，并且该 pod（在template:密钥下描述）中只有一个容器，基于bulletinboard:1.0本教程上一步中的图像。
    - 一项NodePort服务，它将流量从主机上的端口 30001 路由到它路由到的 Pod 内的端口 3000，从而允许您从网络访问您的公告板。

    另外，请注意，虽然 Kubernetes YAML 一开始可能显得冗长而复杂，但它几乎总是遵循相同的模式：
    - The `apiVersion`, 表示Kubernetes API，它分析该对象
    - The `kind` 该kind指示的排序对象的这个是什么
    - Some `metadata` 一些metadata将诸如名称之类的东西应用于您的对象
    - The `spec` 在spec指定所有对象的参数和配置。

## 部署并检查您的应用程序

1.  在终端中，导航到您创建的位置bb.yaml并将应用程序部署到 Kubernetes：

    ```console
    $ kubectl apply -f bb.yaml
    ```

    您应该会看到如下所示的输出，表明您的 Kubernetes 对象已成功创建：

    ```shell
    deployment.apps/bb-demo created
    service/bb-entrypoint created
    ```

2.  通过列出您的部署确保一切正常：

    ```console
    $ kubectl get deployments
    ```

    如果一切顺利，您的部署应如下列出：

    ```shell
    NAME      READY   UP-TO-DATE   AVAILABLE   AGE
    bb-demo   1/1     1            1           40s
    ```

    这表明您在 YAML 中要求的所有 Pod 都已启动并正在运行。对您的服务进行同样的检查：

    ```console
    $ kubectl get services

    NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    bb-entrypoint   NodePort    10.106.145.116   <none>        3000:30001/TCP   53s
    kubernetes      ClusterIP   10.96.0.1        <none>        443/TCP          138d
    ```

    除了默认kubernetes服务之外，我们还看到了我们的bb-entrypoint服务，接受端口 30001/TCP 上的流量。

3.  打开浏览器并访问您的公告板localhost:30001；您应该会看到您的公告板，这与我们在快速入门教程的第 2部分中将其作为独立容器运行时的情况相同。

4.  满意后，拆除您的应用程序：

    ```console
    $ kubectl delete -f bb.yaml
    ```

## 结论

至此，我们已经成功地使用 Docker Desktop 将我们的应用程序部署到我们开发机器上功能齐全的 Kubernetes 环境中。我们还没有对 Kubernetes 做太多事情，但现在大门敞开了；您可以开始向您的应用程序添加其他组件并利用 Kubernetes 的所有功能和强大功能，就在您自己的机器上。

除了部署到 Kubernetes，我们还将我们的应用程序描述为 Kubernetes YAML 文件。这个简单的文本文件包含我们在运行状态下创建应用程序所需的一切。我们可以将其签入版本控制并与我们的同事共享，从而使我们能够轻松地将我们的应用程序分发到其他集群（例如可能在我们的开发环境之后出现的测试和生产集群）。

## Kubernetes 参考

本文中使用的所有新 Kubernetes 对象的更多文档可在此处获得：

 - [Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/)
 - [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
 - [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
