---
title: 使用 BuildKit 构建图像
description: Learn the new features of Docker Build with BuildKit
keywords: build, security, engine, secret, BuildKit
---

Docker Build 是 Docker Engine 最常用的功能之一——从开发人员、构建团队和发布团队的用户都使用 Docker Build。

18.09 版本的 Docker Build 增强功能引入了对构建架构的急需改革。通过集成 BuildKit，用户应该会看到性能、存储管理、特性功能和安全性方面的改进。

* 使用 BuildKit 创建的 Docker 镜像可以像使用遗留构建创建的 Docker 镜像一样推送到 Docker Hub
* 适用于旧版构建的 Dockerfile 格式也适用于 BuildKit 构建
* 新的 `--secret`命令行选项允许用户传递秘密信息以使用指定的 Dockerfile 构建新图像

有关构建选项的更多信息，请参阅[command line build options](../../engine/reference/commandline/build.md) 和 [Dockerfile reference](/engine/reference/builder/)。

## 要求

* 当前版本的 Docker（18.09 或更高版本）
* 下载自定义前端的图像所需的网络连接

## 限制

* 仅支持构建 Linux 容器

## 启用 BuildKit 构建

全新安装 docker 的最简单方法是DOCKER_BUILDKIT=1 在调用docker build命令时设置环境变量，例如：

```console
$ DOCKER_BUILDKIT=1 docker build .
```

要默认启用 docker BuildKit，请将/etc/docker/daemon.jsonfeature 中的守护程序配置设置 为 true 并重新启动守护程序：

```json
{ "features": { "buildkit": true } }
```

## 新的 Docker Build 命令行构建输出

新的 docker build BuildKit TTY 输出（默认）：

```console
$ docker build . 

[+] Building 70.9s (34/59)                                                      
 => [runc 1/4] COPY hack/dockerfile/install/install.sh ./install.sh       14.0s
 => [frozen-images 3/4] RUN /download-frozen-image-v2.sh /build  buildpa  24.9s
 => [containerd 4/5] RUN PREFIX=/build/ ./install.sh containerd           37.1s
 => [tini 2/5] COPY hack/dockerfile/install/install.sh ./install.sh        4.9s
 => [vndr 2/4] COPY hack/dockerfile/install/vndr.installer ./              1.6s
 => [dockercli 2/4] COPY hack/dockerfile/install/dockercli.installer ./    5.9s
 => [proxy 2/4] COPY hack/dockerfile/install/proxy.installer ./           15.7s
 => [tomlv 2/4] COPY hack/dockerfile/install/tomlv.installer ./           12.4s
 => [gometalinter 2/4] COPY hack/dockerfile/install/gometalinter.install  25.5s
 => [vndr 3/4] RUN PREFIX=/build/ ./install.sh vndr                       33.2s
 => [tini 3/5] COPY hack/dockerfile/install/tini.installer ./              6.1s
 => [dockercli 3/4] RUN PREFIX=/build/ ./install.sh dockercli             18.0s
 => [runc 2/4] COPY hack/dockerfile/install/runc.installer ./              2.4s
 => [tini 4/5] RUN PREFIX=/build/ ./install.sh tini                       11.6s
 => [runc 3/4] RUN PREFIX=/build/ ./install.sh runc                       23.4s
 => [tomlv 3/4] RUN PREFIX=/build/ ./install.sh tomlv                      9.7s
 => [proxy 3/4] RUN PREFIX=/build/ ./install.sh proxy                     14.6s
 => [dev 2/23] RUN useradd --create-home --gid docker unprivilegeduser     5.1s
 => [gometalinter 3/4] RUN PREFIX=/build/ ./install.sh gometalinter        9.4s
 => [dev 3/23] RUN ln -sfv /go/src/github.com/docker/docker/.bashrc ~/.ba  4.3s
 => [dev 4/23] RUN echo source /usr/share/bash-completion/bash_completion  2.5s
 => [dev 5/23] RUN ln -s /usr/local/completion/bash/docker /etc/bash_comp  2.1s
```

