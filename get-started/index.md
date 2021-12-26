---
title: "目标和安装"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Get oriented on some basics of Docker and install Docker Desktop.
redirect_from:
- /engine/getstarted-voting-app/
- /engine/getstarted-voting-app/cleanup/
- /engine/getstarted-voting-app/create-swarm/
- /engine/getstarted-voting-app/customize-app/
- /engine/getstarted-voting-app/deploy-app/
- /engine/getstarted-voting-app/node-setup/
- /engine/getstarted-voting-app/test-drive/
- /engine/getstarted/
- /engine/getstarted/last_page/
- /engine/getstarted/step_five/
- /engine/getstarted/step_four/
- /engine/getstarted/step_one/
- /engine/getstarted/step_six/
- /engine/getstarted/step_three/
- /engine/getstarted/step_two/
- /engine/quickstart/
- /engine/tutorials/
- /engine/tutorials/dockerimages/
- /engine/tutorials/dockerizing/
- /engine/tutorials/usingdocker/
- /engine/userguide/containers/dockerimages/
- /engine/userguide/dockerimages/
- /engine/userguide/intro/
- /get-started/part1/
- /get-started/part5/
- /get-started/part6/
- /getstarted/
- /getting-started/
- /learn/
- /linux/last_page/
- /linux/started/
- /linux/step_four/
- /linux/step_one/
- /linux/step_six/
- /linux/step_three/
- /linux/step_two/
- /mac/last_page/
- /mac/started/
- /mac/step_four/
- /mac/step_one/
- /mac/step_six/
- /mac/step_three/
- /mac/step_two/
- /userguide/dockerimages/
- /userguide/dockerrepos/
- /windows/last_page/
- /windows/started/
- /windows/step_four/
- /windows/step_one/
- /windows/step_six/
- /windows/step_three/
- /windows/step_two/
---

> **Docker Desktop 更新条款**
>
> 大型组织中 Docker Desktop 的专业使用 (超过250名员工或年收入超过1000万美元) 要求用户拥有付费的Docker订阅。虽然这些条款的生效日期是2021年8月31日，但对于那些需要付费订阅的条款，宽限期到2022年1月31日。
> 有关更多信息，请参阅博客[Docker is Updating and Extending Our Product Subscriptions](https://www.docker.com/blog/updating-product-subscriptions/){: target="_blank" rel="noopener" class="_" id="dkr_docs_cta"}。

欢迎！我们很高兴您想学习 Docker。

本页包含有关如何开始使用 Docker的分步说明。在本教程中您将学习如何:

- 将镜像构建并运行成为容器
- 使用 Docker Hub 共享镜像
- 部署带有数据库的多个容器应用
- 使用 Docker Compose 运行应用程序

此外，您还将了解构建镜像的最佳实践，包括有关如何扫描镜像以查找安全漏洞的说明。

如果您正在寻找有关如何使用自己喜欢的语言对应用程序进行容器化的信息，请参阅 [Language-specific getting started guides](../language/index.md)。

我们还推荐 DockerCon 2020 的视频演练。

<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/iqqDU2crIEQ?start=30" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## 下载并安装 Docker

本教程假设您的机器上安装了当前版本的 Docker。如果您没有安装 Docker，请在下面选择您喜欢的操作系统来下载 Docker：

[Mac with Intel chip](https://desktop.docker.com/mac/main/amd64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-amd64){: .button .primary-btn }
[Mac with Apple chip](https://desktop.docker.com/mac/main/arm64/Docker.dmg?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-mac-arm64){: .button .primary-btn }
[Windows](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-win-amd64){: .button .primary-btn }
[Linux](../engine/install/index.md){: .button .primary-btn}

有关 Docker Desktop 安装说明，请参阅[在 Mac 上安装 Docker Desktop](../desktop/mac/install.md) 和[在 Windows 上安装 Docker Desktop](../desktop/windows/install.md)。

## 开始教程

如果您已经运行命令以开始本教程，恭喜! 如果不是，请打开命令提示或 bash 窗口，然后运行命令:

```console
$ docker run -d -p 80:80 docker/getting-started
```

你会注意到一些 `flags` 被使用了。这里有一些关于它们的更多信息:

- `-d` - 在分离模式下运行容器 (在后台)
- `-p 80:80` - 将主机的 80 端口映射到容器中的 80 端口
- `docker/getting-started` - 要使用的镜像

> **Info**
>
> 您可以组合单个字符标志以缩短完整命令。例如，上面的命令可以写成:
>
> ```console
> $ docker run -dp 80:80 docker/getting-started
> ```

## Docker Dashboard

在走得太远之前，我们想突出显示 `Docker Dashboard`，它可以让您快速查看机器上运行的容器。`Docker Dashboard` 可用于 Mac 和 Windows。
它让您可以快速访问容器日志，让您在容器内获得一个 shell，并让您轻松管理容器生命周期（停止、删除等）。

要访问 `Dashboard`，请按照 [Docker Desktop 手册](../desktop/dashboard.md) 中的说明进行操作。如果您打开仪表板现在，您将看到本教程正在运行容器 (`jolly_bouman`) 是一个随机创建的名称。所以，你很可能会有一个不同的名字。

![Tutorial container running in Docker Dashboard](images/tutorial-in-dashboard.png)

## 什么是容器？

现在你已经运行了一个容器，什么是容器？简而言之，容器是您机器上的沙盒进程，与主机上的所有其他进程隔离。
这种隔离利用了 [kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504) 技术，这些特性已经在 Linux 中存在了很长时间。Docker 一直致力于使这些功能易于使用且易于使用。总结一下，一个容器：

- 是一个镜像的可运行实例。您可以使用 DockerAPI 或 CLI 创建、启动、停止、移动或删除容器。
- 可以在本地机器、虚拟机上运行，也可以部署到云端。
- 可移植（可以在任何操作系统上运行）
- 容器彼此隔离，并运行自己的软件、二进制文件和配置。

> **从头开始创建容器**
>
> 如果您想了解容器是如何从头开始构建的，Aqua Security 的 Liz Rice 有一个精彩的演讲，其中她用 Go 从头开始​​，创建了一个容器。
> 虽然演讲不进入网络，使用图像的文件系统，以及其他高级主题，它给出了一个梦幻般的深入了解事情是如何工作的。
> 
> <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/8fi7uSYlOdc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## 什么是容器镜像？

运行容器时，它使用隔离的文件系统。此自定义文件系统由容器映像提供。
由于镜像包含容器的文件系统，它必须包含运行应用程序所需的一切——所有依赖项、配置、脚本、二进制文件等。
镜像还包含容器的其他配置，例如环境变量、要运行的默认命令、和其他元数据。

稍后，我们将更深入地研究镜像，涵盖分层、最佳实践等主题。

> **Info**
>
> 如果你熟悉 `chroot`，可以认为容器是 `chroot` 的一个扩展版本。
> 文件系统仅仅来自镜像，但是相比 `chroot`，Docker 增加了额外的隔离性

## CLI references

Refer to the following topics for further documentation on all CLI commands used in this article:

- [docker version](../engine/reference/commandline/version.md)
- [docker run](../engine/reference/commandline/run.md)
- [docker image](../engine/reference/commandline/image.md)
- [docker container](../engine/reference/commandline/container.md)

