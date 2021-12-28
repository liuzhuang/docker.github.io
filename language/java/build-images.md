---
title: "构建你的 Java 镜像"
keywords: Java, build, images, dockerfile
description: Learn how to build your first Docker image by writing a Dockerfile
---

{% include_relative nav.html selected="1" %}

## 先决条件

阅读 [Part 1](../../get-started/index.md){:target="_blank" rel="noopener" class="_"} 中的目标和安装了解 Docker 概念。有关 Java 先决条件，请参阅以下部分。

{% include guides/enable-buildkit.md %}

## 概览

现在我们对容器和 Docker 平台有了一个很好的了解，让我们来看看构建我们的第一个镜像。镜像包括运行应用程序所需的一切——代码或二进制文件、运行时、依赖项以及所需的任何其他文件系统对象。

要完成本教程，您需要具备以下条件：

- 本地运行的 Docker。按照说明[下载并安装 Docker](../../get-docker.md)
- 一个 Git 客户端
- 用于编辑文件的 IDE 或文本编辑器。我们建议使用 [IntelliJ Community Edition](https://www.jetbrains.com/idea/download/){: target="_blank" rel="noopener" class="_"}。


## 示例应用程序

让我们将我们将在此模块中使用的示例应用程序克隆到我们的本地开发机器。在终端中运行以下命令来克隆 repo。

```console
$ cd /path/to/working/directory
$ git clone https://github.com/spring-projects/spring-petclinic.git
$ cd spring-petclinic
```

## 在没有 Docker 的情况下测试应用程序（可选）

在这一步中，我们将在没有 Docker 的情况下在本地测试应用程序，然后继续使用 Docker 构建和运行应用程序。本节要求您在您的机器上安装 Java OpenJDK 15 或更高版本。 [Download and install Java](https://jdk.java.net/){: target="_blank" rel="noopener" class="_"}

如果你不想在你的机器上安装 Java，你可以跳过这一步，直接继续下一节，我们将解释如何在 Docker 中构建和运行应用程序，这不需要你在你的机器上安装 Java。


让我们启动我们的应用程序并确保它正常运行。
Maven 将管理所有项目流程（编译、测试、打包等）。
我们之前克隆的 **Spring Pets Clinic** 项目包含一个嵌入式版本的 Maven。因此，我们不需要在您的本地机器上单独安装 Maven。

打开终端并进入到我们创建的工作目录并运行以下命令：

```console
$ ./mvnw spring-boot:run
```

这将下载依赖项、构建项目并启动它。

要测试应用程序是否正常工作，请打开一个新浏览器并访问 `http://localhost:8080`。

切换回服务器运行的终端，您应该在服务器日志中看到以下请求。您机器上的数据会有所不同。

```console
o.s.s.petclinic.PetClinicApplication     : Started
PetClinicApplication in 11.743 seconds (JVM running for 12.364)
```

太好了! 我们验证了该应用程序是否有效。在这个阶段，你已经完成了在本地测试服务器脚本。

在命令行中按下 `CTRL-c`，停止应用程序

我们现在将继续在 Docker 中构建和运行应用程序。

## 为 Java 创建一个 Dockerfile

{% include guides/create-dockerfile.md %}

接下来，我们需要在我们的 Dockerfile 中添加一行，告诉 Docker 我们希望为我们的应用程序使用什么基础镜像。

```dockerfile
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13
```

Docker 镜像可以从其他镜像继承。在本指南中，我们使用来自 Docker Hub 的官方镜像和 Java JDK `openjdk`，它已经拥有运行 Java 应用程序所需的所有工具和包。

为了在运行其余命令时更轻松，让我们设置镜像的工作目录。这会指示 Docker 使用此路径作为所有后续命令的默认位置。通过这样做，我们不必输入完整的文件路径，而是可以使用基于工作目录的相对路径。

```dockerfile
WORKDIR /app
```

通常，一旦您下载了使用 Maven 进行项目管理的 Java 编写的项目，您要做的第一件事就是安装依赖项。


在我们可以运行 `mvnw dependency` 之前，我们需要将 Maven 包装器和我们的 `pom.xml` 文件放入我们的镜像中。
我们将使用 `COPY` 命令来执行此操作。该 `COPY` 命令有两个参数。
第一个参数告诉 Docker 您想要复制到镜像中的文件。
第二个参数告诉 Docker 您希望将该文件复制到何处。我们将所有这些文件和目录复制到我们的工作目录 - `/app`。

```dockerfile
COPY .mvn/ .mvn
COPY mvnw pom.xml ./
```

一旦我们在镜像有了 `pom.xml` 文件，我们就可以使用 `RUN` 命令来执行命令 `mvnw dependency:go-offline`。这与我们在本地机器上运行 `mvnw` (（或 `mvn`）依赖项的方式完全相同，但这次依赖项将安装到镜像中。

```dockerfile
RUN ./mvnw dependency:go-offline
```

此时，我们有一个基于 OpenJDK 16 版的 Alpine 3.13 版镜像，并且我们还安装了我们的依赖项。我们需要做的下一件事是将我们的源代码添加到镜像中。我们将使用 `COPY` 命令就像我们对上面的 `pom.xml` 文件。

```dockerfile
COPY src ./src
```

此 `COPY` 命令获取位于当前目录中的所有文件并将它们复制到映像中。
现在，我们要做的就是告诉 Docker 当我们的镜像在容器内执行时我们想要运行什么命令。
我们使用 `CMD` 命令来执行此操作。

```dockerfile
CMD ["./mvnw", "spring-boot:run"]
```

这是完整的 Dockerfile。

```dockerfile
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline

COPY src ./src

CMD ["./mvnw", "spring-boot:run"]
```

### 创建 `.dockerignore` 文件

为了提高构建的性能，作为一般的最佳实践，我们建议您 `.dockerignore` 在与 Dockerfile 相同的目录中创建一个文件。对于本教程，您的 `.dockerignore` 文件应仅包含一行：

```
target
```

这一行 `target` 从 Docker 构建上下文中排除了包含 Maven 输出的目录。仔细构建 `.dockerignore`  文件有很多很好的理由，但这个单行文件现在已经足够了。


## 构建镜像

现在我们已经创建了我们的 Dockerfile，让我们构建我们的镜像。为此，我们使用 `docker build` 命令。该 `docker build` 命令从 Dockerfile 和 “context” 构建 Docker 镜像。
构建的上下文是位于指定 PATH 或 URL 中的一组文件。Docker 构建过程可以访问位于此上下文中的任何文件。

build 命令可选地接收一个` --tag` 标志。
标签用于设置镜像的名称和格式中的可选标签name:tag。
我们暂时不使用可选选项 `tag` 以帮助简化事情。
如果我们不传递 tag，Docker 将使用 “latest” 作为其默认 tag。
您可以在构建输出的最后一行看到这一点。

让我们构建我们的第一个 Docker 镜像。

```console
$ docker build --tag java-docker .
```

```console
Sending build context to Docker daemon  5.632kB
Step 1/7 : FROM java:3.7-alpine
Step 2/7 : WORKDIR /app
...
Successfully built a0bb458aabd0
Successfully tagged java-docker:latest
```

## 查看本地镜像

要查看本地机器上的镜像列表，我们有两个选项。一种是使用 CLI，另一种是使用 Docker Desktop。由于我们目前在 terminal 中，让我们来看看使用 CLI 列出镜像。

要列出镜像，只需运行 `docker images` 命令。

```console
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED          SIZE
java-docker         latest              b1b5f29f74f0        47 minutes ago   567MB
```

您应该看到我们刚刚构建的 `java-docker:latest`.

## 给镜像打标签

镜像名称由斜杠分隔的名称组件组成。名称组件可能包含小写字母、数字和分隔符。分隔符定义为句点、一个或两个下划线或一个或多个破折号。名称组件不能以分隔符开头或结尾。

镜像由清单和层列表组成。除了“标签”指向这些工件的组合外，此时不要过多担心清单和层。一个镜像可以有多个标签。让我们为我们构建的镜像创建第二个标签并查看它的层。

要为我们在上面构建的镜像创建新标签，请运行以下命令：

```console
$ docker tag java-docker:latest java-docker:v1.0.0
```

`docker tag`  命令为镜像创建一个新标签。它不会创建新镜像。标签指向同一张图片，只是引用图片的另一种方式。

现在，运行该 `docker images` 命令以查看我们本地镜像的列表。

```console
$ docker images
REPOSITORY    TAG      IMAGE ID		  CREATED		  SIZE
java-docker   latest   b1b5f29f74f0	  59 minutes ago	567MB
java-docker   v1.0.0   b1b5f29f74f0	  59 minutes ago	567MB
```

您可以看到我们有两个以 `java-docker` 开头的镜像。我们知道它们是相同的镜像，因为如果您查看该`IMAGE ID`列，您会发现这两个镜像的值相同。

让我们删除刚刚创建的标签。为此，我们将使用该 `rmi` 命令。该 `rmi` 命令代表“删除镜像”。

```console
$ docker rmi java-docker:v1.0.0
Untagged: java-docker:v1.0.0
```

请注意，来自 Docker 的响应告诉我们该映像尚未删除，而只是 “untagged”。您可以通过运行 `docker images` 命令来检查这一点。


```console
$ docker images
REPOSITORY      TAG     IMAGE ID        CREATED              SIZE
java-docker    	latest	b1b5f29f74f0	59 minutes ago	     567MB
```

我们标记为 `:v1.0.0`  镜像已被删除，但我们 `java-docker:latest `的机器上仍有可用的标签。


## 下一步

在本模块中，我们了解了如何设置我们将在本教程的其余部分中使用的示例 Java 应用程序。我们还创建了一个用于构建 Docker 镜像的 Dockerfile。然后，我们研究了标记镜像和删除镜像。在下一个模块中，我们将看看如何：

[Run your image as a container](run-containers.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.

<br />
