---
description: Docker Compose FAQ
keywords: documentation, docs,  docker, compose, faq
title: Frequently asked questions
---

如果您在此处没有看到您的问题，请随时访问 [Docker Community Slack](https://dockr.ly/slack) 上的 [#docker-compose](https://dockercommunity.slack.com/archives/C2X82D9PA)。

## 我可以控制服务的启动顺序吗？

可以  - 请参阅 [Controlling startup order](startup-order.md).


## 为什么我的服务需要 10 秒才能重新创建或停止？

Compose 尝试通过发送 `SIGTERM` 停止一个容器. 然后等待 [10 秒的默认超时时间](reference/stop.md)。
超时后，再向容器发送一个 `SIGKILL` 信号强行杀死它。如果您正在等待此超时，则意味着您的容器在收到 `SIGTERM` 信号时不会关闭。

已经有很多在容器中处理关于进程信号的[文章](https://medium.com/@gchudnov/trapping-signals-in-docker-containers-7a57fdda7d86)。

要解决此问题，请尝试以下操作：

* 确保在你的 Dockerfile 中，exec 使用 `CMD` 和 `ENTRYPOINT` 的格式。

  例如使用 `["program", "arg1", "arg2"]` 而不是 `"program arg1 arg2"`。
  使用字符串形式会导致 Docker 运行您的进程，使用bash它不能正确处理信号。
  Compose 始终使用 JSON 形式，因此如果您覆盖了 Compose 文件中的命令或入口点，请不要担心。

* 如果可以，请修改您正在运行的应用程序，为 `SIGTERM` 编写信号处理 handler。

* 设置 `stop_signal` 参数，值是应用程序知道如何处理的信号：

```yaml
services:
  web:
    build: .
    stop_signal: SIGINT
```

* If you can't modify the application, wrap the application in a lightweight init
system (like [s6](https://skarnet.org/software/s6/)) or a signal proxy (like
[dumb-init](https://github.com/Yelp/dumb-init) or
[tini](https://github.com/krallin/tini)).  Either of these wrappers takes care of
handling `SIGTERM` properly.

## 如何在同一主机上运行 Compose 文件的多个副本？

Compose 使用项目名称为项目的所有容器和其他资源创建唯一标识符。
要运行一个项目的多个副本，请使用 [`-p` command line option](reference/index.md) 或 [`COMPOSE_PROJECT_NAME` environment variable](reference/envvars.md#compose_project_name) 设置自定义项目名称。

## `up`, `run`, 和 `start` 之间的区别？

通常，你希望使用 `docker-compose up`。 
使用 `up` 启动或重新启动 `docker-compose.yml` 定义的所有服务。
在默认的 "attached" 模式下，您会看到来自所有容器的所有日志。
在 "attached" 模式 (`-d`) 下，Compose 在启动容器后退出，但容器继续在后台运行。

`docker-compose run` 命令用于运行“一次性”或“临时”任务。
它需要您要运行的服务名称，并且只为正在运行的服务所依赖的服务启动容器。
使用 `run` 运行测试或执行管理任务，如删除或添加数据的数据量的容器。
该 `run`  命令的作用类似于 `docker run -ti` 它向容器打开一个交互式终端，并返回与容器中进程的退出状态匹配的退出状态。

`docker-compose start` 命令仅用于重新启动先前创建但已停止的容器。它从不创建新容器。



## 我可以在 Compose 文件中使用 json 而不是 yaml 吗？

是的。[Yaml 是 json 的超集](https://stackoverflow.com/a/1729545/444646)，因此任何 JSON 文件都应该是有效的 Yaml。要在 Compose 中使用 JSON 文件，请指定要使用的文件名，例如：

```console
$ docker-compose -f docker-compose.json up
```

## 我应该用`COPY`/`ADD`包含我的 code 或 volume 吗？

你可以在 Dockerfile 中使用 `COPY` 和 `ADD` 指令把代码添加到镜像中。这将非常有用，当你需要在镜像中移动你的代码，例如，把你的代码发送到另外的环境（生产，CI，等等）。


如果您想对代码进行更改并立即看到它们的反映，则应该使用 `volume`，例如，当您正在开发代码并且您的服务器支持热代码重新加载或实时重新加载时。

在某些情况下，您可能想同时使用两者。您可以在镜像中使用 `COPY`，并在您的 Compose 文件中使用 `volume`。volume 会覆盖镜像的目录内容。

## 我在哪里可以找到 compose 示例文件？

[GitHub](https://github.com/search?q=in%3Apath+docker-compose.yml+extension%3Ayml&type=Code) 上有很多 Compose 的示例文件。

## Compose documentation

- [User guide](index.md)
- [Installing Compose](install.md)
- [Getting Started](gettingstarted.md)
- [Command line reference](reference/index.md)
- [Compose file reference](compose-file/index.md)
- [Sample apps with Compose](samples-for-compose.md)