新的 docker build BuildKit 纯输出：

```console
$ docker build --progress=plain . 

#1 [internal] load .dockerignore
#1       digest: sha256:d0b5f1b2d994bfdacee98198b07119b61cf2442e548a41cf4cd6d0471a627414
#1         name: "[internal] load .dockerignore"
#1      started: 2018-08-31 19:07:09.246319297 +0000 UTC
#1    completed: 2018-08-31 19:07:09.246386115 +0000 UTC
#1     duration: 66.818µs
#1      started: 2018-08-31 19:07:09.246547272 +0000 UTC
#1    completed: 2018-08-31 19:07:09.260979324 +0000 UTC
#1     duration: 14.432052ms
#1 transferring context: 142B done


#2 [internal] load Dockerfile
#2       digest: sha256:2f10ef7338b6eebaf1b072752d0d936c3d38c4383476a3985824ff70398569fa
#2         name: "[internal] load Dockerfile"
#2      started: 2018-08-31 19:07:09.246331352 +0000 UTC
#2    completed: 2018-08-31 19:07:09.246386021 +0000 UTC
#2     duration: 54.669µs
#2      started: 2018-08-31 19:07:09.246720773 +0000 UTC
#2    completed: 2018-08-31 19:07:09.270231987 +0000 UTC
#2     duration: 23.511214ms
#2 transferring dockerfile: 9.26kB done
```

## 覆盖默认前端

Dockerfile如果您覆盖默认前端，则可以使用 中的新语法功能。要覆盖默认前端，请将 的第一行设置 Dockerfile为带有特定前端图像的注释：

```dockerfile
# syntax=<frontend image>, e.g. # syntax=docker/dockerfile:1.2
```

此页面上的示例使用docker/dockerfile 1.2.0 及更高版本中可用的功能。我们建议使用docker/dockerfile:1，它始终指向版本 1 语法的最新版本。BuildKit 在构建之前会自动检查语法的更新，确保您使用的是最新版本。syntax在Dockerfile 参考 中了解有关该指令的 更多信息。


## 新的 Docker Build secret 信息

The new `--secret` flag for docker build allows the user to pass secret
information to be used in the Dockerfile for building docker images in a safe
way that will not end up stored in the final image.

`id` is the identifier to pass into the `docker build --secret`. This identifier
is  associated with the `RUN --mount` identifier to use in the Dockerfile. Docker
does not use the filename of where the secret is kept outside of the Dockerfile,
since this may be sensitive information.

`dst` renames the secret file to a specific file in the Dockerfile `RUN` command
to use.

For example, with a secret piece of information stored in a text file:

```console
$ echo 'WARMACHINEROX' > mysecret.txt
```

And with a Dockerfile that specifies use of a BuildKit frontend
`docker/dockerfile:1.2`, the secret can be accessed when performing a `RUN`:

```dockerfile
# syntax=docker/dockerfile:1.2

FROM alpine

# shows secret from default secret location:
RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret

# shows secret from custom secret location:
RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar
```

The secret needs to be passed to the build using the `--secret` flag.
This Dockerfile is only to demonstrate that the secret can be accessed. As you
can see the secret printed in the build output. The final image built will not
have the secret file:

```console
$ docker build --no-cache --progress=plain --secret id=mysecret,src=mysecret.txt .
...
#8 [2/3] RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret
#8       digest: sha256:5d8cbaeb66183993700828632bfbde246cae8feded11aad40e524f54ce7438d6
#8         name: "[2/3] RUN --mount=type=secret,id=mysecret cat /run/secrets/mysecret"
#8      started: 2018-08-31 21:03:30.703550864 +0000 UTC
#8 1.081 WARMACHINEROX
#8    completed: 2018-08-31 21:03:32.051053831 +0000 UTC
#8     duration: 1.347502967s


#9 [3/3] RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar
#9       digest: sha256:6c7ebda4599ec6acb40358017e51ccb4c5471dc434573b9b7188143757459efa
#9         name: "[3/3] RUN --mount=type=secret,id=mysecret,dst=/foobar cat /foobar"
#9      started: 2018-08-31 21:03:32.052880985 +0000 UTC
#9 1.216 WARMACHINEROX
#9    completed: 2018-08-31 21:03:33.523282118 +0000 UTC
#9     duration: 1.470401133s
...
```

