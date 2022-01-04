---
title: 使用 host 网络进行联网
description: Tutorials for networking using the host network, disabling network isolation
keywords: networking, host, standalone
---

本系列教程涉及网络独立容器，这些容器直接绑定到 Docker 主机的网络，没有网络隔离。
有关其他网络主题，请参阅[概述](index.md)。

## 目标

本教程的目标是启动一个 `nginx` 直接绑定到 Docker 主机上的 80 端口的容器。
从网络的角度来看，这与 `nginx` 进程直接在 Docker 主机上运行而不是在容器中运行的隔离级别相同。
但是，在所有其他方式中，例如存储、进程命名空间和用户命名空间，`nginx` 进程与主机是隔离的。


## 先决条件

- 此过程要求 Docker 主机上的端口 80 可用。
  要让 Nginx 侦听不同的端口，请参阅[`nginx`容器文档](https://hub.docker.com/_/nginx/)

- 在 `host` 网络驱动程序仅适用于 Linux 主机，并且不支持 Docker Desktop for Mac, Docker Desktop for Windows, 或 Docker EE for Windows Server.

## 步骤

1.  创建并启动容器作为一个分离的进程。
    `--rm` 选项意味着一旦容器退出/停止就将其移除。
    `-d` 标志意味着启动容器分离（在后台）。

    ```console
    $ docker run --rm -d --network host --name my_nginx nginx
    ```

2.  浏览 http://localhost:80/ 访问 Nginx。

3.  使用以下命令检查您的网络堆栈：

    - 检查所有网络接口并确认未创建新接口。

      ```console
      $ ip addr show
      ```

    - 使用 `netstat` 命令验证哪个进程绑定到 80 端口。
      您需要使用，`sudo` 因为该进程归 Docker 守护程序用户所有，否则您将无法看到其名称或 PID。

      ```console
      $ sudo netstat -tulpn | grep :80
      ```

4.  停止容器。它将在使用 `--rm` 选项启动时自动删除。

    ```basn
    docker container stop my_nginx
    ```

## 其他网络资源

现在您已经完成了独立容器的网络教程，您可能想要浏览这些其他网络教程：

- [Standalone networking tutorial](network-tutorial-standalone.md)
- [Overlay networking tutorial](network-tutorial-overlay.md)
- [Macvlan networking tutorial](network-tutorial-macvlan.md)

