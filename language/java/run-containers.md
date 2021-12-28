---
title: "将你的镜像作为容器运行"
keywords: Java, run, image, container,
description: Learn how to run the image as a container.
---

{% include_relative nav.html selected="2" %}

## 先决条件

浏览构建 Java 镜像 [Build your Java image](build-images.md).

## 概览

在上一个模块中，我们创建了示例应用程序，然后创建了用于构建镜像的 Dockerfile。
我们使用命令 `docker build` 创建了我们的镜像。现在我们有了一个镜像，我们可以运行该镜像，然后看看我们的应用程序是否正常运行。


容器是一个正常的操作系统进程，除了这个进程是隔离的，并且有自己的文件系统、自己的网络以及与主机分离的隔离进程树。

要在容器中运行镜像，我们使用 `docker run` 命令。`docker run` 命令需要一个参数，该参数是镜像的名称。让我们开始我们的镜像，并确保它正确运行。在终端中运行以下命令:

```console
$ docker run java-docker
```

运行此命令后，您会注意到我们没有返回到命令提示符。这是因为我们的应用程序是一个 REST 服务器，并且在循环中运行，等待传入的请求，而不会将控制权返回给操作系统，直到我们停止容器。

让我们打开一个新终端，然后使用 `curl` 命令向服务器发出 `GET` 请求。

```console
$ curl --request GET \
--url http://localhost:8080/actuator/health \
--header 'content-type: application/json'
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

如您所见，我们的 `curl` 命令失败了，因为与我们的服务器的连接被拒绝。这意味着我们无法在本地主机上连接到 8080 端口。这是意料之中的，因为我们的容器是独立运行的，包括网络。让我们停止容器并使用我们本地网络上发布的端口 8080 重新启动。

要停止容器，请按 `ctrl-c`。这将使您返回到终端提示。

要为我们的容器发布一个端口，我们将在 `docker run` 命令中使用 `--publish flag` (简写 `-p`) 。`--publish` 命令的格式为 `[host port]:[container port]`. 
因此，如果我们想将容器内的 8000 端口暴露给容器外的 8080端口，我们将传递 `8080:8000` 给 `--publish` 标志。

启动容器并将端口 8080 暴露给主机上的 8080 端口。

```console
$ docker run --publish 8080:8080 java-docker
```

现在，让我们从上面重新运行 curl 命令。

```console
$ curl --request GET \
--url http://localhost:8080/actuator/health \
--header 'content-type: application/json'
{"status":"UP"}
```

成功！我们能够通过 8080 端口连接到在容器内运行的应用程序。

现在，按 ctrl-c 停止容器。

## 在分离模式下运行

到目前为止，这很棒，但我们的示例应用程序是一个 Web 服务器，我们不必连接到容器。
Docker 可以在分离模式或后台运行您的容器。为此，我们可以使用 `--detach` 或者简称 `-d`。Docker 会像之前一样启动您的容器，但这一次，它将与容器 “分离” 并返回到终端提示符。

```console
$ docker run -d -p 8080:8080 java-docker
5ff83001608c7b787dbe3885277af018aaac738864d42c4fdf5547369f6ac752
```

Docker 在后台启动我们的容器并在终端上打印容器 ID。

同样，让我们确保容器正常运行。运行上面相同的 curl 命令。

```console
$ curl --request GET \
--url http://localhost:8080/actuator/health \
--header 'content-type: application/json'
{"status":"UP"}
```

## 容器列表

当我们在后台运行我们的容器时，我们如何知道我们的容器是否正在运行，或者我们的机器上正在运行哪些其他容器？
我们可以运行 `docker ps` 命令。就像我们如何在 Linux 中运行 `ps` 命令查看我们机器上的进程列表，我们可以运行 `docker ps` 命令查看在我们计算机上运行的容器列表。

```console
$ docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                    NAMES
5ff83001608c   java-docker      "./mvnw spring-boot:…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp   trusting_beaver
```

`docker ps` 命令提供了有关我们正在运行的容器的一堆信息。
我们可以看到容器 ID、容器内运行的镜像、用于启动容器的命令、容器创建时间、状态、暴露的端口和容器名称。

您可能想知道我们容器的名称从何而来。由于我们在启动时没有为容器提供名称，因此 Docker 生成了一个随机名称。
我们将在一分钟内解决这个问题，但首先我们需要停止容器。
要停止容器，请运行执行此操作的 `docker stop` 命令，停止容器。我们需要传递容器的名称，或者我们可以使用容器 ID。

```console
$ docker stop trusting_beaver
trusting_beaver
```

现在，重新运行 `docker ps`命令查看正在运行的容器列表。

```console
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

