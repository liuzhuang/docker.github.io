---
description: Controlling and configuring Docker using systemd
keywords: docker, daemon, systemd, configuration
redirect_from:
- /articles/host_integration/
- /articles/systemd/
- /engine/admin/systemd/
- /engine/articles/systemd/
title: 使用 systemd 控制 Docker
---

许多 Linux 发行版使用 systemd 来启动 Docker 守护进程。本文档展示了一些如何自定义 Docker 设置的示例。

## 启动 Docker 守护进程

### 手动启动

安装 Docker 后，您需要启动 Docker 守护程序。大多数 Linux 发行版用于systemctl启动服务。

```console
$ sudo systemctl start docker
```

### 在系统启动时自动启动

如果您希望 Docker 在启动时启动，请参阅 [配置 Docker 以在启动时启动](../../engine/install/linux-postinstall.md#configure-docker-to-start-on-boot).。

## 自定义 Docker 守护进程选项

有多种方法可以为 Docker 守护程序配置守护程序标志和环境变量。推荐的方式是使用与平台无关的 daemon.json文件，该文件/etc/docker/默认位于Linux 中。请参阅 守护程序配置文件。

您可以使用daemon.json. 以下示例配置了两个选项。您无法使用daemon.json机制配置的一件事是HTTP 代理。


### 运行时目录和存储驱动程序

您可能希望通过将其移动到单独的分区来控制用于 Docker 镜像、容器和卷的磁盘空间。

为此，请在daemon.json文件中设置以下标志：

```json
{
    "data-root": "/mnt/docker-data",
    "storage-driver": "overlay2"
}
```

### HTTP/HTTPS 代理

泊坞窗守护程序使用HTTP_PROXY，HTTPS_PROXY以及NO_PROXY在其启动环境环境变量来配置HTTP或HTTPS代理的行为。您不能使用该daemon.json文件配置这些环境变量。

此示例覆盖默认docker.service文件。

如果您使用 HTTP 或 HTTPS 代理服务器，例如在公司设置中，则需要在 Docker systemd 服务文件中添加此配置。


> **Note for rootless mode**
>
> 在rootless 模式下运行 Docker 时，systemd 配置文件的位置是不同的。在无根模式下运行时，Docker 作为用户模式的 systemd 服务启动，并使用存储在~/.config/systemd/user/docker.service.d/. 此外，systemctl必须在没有sudo和有--user 标志的情况下执行。如果您在无根模式下运行 Docker，请选择下面的“无根模式”选项卡。

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" data-target="#rootful">regular install</a></li>
  <li><a data-toggle="tab" data-target="#rootless">rootless mode</a></li>
</ul>
<div class="tab-content">
<div id="rootful" class="tab-pane fade in active" markdown="1">

1.  为 docker 服务创建一个 systemd 插入目录：

    ```console
    $ sudo mkdir -p /etc/systemd/system/docker.service.d
    ```

2.  创建一个名为/etc/systemd/system/docker.service.d/http-proxy.conf 添加HTTP_PROXY环境变量的文件：

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    ```

    如果您使用 HTTPS 代理服务器，请设置HTTPS_PROXY环境变量：

    ```systemd
    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    ```
    
    可以设置多个环境变量；设置非 HTTPS 和 HTTPS 代理；

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    ```
     
3.  如果您有内部 Docker 注册表需要联系而无需代理，您可以通过NO_PROXY环境变量指定它们。

    该NO_PROXY变量指定一个字符串，其中包含应从代理中排除的主机的逗号分隔值。这些是您可以指定以排除主机的选项：
    * IP 地址前缀 ( 1.2.3.4)
    * 域名，或特殊的 DNS 标签 ( *) 域名与该名称和所有子域相匹配。以“.”开头的域名 仅匹配子域。例如，给定域 foo.example.com和example.com：
        * example.com匹配example.com和foo.example.com，和
        * .example.com 只匹配 foo.example.com
    * 单个星号 ( *) 表示不应进行代理
    * IP 地址前缀 ( 1.2.3.4:80) 和域名 ( foo.example.com:80)接受文字端口号
    
    配置示例：

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
    ```

4.  刷新更改并重新启动 Docker

    ```console
    $ sudo systemctl daemon-reload
    $ sudo systemctl restart docker
    ```

5.  验证配置是否已加载并与您所做的更改匹配，例如：

    ```console
    $ sudo systemctl show --property=Environment docker
    
    Environment=HTTP_PROXY=http://proxy.example.com:80 HTTPS_PROXY=https://proxy.example.com:443 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
    ```

</div>
<div id="rootless" class="tab-pane fade in" markdown="1">

1.  为 docker 服务创建一个 systemd 插入目录：

    ```console
    $ mkdir -p ~/.config/systemd/user/docker.service.d
    ```

2.  创建一个名为~/.config/systemd/user/docker.service.d/http-proxy.conf 添加HTTP_PROXY环境变量的文件：

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    ```

    如果您使用 HTTPS 代理服务器，请设置HTTPS_PROXY环境变量：

    ```systemd
    [Service]
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    ```
    
    可以设置多个环境变量；设置非 HTTPS 和 HTTPS 代理；

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    ```
     
3.  如果您有需要在没有代理的情况下联系的内部 Docker 注册表，您可以通过NO_PROXY环境变量指定它们。

    该NO_PROXY变量指定一个字符串，其中包含应从代理中排除的主机的逗号分隔值。这些是您可以指定以排除主机的选项：
    * IP 地址前缀 ( 1.2.3.4)
    * 域名，或特殊的 DNS 标签 ( *) 域名与该名称和所有子域相匹配。以“.”开头的域名 仅匹配子域。例如，给定域 foo.example.com和example.com：
        * example.com匹配example.com和foo.example.com，和
        * .example.com 只匹配 foo.example.com
    * 单个星号 ( *) 表示不应进行代理
    * IP 地址前缀 ( 1.2.3.4:80) 和域名 ( foo.example.com:80)接受文字端口号

    配置示例：

    ```systemd
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80"
    Environment="HTTPS_PROXY=https://proxy.example.com:443"
    Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
    ```

4.  刷新更改并重新启动 Docker

    ```console
    $ systemctl --user daemon-reload
    $ systemctl --user restart docker
    ```

5.  验证配置是否已加载并与您所做的更改匹配，例如：

    ```console
    $ systemctl --user show --property=Environment docker

    Environment=HTTP_PROXY=http://proxy.example.com:80 HTTPS_PROXY=https://proxy.example.com:443 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
    ```

</div>
</div> <!-- tab-content -->


## 配置 Docker 守护进程在哪里监听连接

请参阅 [配置 Docker 守护程序侦听连接的位置](../../engine/install/linux-postinstall.md#configure-where-the-docker-daemon-listens-for-connections).。

## 手动创建 systemd 单元文件

在没有包的情况下安装二进制文件时，您可能希望将 Docker 与 systemd 集成。为此，将两个单元文件（service和socket）从github 存储库安装 到/etc/systemd/system.
