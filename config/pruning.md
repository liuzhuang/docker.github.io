---
description: 清理未使用的 Docker 对象
keywords: pruning, prune, images, volumes, containers, networks, disk, administration, garbage collection
title: Prune unused Docker objects
redirect_from:
- /engine/admin/pruning/
---

Docker 采取保守的方法来清理未使用的对象（通常称为“垃圾收集”），例如图像、容器、卷和网络：除非您明确要求 Docker 这样做，否则通常不会删除这些对象。这会导致 Docker 使用额外的磁盘空间。对于每种类型的对象，Docker 都提供了一个prune命令。此外，您可以使用docker system prune一次清理多种类型的对象。本主题展示了如何使用这些prune命令。

## Prune images

该docker image prune命令允许您清理未使用的图像。默认情况下，docker image prune只清理悬空图像。悬空图像是未标记且未被任何容器引用的图像。要删除悬空图像：

```console
$ docker image prune

WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
```

要删除现有容器未使用的所有图像，请使用以下-a 标志：

```console
$ docker image prune -a

WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示您继续。要绕过提示，请使用-f或 --force标志。

您可以使用带有--filter标志的过滤表达式来限制修剪哪些图像 。例如，仅考虑 24 小时前创建的图像：

```console
$ docker image prune -a --filter "until=24h"
```

其他过滤表达式可用。有关 更多示例，请参阅 [`docker image prune` reference](../engine/reference/commandline/image_prune.md) 参考资料。

## Prune containers

当您停止容器时，它不会自动删除，除非您使用--rm标志启动它。要查看 Docker 主机上的所有容器，包括停止的容器，请使用docker ps -a. 您可能会惊讶于存在多少容器，尤其是在开发系统上！停止容器的可写层仍然占用磁盘空间。要清理它，您可以使用该docker container prune命令。

```console
$ docker container prune

WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示您继续。要绕过提示，请使用-f或 --force标志。

默认情况下，所有停止的容器都会被删除。您可以使用--filter标志限制范围。例如，以下命令仅删除超过 24 小时的停止容器：

```console
$ docker container prune --filter "until=24h"
```

其他过滤表达式可用。有关 更多示例，请参阅[`docker container prune` reference](../engine/reference/commandline/container_prune.md) 参考资料。

## Prune volumes

卷可以被一个或多个容器使用，并占用 Docker 主机上的空间。卷永远不会自动删除，因为这样做可能会破坏数据。

```console
$ docker volume prune

WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示您继续。要绕过提示，请使用-f或 --force标志。

默认情况下，删除所有未使用的卷。您可以使用--filter标志限制范围。例如，以下命令仅删除未标有keep标签的卷：

```console
$ docker volume prune --filter "label!=keep"
```

其他过滤表达式可用。有关 更多示例，请参阅 [`docker volume prune` reference](../engine/reference/commandline/volume_prune.md) 参考资料。


## Prune networks

Docker 网络不会占用太多磁盘空间，但它们确实会创建iptables 规则、桥接网络设备和路由表条目。要清理这些东西，您可以使用docker network prune清理未被任何容器使用的网络。

```console
$ docker network prune

WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示您继续。要绕过提示，请使用-f或 --force标志。

默认情况下，删除所有未使用的网络。您可以使用--filter标志限制范围。例如，以下命令仅删除超过 24 小时的网络：

```console
$ docker network prune --filter "until=24h"
```

其他过滤表达式可用。有关 更多示例，请参阅 [`docker network prune` reference](../engine/reference/commandline/network_prune.md) 参考资料。

## Prune everything

该docker system prune命令是修剪图像、容器和网络的快捷方式。默认情况下不修剪卷，您必须指定--volumes用于docker system prune修剪卷的 标志。

```console
$ docker system prune

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] y
```

要同时修剪卷，请添加--volumes标志：

```console
$ docker system prune --volumes

WARNING! This will remove:
        - all stopped containers
        - all networks not used by at least one container
        - all volumes not used by at least one container
        - all dangling images
        - all build cache
Are you sure you want to continue? [y/N] y
```

默认情况下，系统会提示您继续。要绕过提示，请使用-f或 --force标志。