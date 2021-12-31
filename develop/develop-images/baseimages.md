---
description: How to create base images
keywords: images, base image, examples
redirect_from:
- /articles/baseimages/
- /engine/articles/baseimages/
- /engine/userguide/eng-image/baseimages/
title: 创建一个基础镜像
---

大多数 Dockerfile 从父映像开始。如果您需要完全控制图像的内容，则可能需要创建一个基本图像。这是区别：

- 一个[父镜像](../../glossary.md#parent-image)是你的形象是基于图像。它指的是`FROM` Dockerfile 中指令的内容。Dockerfile 中的每个后续声明都会修改此父映像。大多数 Dockerfile 从父映像开始，而不是从基础映像开始。但是，这些术语有时可以互换使用。

- 一个基础镜像在 Dockerfile 中有 `FROM scratch`。

本主题向您展示了创建基础映像的几种方法。具体过程将在很大程度上取决于您要打包的 Linux 发行版。我们在下面有一些示例，我们鼓励您提交拉取请求以贡献新的请求。

## 使用 tar 创建完整图像

通常，从运行您想要打包为父映像的发行版的工作机器开始，尽管这对于某些工具（例如 Debian 的 [Debootstrap](https://wiki.debian.org/Debootstrap)）不是必需的 ，您也可以使用它来构建 Ubuntu 映像。


创建 Ubuntu 父映像可以像这样简单：

    $ sudo debootstrap focal focal > /dev/null
    $ sudo tar -C focal -c . | docker import - focal

    sha256:81ec9a55a92a5618161f68ae691d092bf14d700129093158297b3d01593f4ee3

    $ docker run focal cat /etc/lsb-release

    DISTRIB_ID=Ubuntu
    DISTRIB_RELEASE=20.04
    DISTRIB_CODENAME=focal
    DISTRIB_DESCRIPTION="Ubuntu 20.04 LTS"

在 [the Docker GitHub repository](https://github.com/docker/docker/blob/master/contrib) 中有更多用于创建父映像的示例脚本。

## 使用 scratch 创建一个简单的父图像

您可以使用 Docker 保留的最小映像`scratch`,作为构建容器的起点。使用`scratch`“图像”信号向构建过程表明您希望 中的下一个命令`Dockerfile`成为图像中的第一个文件系统层。

虽然scratch出现在集线器上的 Docker 存储库中，但您无法拉取、运行它或使用名称标记任何图像scratch。相反，您可以在您的Dockerfile. 例如，要使用scratch以下命令创建最小容器 ：


```dockerfile
# syntax=docker/dockerfile:1
FROM scratch
ADD hello /
CMD ["/hello"]
```

假设您使用https://github.com/docker-library/hello-world 上的源代码构建了“hello”可执行示例 ，并使用-static标志编译它，您可以使用以下docker build命令构建此 Docker 映像：

```console
$ docker build --tag hello .
```

不要忘记`.`末尾的字符，它将构建上下文设置为当前目录。

> **Note**: 
>
> 由于 Docker Desktop for Mac 和 Docker Desktop for Windows 使用 Linux VM，
> 因此您需要 Linux 二进制文件，而不是 Mac 或 Windows 二进制文件。您可以使用 Docker 容器来构建它：
>
> ```console
> $ docker run --rm -it -v $PWD:/build ubuntu:20.04
>
> container# apt-get update && apt-get install build-essential
> container# cd /build
> container# gcc -o hello -static hello.c
> ```

要运行您的新映像，请使用以下`docker run`命令：

```console
$ docker run --rm hello
```

此示例创建教程中使用的 hello-world 图像。如果你想测试它，你可以克隆 [the image repo](https://github.com/docker-library/hello-world)。

## 更多资源

有很多资源可以帮助您编写 `Dockerfile`。

* 参考部分中的 a中提供了所有可用说明的完整指南Dockerfile。
* 为了帮助您编写清晰、可读、可维护的Dockerfile，我们还编写了Dockerfile最佳实践指南。
* 如果您的目标是创建一个新的 Docker 官方镜像，请阅读Docker 官方镜像。

* There's a [complete guide to all the instructions](../../engine/reference/builder.md) available for use in a `Dockerfile` in the reference section.
* To help you write a clear, readable, maintainable `Dockerfile`, we've also
written a [`Dockerfile` best practices guide](dockerfile_best-practices.md).
* If your goal is to create a new Docker Official Image, read [Docker Official Images](../../docker-hub/official_images.md).
