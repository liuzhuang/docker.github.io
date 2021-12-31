---
title: Docker 开发最佳实践
description: Rules of thumb for making your life easier as a Docker application developer
keywords: application, development
---

以下开发模式已被证明有助于人们使用 Docker 构建应用程序。如果您发现了我们应该添加的内容，
请 [let us know](https://github.com/docker/docker.github.io/issues/new){: target="_blank" rel="noopener" class="_"}。

## 如何让你的镜像变小

在启动容器或服务时，小图像可以更快地通过网络拉动并加载到内存中。有一些经验法则可以保持较小的图像尺寸：

- 从适当的基础映像开始。例如，如果您需要 JDK，请考虑将您的镜像基于官方`openjdk` 镜像，而不是从通用 `ubuntu`镜像开始，并将 `openjdk` 作为 Dockerfile 的一部分进行安装。

- 使用多阶段构建。例如，您可以使用该maven映像来构建您的 Java 应用程序，然后重置为该tomcat映像并将 Java 工件复制到正确的位置以部署您的应用程序，所有这些都在同一个 Dockerfile 中。这意味着您的最终映像不包含构建引入的所有库和依赖项，而仅包含运行它们所需的工件和环境。

    ```dockerfile
    RUN apt-get -y update
    RUN apt-get install -y python
    ```

    ```dockerfile
    RUN apt-get -y update && apt-get install -y python
    ```

  - 如果您需要使用不包含多阶段构建的 Docker 版本，请尝试通过最小化RUNDockerfile中单独命令的数量来减少映像中的层数。您可以通过将多个命令合并为RUN一行并使用您的 shell 机制将它们组合在一起来实现这一点。考虑以下两个片段。第一个在图像中创建两个图层，而第二个仅创建一个。

  - 如果您有多个具有很多共同点的图像，请考虑使用共享组件创建您自己的 基本图像，并以此为基础构建您的独特图像。Docker 只需要加载一次公共层，它们就会被缓存。这意味着您的衍生镜像可以更有效地使用 Docker 主机上的内存并更快地加载。

- 为了使您的生产映像保持精简但允许调试，请考虑使用生产映像作为调试映像的基础映像。可以在生产映像之上添加额外的测试或调试工具。

- 在构建镜像时，总是用有用的标签来标记它们，这些标签编码了版本信息、预期目的地（prod或test，例如）、稳定性或在不同环境中部署应用程序时有用的其他信息。不要依赖自动创建的latest标签。



- Start with an appropriate base image. For instance, if you need a JDK,
  consider basing your image on the official `openjdk` image, rather than
  starting with a generic `ubuntu` image and installing `openjdk` as part of the
  Dockerfile.

- [Use multistage builds](develop-images/multistage-build.md). For
  instance, you can use the `maven` image to build your Java application, then
  reset to the `tomcat` image and copy the Java artifacts into the correct
  location to deploy your app, all in the same Dockerfile. This means that your
  final image doesn't include all of the libraries and dependencies pulled in by
  the build, but only the artifacts and the environment needed to run them.

  - If you need to use a version of Docker that does not include multistage
    builds, try to reduce the number of layers in your image by minimizing the
    number of separate `RUN` commands in your Dockerfile. You can do this by
    consolidating multiple commands into a single `RUN` line and using your
    shell's mechanisms to combine them together. Consider the following two
    fragments. The first creates two layers in the image, while the second
    only creates one.

    ```dockerfile
    RUN apt-get -y update
    RUN apt-get install -y python
    ```

    ```dockerfile
    RUN apt-get -y update && apt-get install -y python
    ```

- If you have multiple images with a lot in common, consider creating your own
  [base image](develop-images/baseimages.md) with the shared
  components, and basing your unique images on that. Docker only needs to load
  the common layers once, and they are cached. This means that your
  derivative images use memory on the Docker host more efficiently and load more
  quickly.

- To keep your production image lean but allow for debugging, consider using the
  production image as the base image for the debug image. Additional testing or
  debugging tooling can be added on top of the production image.

- When building images, always tag them with useful tags which codify version
  information, intended destination (`prod` or `test`, for instance), stability,
  or other information that is useful when deploying the application in
  different environments. Do not rely on the automatically-created `latest` tag.

## 在哪里以及如何持久化应用程序数据

- 避免使用存储驱动程序将应用程序数据存储在容器的可写层中 。这会增加容器的大小，并且从 I/O 的角度来看，与使用卷或绑定安装相比效率较低。
- 相反，使用卷存储数据。
- 一种适合使用 绑定挂载的情况是在开发期间，当您可能想要挂载源目录或刚刚构建到容器中的二进制文件时。对于生产，请改用卷，将其安装到与开发期间安装绑定安装相同的位置。
- 对于生产，使用secrets存储服务使用的敏感应用程序数据，并使用configs 存储 非敏感数据，例如配置文件。如果您当前使用独立容器，请考虑迁移到使用单副本服务，以便您可以利用这些仅限服务的功能。


- **Avoid** storing application data in your container's writable layer using
  [storage drivers](../storage/storagedriver/select-storage-driver.md). This increases the
  size of your container and is less efficient from an I/O perspective than
  using volumes or bind mounts.
- Instead, store data using [volumes](../storage/volumes.md).
- One case where it is appropriate to use
  [bind mounts](../storage/bind-mounts.md) is during development,
  when you may want to mount your source directory or a binary you just built
  into your container. For production, use a volume instead, mounting it into
  the same location as you mounted a bind mount during development.
- For production, use [secrets](../engine/swarm/secrets.md) to store sensitive
  application data used by services, and use [configs](../engine/swarm/configs.md)
  for non-sensitive data such as configuration files. If you currently use
  standalone containers, consider migrating to use single-replica services, so
  that you can take advantage of these service-only features.


## 使用 CI/CD 进行测试和部署

- 当您签入对源代码控制的更改或创建拉取请求时，请使用 [Docker Hub](../docker-hub/builds/index.md) 或其他 CI/CD 管道自动构建和标记 Docker 映像并对其进行测试。

- 通过要求您的开发、测试和安全团队在将镜像部署到生产中之前对其进行 [sign images](../engine/reference/commandline/trust.md)，可以进一步实现这一点。这样，在将映像部署到生产中之前，它已经过测试并由开发、质量和安全团队等人员签署。

## 开发环境和生产环境的区别

| 开发环境 | 生产环境 |
|:-------------------|:---------------------|
| 使用绑定挂载让您的容器可以访问您的源代码。  | 使用卷来存储容器数据。 |
| 使用 Docker Desktop for Mac 或 Docker Desktop for Windows。| 使用 Docker 引擎，如果可能的话，使用 [userns mapping](../engine/security/userns-remap.md) 来更好地隔离 Docker 进程与主机进程。|
| 不用担心时间漂移。 | 不用担心时间漂移。	始终在 Docker 主机和每个容器进程中运行 NTP 客户端，并将它们全部同步到同一个 NTP 服务器。如果您使用 swarm 服务，还要确保每个 Docker 节点将其时钟同步到与容器相同的时间源。 |
