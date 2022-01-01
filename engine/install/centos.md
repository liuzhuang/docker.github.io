---
description: Instructions for installing Docker Engine on CentOS
keywords: requirements, apt, installation, centos, rpm, install, uninstall, upgrade, update
redirect_from:
- /ee/docker-ee/centos/
- /engine/installation/centos/
- /engine/installation/linux/centos/
- /engine/installation/linux/docker-ce/centos/
- /engine/installation/linux/docker-ee/centos/
- /install/linux/centos/
- /install/linux/docker-ce/centos/
- /install/linux/docker-ee/centos/
title: 在 CentOS 上安装 Docker Engine
toc_max: 4
---

要在 CentOS 上开始使用 Docker Engine，请确保满足 [先决条件](#先决条件),，然后 [安装 Docker](#installation-methods)。


## 先决条件

### 操作系统要求

要安装 Docker 引擎，您需要 CentOS 7 或 8 的维护版本。不支持或测试存档版本。

`centos-extras` 库必须启用。默认情况下启用此存储库，但如果您已禁用它，则需要 [重新启用它](https://wiki.centos.org/AdditionalResources/Repositories){: target="_blank" rel="noopener" class="_" }。

推荐使用 `overlay2` 存储驱动。

### 卸载旧版本

旧版本的 Docker 被称为 `docker` 或 `docker-engine`。如果安装了这些，请卸载它们以及相关的依赖项。


```console
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

如果 `yum` 报告没有安装这些软件包，那也没关系。

`/var/lib/docker/` 中的 images, containers, volumes, 和 networks 将被保留。

Docker Engine 的包现在被称为 `docker-ce`.

## 安装方式

您可以根据需要以不同方式安装 Docker Engine：

- 大多数用户使用 [使用 repository 安装](#使用 repository 安装) 并从中安装，以便于安装和升级任务。
  这是推荐的方法。

- 一些用户下载 RPM 包并 [install it manually](#install-from-a-package) 并完全手动管理升级。
  这在某些情况下非常有用，例如在无法访问互联网的气隙系统上安装 Docker。

- 在测试和开发环境中，部分用户选择使用自动化 [convenience scripts](#install-using-the-convenience-script) 来安装Docker。

### 使用 repository 安装

在新主机上首次安装 Docker Engine 之前，您需要设置 Docker repository。之后，您可以从 repository 安装和更新 Docker。

#### 设置 repository

{% assign download-url-base = "https://download.docker.com/linux/centos" %}

安装 `yum-utils` 包（提供 `yum-config-manager` 实用程序）并设置 **stable** repository。

```console
$ sudo yum install -y yum-utils

$ sudo yum-config-manager \
    --add-repo \
    {{ download-url-base }}/docker-ce.repo
```

> **Optional**: 启用 **nightly** 或者 **test** repositories.
>
> 这些存储库包含在 `docker.repo` 上面的文件中，但默认情况下是禁用的。您可以在稳定存储库旁边启用它们。
> 以下命令启用 **nightly** repository。
>
> ```console
> $ sudo yum-config-manager --enable docker-ce-nightly
> ```
>
> 要启用 **test**  通道，请运行以下命令：
>
> ```console
> $ sudo yum-config-manager --enable docker-ce-test
> ```
>
> 您可以通过运行带有标志的命令来禁用 **nightly** 或 **test** repository 。
> 要重新启用它，请使用该标志。`yum-config-manager` `--disable` `--enable` 
>
> 以下命令禁用 **nightly** repository
>
> ```console
> $ sudo yum-config-manager --disable docker-ce-nightly
> ```
>
> [了解 **nightly** 和 **test** 渠道](index.md).

#### 安装 Docker Engine

1.  安装 _最新版本_ 的 Docker Engine 和 containerd，或者进入下一步安装特定的版本：

    ```console
    $ sudo yum install docker-ce docker-ce-cli containerd.io
    ```

    如果提示接受 GPG 密钥，请验证指纹是否匹配  `060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35`，如果匹配 ，请接受。

    > 有多个 Docker repositories？
    >
    > 如果您启用了多个 Docker 存储库，则`yum install`或 `yum update`  命令中未指定版本的情况下安装或更新始终会安装可能的最高版本，这可能不适合您的稳定性需求。

    此命令会安装 Docker，但不会启动 Docker。它还会创建一个 `docker` 组，但是，默认情况下它不会向该组添加任何用户。

2.  要安装特定版本的 Docker Engine，请在 repo 中列出可用版本，然后选择并安装：

    a. 列出并排序您的存储库中可用的版本。本示例按版本号对结果进行排序，从高到低，并被截断：

    ```console
    $ yum list docker-ce --showduplicates | sort -r

    docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
    docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
    docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
    docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
    ```

    返回的列表取决于启用的存储库，并且特定于您的 CentOS 版本（在本示例中由后缀 `.el7` 表示）。

    b. 通过完全限定的包名称安装特定版本，即包名称 (`docker-ce`) 加上从第一个冒号 (`:`)开始的版本字符串（第 2 列），直到第一个连字符，由连字符 (`-`)分隔。例如，`docker-ce-18.09.1`。

    ```console
    $ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
    ```

    此命令会安装 Docker，但不会启动 Docker。它还会创建一个 `docker` 组，但是，默认情况下它不会向该组添加任何用户。

3.  启动 Docker。

    ```console
    $ sudo systemctl start docker
    ```

4.  通过运行 `hello-world` 镜像验证 Docker Engine 是否已正确安装。

    ```console
    $ sudo docker run hello-world
    ```

    此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条消息并退出。

这将安装并运行 Docker 引擎。
使用`sudo` 运行 Docker 命令。继续 [Linux postinstall](linux-postinstall.md) 以允许非特权用户运行 Docker 命令和其他可选配置步骤。

#### 更新 Docker Engine

要升级 Docker Engine，请按照[installation instructions](#install-using-the-repository)，选择要安装的新版本。

### 通过 package 安装

如果您无法使用 Docker 的存储库来安装 Docker，您可以下载该`.rpm` 版本的 文件并手动安装。每次要升级 Docker Engine 时都需要下载一个新文件。

1. 前往 [{{ download-url-base }}/]({{ download-url-base }}/){: target="_blank" rel="noopener" class="_" } 并选择您  的 CentOS 版本。然后浏览`x86_64/stable/Packages/`并下载要安装的 Docker 版本的 `.rpm` 文件。

    > **Note**
    >
    > 要安装 **nightly** 或 **test** (预发布) 包，请将上述 URL 中的 `stable` 单词更改为 `nightly` 或 `test`。 
    > [Learn about **nightly** and **test** channels](index.md).

2.  安装 Docker Engine，将下面的路径更改为您下载 Docker 包的路径。

    ```console
    $ sudo yum install /path/to/package.rpm
    ```

    Docker 已安装但未启动。该docker组被创建，但没有用户添加到组

3.  启动 Docker。

    ```console
    $ sudo systemctl start docker
    ```

4.  通过运行hello-world 映像验证 Docker Engine 是否已正确安装。

    ```console
    $ sudo docker run hello-world
    ```

    此命令下载测试映像并在容器中运行它。当容器运行时，它会打印一条消息并退出。

这将安装并运行 Docker 引擎。使用sudo运行泊坞窗命令。继续Linux 的安装后步骤以允许非特权用户运行 Docker 命令和其他可选配置步骤。

#### 升级 Docker Engine

要升级 Docker Engine，请下载更新的包文件并重复 [installation procedure](#install-from-a-package)，使用 `yum -y upgrade` 代替 `yum -y install`，并指向新文件。

{% include install-script.md %}

## 卸载 Docker Engine

1.  卸载 Docker Engine、CLI 和 Containerd 包：

    ```console
    $ sudo yum remove docker-ce docker-ce-cli containerd.io
    ```

2.  主机上的映像、容器、卷或自定义配置文件不会自动删除。删除所有镜像、容器和卷：

    ```console
    $ sudo rm -rf /var/lib/docker
    $ sudo rm -rf /var/lib/containerd
    ```

您必须手动删除任何已编辑的配置文件。

## 下一步

- Continue to [Post-installation steps for Linux](linux-postinstall.md).
- Review the topics in [Develop with Docker](../../develop/index.md) to learn how to build new applications using Docker.
