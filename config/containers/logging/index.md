---
description: How to write to and view a container's logs
keywords: docker, logging
title: 查看容器或服务的日志
redirect_from:
- /engine/admin/logging/
- /engine/admin/logging/view_container_logs/
---

`docker logs` 命令显示正在运行的容器记录的信息。
`docker service logs` 命令显示参与服务的所有容器记录的信息。
记录的信息和日志的格式几乎完全取决于容器的端点命令。

默认情况下，`docker logs` 或 `docker service logs` 显示命令的输出，就像在终端中以交互方式运行命令一样。
UNIX 和 Linux 命令通常打开三个 I/O 流，所谓的 `STDIN`, `STDOUT`, 和 `STDERR`。
`STDIN` 是命令的输入流，它可能包括来自键盘的输入或来自另一个命令的输入。`STDOUT` 通常是正常输出， `STDERR` 通常用于输出错误消息。默认情况下，docker logs显示命令的 `STDOUT` 和 `STDERR`。
要阅读有关 I/O 和 Linux 的更多信息，请参阅[有关 I/O 重定向的 Linux 文档项目文章](https://tldp.org/LDP/abs/html/io-redirection.html)。

在某些情况下，`docker logs` 可能不会显示有用的信息，除非您采取其他步骤。

- 如果您使用了将日志发送到文件、外部主机、数据库或其他日志记录后端的[日志记录驱动程序](configure.md)，并且禁用了[“双日志记录”](dual-logging.md)，则 `docker logs`  可能不会显示有用的信息。
- 如果您的图像运行非交互式进程，例如 Web 服务器或数据库，则该应用程序可能会将其输出发送到日志文件而不是 `STDOUT` 和 `STDERR`.。

在第一种情况下，您的日志以其他方式处理，您可以选择不使用 `docker logs`. 
在第二种情况下，官方 `nginx` 镜像给出了一种解决方法，而官方 Apache `httpd` 镜像给出了另一种解决方法。

官方 `nginx` 镜像创建了一个从 `/var/log/nginx/access.log` 到 `/dev/stdout` 的符号链接，并创建了另一个从 `/var/log/nginx/error.log` 到 `/dev/stderr` 的符号链接，覆盖了日志文件，导致日志被发送到相关的特殊设备。
请参阅[Dockerfile](https://github.com/nginxinc/docker-nginx/blob/8921999083def7ba43a06fabd5f80e4406651353/mainline/jessie/Dockerfile#L21-L23)。


官方 `httpd` 驱动程序将 `httpd` 应用程序的配置更改为将其正常输出直接写入 `/proc/self/fd/1` （即`STDOUT`）并将其错误写入 `/proc/self/fd/2`（即`STDERR`）。
请参阅 [Dockerfile](https://github.com/docker-library/httpd/blob/b13054c7de5c74bbaa6d595dbe38969e6d4f860c/2.2/Dockerfile#L72-L75)。

## 下一步

- 配置 [日志驱动](configure.md).
- 编写一个 [Dockerfile](../../../engine/reference/builder.md).