## 使用 SSH 访问构建中的私有数据

> **Acknowledgment**
>
> 有关更多信息和示例，请参阅[在 Docker 18.09 中构建机密和 SSH 转发](https://medium.com/@tonistiigi/build-secrets-and-ssh-forwarding-in-docker-18-09-ae8161d066)

该docker build有一个--ssh选项，允许多克尔引擎进行转发SSH代理连接。有关 SSH 代理的更多信息，请参阅 OpenSSH 手册页。

只有Dockerfile通过定义type=sshmount明确请求 SSH 访问的命令才能访问 SSH 代理连接。其他命令不知道任何可用的 SSH 代理。

要为 中的RUN命令请求 SSH 访问Dockerfile，请定义一个类型为 的挂载ssh。这将设置SSH_AUTH_SOCK环境变量以使依赖 SSH 的程序自动使用该套接字。

<!-- 
The `docker build` has a `--ssh` option to allow the Docker Engine to forward
SSH agent connections. For more information on SSH agent, see the
[OpenSSH man page](https://man.openbsd.org/ssh-agent).

Only the commands in the `Dockerfile` that have explicitly requested the SSH
access by defining `type=ssh` mount have access to SSH agent connections. The
other commands have no knowledge of any SSH agent being available.

To request SSH access for a `RUN` command in the `Dockerfile`, define a mount
with type `ssh`. This will set up the `SSH_AUTH_SOCK` environment variable to
make programs relying on SSH automatically use that socket. -->

这是在容器中使用 SSH 的 Dockerfile 示例：

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine

# Install ssh client and git
RUN apk add --no-cache openssh-client git

# Download public key for github.com
RUN mkdir -p -m 0600 ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts

# Clone private repository
RUN --mount=type=ssh git clone git@github.com:myorg/myproject.git myproject
```

一旦 `Dockerfile` 被创建，使用`--ssh` 与SSH代理连接选项。

```console
$ docker build --ssh default .
```

您可能需要先运行`ssh-add`以将私钥身份添加到身份验证代理才能使其工作。

## 故障排除：私有注册问题

#### x509：由未知机构签署的证书

如果您从不安全的注册表（使用自签名证书）获取图像和/或使用这样的注册表作为镜像，您将面临 Docker 18.09 中的一个已知问题：

```console
[+] Building 0.4s (3/3) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 169B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => ERROR resolve image config for docker.io/docker/dockerfile:experimental
------
 > resolve image config for docker.io/docker/dockerfile:experimental:
------
failed to do request: Head https://repo.mycompany.com/v2/docker/dockerfile/manifests/experimental: x509: certificate signed by unknown authority
```

解决方案：正确保护您的注册表。您可以从 Let's Encrypt 免费获取 SSL 证书。请 [Deploy a registry server](../../registry/deploying.md)。


#### 当私有注册表在 Sonatype Nexus 版本 < 3.15 上运行时找不到图像

如果您使用 Sonatype Nexus 版本 < 3.15 运行私有注册表，并收到类似于以下内容的错误：

```console
------
 > [internal] load metadata for docker.io/library/maven:3.5.3-alpine:
------
------
 > [1/4] FROM docker.io/library/maven:3.5.3-alpine:
------
rpc error: code = Unknown desc = docker.io/library/maven:3.5.3-alpine not found
```

您可能正面临以下错误：[NEXUS-12684](https://issues.sonatype.org/browse/NEXUS-12684)

解决方案是将您的 Nexus 升级到 3.15 或更高版本。
