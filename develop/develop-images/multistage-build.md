---
description: Keeping your images small with multi-stage images
keywords: images, containers, best practices, multi-stage, multistage
title: 使用多阶段构建
redirect_from:
- /engine/userguide/eng-image/multistage-build/
---

Multistage builds are useful to anyone who has struggled to optimize Dockerfiles
while keeping them easy to read and maintain.

> **致谢**:
> 特别感谢 [Alex Ellis](https://twitter.com/alexellisuk) 允许使用他的博客文章 
> [构建器模式与 Docker 中的多阶段构建](https://blog.alexellis.io/mutli-stage-docker-builds/)作为以下示例的基础。

## 在多阶段构建之前

构建图像最具挑战性的事情之一是缩小图像大小。Dockerfile 中的每条指令都为镜像添加了一层，您需要记住在继续下一层之前清除所有不需要的工件。为了编写一个真正高效的 Dockerfile，传统上您需要使用 shell 技巧和其他逻辑来保持层尽可能小，并确保每一层都有它需要的来自前一层的工件，而不是其他任何东西。

实际上，将一个 Dockerfile 用于开发（其中包含构建应用程序所需的一切）和一个用于生产的精简版 Dockerfile 是很常见的，其中仅包含您的应用程序以及运行它所需的内容。这被称为“构建器模式”。维护两个 Dockerfile 并不理想。

下面是一个遵循上述构建器模式的 `Dockerfile.build` 和 `Dockerfile` ：

**`Dockerfile.build`**:

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.16
WORKDIR /go/src/github.com/alexellis/href-counter/
COPY app.go ./
RUN go get -d -v golang.org/x/net/html \
  && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
```

请注意，此示例还`RUN`使用 Bash `&&` 运算符人为地将两个命令压缩在一起，以避免在图像中创建附加层。这很容易失败并且难以维护。例如，很容易插入另一个命令而忘记使用`\`字符继续该行。

**`Dockerfile`**:

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY app ./
CMD ["./app"]  
```

**`build.sh`**:

```bash
#!/bin/sh
echo Building alexellis2/href-counter:build

docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \  
    -t alexellis2/href-counter:build . -f Dockerfile.build

docker container create --name extract alexellis2/href-counter:build  
docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app  
docker container rm -f extract

echo Building alexellis2/href-counter:latest

docker build --no-cache -t alexellis2/href-counter:latest .
rm ./app
```

当您运行`build.sh`脚本时，它需要构建第一个映像，从中创建一个容器以复制工件，然后构建第二个映像。两个图像都占用了您系统上的空间，并且您`app` 的本地磁盘上仍然有工件。

多阶段构建大大简化了这种情况！


## 使用多阶段构建

通过多阶段构建，您可以`FROM`在 Dockerfile 中使用多个语句。每条FROM指令都可以使用不同的基础，并且每条指令都开始构建的新阶段。您可以有选择地将工件从一个阶段复制到另一个阶段，在最终图像中留下您不想要的所有内容。为了展示它是如何工作的，让我们调整上一节中的 Dockerfile 以使用多阶段构建。


**`Dockerfile`**:

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.16
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]  
```

您只需要单个 Dockerfile。您也不需要单独的构建脚本。就跑docker build。

```console
$ docker build -t alexellis2/href-counter:latest .
```

最终结果是与以前相同的微小生产图像，但复杂性显着降低。您不需要创建任何中间映像，也不需要将任何工件提取到本地系统。

它是如何工作的？第二FROM条指令以alpine:latest图像为基础开始一个新的构建阶段。该COPY --from=0行仅将上一阶段的构建工件复制到这个新阶段。Go SDK 和任何中间工件都会被留下，并且不会保存在最终图像中。

## 命名您的构建阶段

默认情况下，阶段没有命名，您可以通过它们的整数来引用它们，第一FROM条指令从 0 开始。但是，您可以通过AS <NAME>在FROM指令中添加 来命名您的阶段。此示例通过命名阶段并在COPY指令中使用名称来改进前一个示例。这意味着即使 Dockerfile 中的指令稍后重新排序，COPY也不会中断。

```dockerfile
# syntax=docker/dockerfile:1
FROM golang:1.16 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go    ./
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]  
```

## 在特定的构建阶段停止

构建镜像时，您不一定需要构建整个 Dockerfile，包括每个阶段。您可以指定目标构建阶段。以下命令假设您正在使用前一个Dockerfile但在名为 的阶段停止builder：

```console
$ docker build --target builder -t alexellis2/href-counter:latest .
```

这可能非常强大的一些场景是：

- 调试特定的构建阶段
- 使用debug启用所有调试符号或工具的production阶段和精益阶段
- 使用一个testing阶段，您的应用程序在其中填充测试数据，但使用使用真实数据的不同阶段为生产构建

## 使用外部图像作为“阶段” 

使用多阶段构建时，您不仅限于从之前在 Dockerfile 中创建的阶段进行复制。您可以使用该COPY --from指令从单独的镜像复制，使用本地镜像名称、本地或 Docker 注册表上可用的标签或标签 ID。Docker 客户端在必要时拉取映像并从那里复制工件。语法是：

```dockerfile
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
```

## 使用上一阶段作为新阶段

在使用FROM指令时，您可以通过引用它来从上一阶段停止的地方开始。例如：

```dockerfile
# syntax=docker/dockerfile:1
FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

## 版本兼容性

Docker Engine 17.05 中引入了多阶段构建语法。