## 停止、启动和给容器起名

您可以启动、停止和重新启动 Docker 容器。
当我们停止一个容器时，它并没有被移除，而是状态变成了stopped，容器内的进程也停止了。
当我们在上一个模块中运行 `docker ps` 命令时，默认输出仅显示正在运行的容器。
当我们通过 `--all` 或简写 `-a` 时，我们会看到机器上的所有容器，而不管它们的启动或停止状态。

```console
$ docker ps -a
CONTAINER ID   IMAGE               COMMAND                  CREATED          STATUS                        PORTS                    NAMES
5ff83001608c   java-docker         "./mvnw spring-boot:…"   5 minutes ago    Exited (143) 18 seconds ago                            trusting_beaver
630f2872ddf5   java-docker         "./mvnw spring-boot:…"   11 minutes ago   Exited (1) 8 minutes ago                               modest_khayyam
a28f9d587d95   java-docker         "./mvnw spring-boot:…"   17 minutes ago   Exited (1) 11 minutes ago                              lucid_greider
```

您现在应该看到列出了几个容器。这些是我们启动和停止但尚未删除的容器。

让我们重新启动刚刚停止的容器。找到我们刚刚停止的容器名称，并使用 `restart` 命令替换下面的容器名称。

```console
$ docker restart trusting_beaver
```

现在，使用`docker ps`命令再次列出所有容器。

```console
$ docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS                      PORTS                    NAMES
5ff83001608c   java-docker   "./mvnw spring-boot:…"   10 minutes ago   Up 2 seconds                0.0.0.0:8080->8080/tcp   trusting_beaver
630f2872ddf5   java-docker   "./mvnw spring-boot:…"   16 minutes ago   Exited (1) 13 minutes ago                            modest_khayyam
a28f9d587d95   java-docker   "./mvnw spring-boot:…"   22 minutes ago   Exited (1) 16 minutes ago                            lucid_greider
```

请注意，我们刚刚重新启动的容器已在分离模式下启动并暴露了端口 8080。另外，观察容器的状态是 “Up X seconds”。
当您重新启动容器时，它会以与最初启动时相同的标志或命令启动。

现在，让我们停止并移除我们所有的容器，看看如何修复随机命名问题。
找到您正在运行的容器的名称，并将以下命令中的名称替换为您系统上的容器名称。

```console
$ docker stop trusting_beaver
trusting_beaver
```

现在我们的容器已停止，让我们将其删除。当您移除容器时，容器内的进程将停止并且容器的元数据将被移除。

要删除容器，只需运行 `docker rm` 命令，传递容器名称的即可。
您可以使用单个命令将多个容器名称传递给命令。
同样，将以下命令中的容器名称替换为您系统中的容器名称。

```console
$ docker rm trusting_beaver modest_khayyam lucid_greider
trusting_beaver
modest_khayyam
lucid_greider
```

再次运行 `docker ps --all` 命令，可以查看到所有容器都已删除。

现在，让我们解决随机命名问题。
标准做法是为您的容器命名，原因很简单，这样更容易识别容器中运行的内容以及它关联的应用程序或服务。

要命名容器，我们只需要给 `docker run` 命令传递 `--name` 标志。

```console
$ docker run --rm -d -p 8080:8080 --name springboot-server java-docker
2e907c68d1c98be37d2b2c2ac6b16f353c85b3757e549254de68746a94a8a8d3
$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                    NAMES
2e907c68d1c9   java-docker   "./mvnw spring-boot:…"   8 seconds ago   Up 8 seconds   0.0.0.0:8080->8080/tcp   springboot-server
```

这样更好！我们现在可以根据名称轻松识别我们的容器。

## 下一步

在本模块中，我们了解了在分离模式下运行容器、发布端口和运行容器。
我们还研究了通过启动、停止和重新启动容器来管理容器。
我们还考虑了命名我们的容器，以便它们更容易识别。

在下一个模块中，我们将学习如何在容器中运行数据库并将其连接到我们的应用程序

[Use containers for development](develop.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.

<br />
