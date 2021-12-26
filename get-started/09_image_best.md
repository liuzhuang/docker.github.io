---
title: "构建镜像的最佳实践"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Tips for building the images for our application
---

## 安全扫描

构建镜像后，最好使用 `docker scan `命令扫描安全漏洞。Docker 与 [Snyk](https://snyk.io){:target="_blank" rel="noopener" class="_"} 合作提供漏洞扫描服务。


> **Note**
> 
> 您必须登录 Docker Hub 才能扫描镜像。运行命令 `docker scan --login`，然后使用 `docker scan <image-name>`。

例如，要扫描您之前在本教程中创建的镜像 `getting-started`，只需键入

```console
$ docker scan getting-started
```

扫描使用不断更新的漏洞数据库，因此您看到的输出会随着发现新漏洞而有所不同，但可能如下所示：

```plaintext
✗ Low severity vulnerability found in freetype/freetype
  Description: CVE-2020-15999
  Info: https://snyk.io/vuln/SNYK-ALPINE310-FREETYPE-1019641
  Introduced through: freetype/freetype@2.10.0-r0, gd/libgd@2.2.5-r2
  From: freetype/freetype@2.10.0-r0
  From: gd/libgd@2.2.5-r2 > freetype/freetype@2.10.0-r0
  Fixed in: 2.10.0-r1

✗ Medium severity vulnerability found in libxml2/libxml2
  Description: Out-of-bounds Read
  Info: https://snyk.io/vuln/SNYK-ALPINE310-LIBXML2-674791
  Introduced through: libxml2/libxml2@2.9.9-r3, libxslt/libxslt@1.1.33-r3, nginx-module-xslt/nginx-module-xslt@1.17.9-r1
  From: libxml2/libxml2@2.9.9-r3
  From: libxslt/libxslt@1.1.33-r3 > libxml2/libxml2@2.9.9-r3
  From: nginx-module-xslt/nginx-module-xslt@1.17.9-r1 > libxml2/libxml2@2.9.9-r3
  Fixed in: 2.9.9-r4
```

输出列出了漏洞的类型、了解更多信息的 URL，以及重要的是哪个版本的相关库修复了漏洞。

还有其他几个选项，您可以在 [docker扫描文档](../engine/scan/index.md)。

除了在命令行扫描你新建的镜像，，您还可以[configure Docker Hub](../docker-hub/vulnerability-scanning.md) 自动扫描所有新推送的镜像，然后您可以在 Docker Hub 和 Docker Desktop 中看到结果。

![Hub vulnerability scanning](images/hvs.png){: style=width:75% }
{: .text-center }

## 镜像分层

你知道你可以看看是什么构成了一个镜像吗？使用 `docker image history` 命令，您可以看到用于在镜像中创建每个层的命令。

1. 使用 `docker image history` 命令查看 `getting-started` 图层。

    ```console
    $ docker image history getting-started
    ```

    您应该得到类似这样的输出（日期/ID 可能不同）。

    ```plaintext
    IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
    a78a40cbf866        18 seconds ago      /bin/sh -c #(nop)  CMD ["node" "src/index.j…    0B                  
    f1d1808565d6        19 seconds ago      /bin/sh -c yarn install --production            85.4MB              
    a2c054d14948        36 seconds ago      /bin/sh -c #(nop) COPY dir:5dc710ad87c789593…   198kB               
    9577ae713121        37 seconds ago      /bin/sh -c #(nop) WORKDIR /app                  0B                  
    b95baba1cfdb        13 days ago         /bin/sh -c #(nop)  CMD ["node"]                 0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) COPY file:238737301d473041…   116B                
    <missing>           13 days ago         /bin/sh -c apk add --no-cache --virtual .bui…   5.35MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV YARN_VERSION=1.21.1      0B                  
    <missing>           13 days ago         /bin/sh -c addgroup -g 1000 node     && addu…   74.3MB              
    <missing>           13 days ago         /bin/sh -c #(nop)  ENV NODE_VERSION=12.14.1     0B                  
    <missing>           13 days ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B                  
    <missing>           13 days ago         /bin/sh -c #(nop) ADD file:e69d441d729412d24…   5.59MB   
    ```

    每一行代表镜像中的一个层。此处的显示在底部显示底部，在顶部显示最新层。使用它，您还可以快速查看每一层的大小，有助于诊断大镜像。

2. 您会注意到有几行被截断了。。如果您添加 `--no-trunc` 标志，你会得到完整的输出（是的......有趣的是你如何使用截断的标志来获得未截断的输出，是吧？）

    ```console
    $ docker image history --no-trunc getting-started
    ```

## 层缓存

既然您已经看到了分层的作用，那么可以学习重要的一课，来帮助减少容器映像的构建时间。

> 一旦层发生变化，所有下游层也必须重新创建

让我们再看一次我们使用的 Dockerfile...

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
```

回到镜像历史输出，我们看到 Dockerfile 中的每个命令都变成了镜像中的一个新层。您可能还记得，当我们对映像进行更改时，必须重新安装 yarn 依赖。有没有办法解决这个问题？每次构建时都传递相同的依赖关系没有多大意义，对吧？

为了解决这个问题，我们需要重构我们的 Dockerfile 以帮助支持依赖项的缓存。对于 Node 应用程序，这些依赖在 `package.json` 文件中定义。那么，如果我们首先只复制那个文件，安装依赖项，然后再复制其他所有东西呢？然后，如果package.json. 说得通？

1. 更新 Dockerfile， 首先复制 `package.json` 安装依赖项，然后复制其他所有内容。

    ```dockerfile
    # syntax=docker/dockerfile:1
    FROM node:12-alpine
    WORKDIR /app
    COPY package.json yarn.lock ./
    RUN yarn install --production
    COPY . .
    CMD ["node", "src/index.js"]
    ```

2. 在与 `Dockerfile` 相同的文件夹中，创建一个名为 `.dockerignore` 的文件，内容以下。

    ```ignore
    node_modules
    ```

    `.dockerignore` 文件是一种仅复制镜像相关文件的简单方法。你可以阅读更多关于这个的内容 [这里](../engine/reference/builder.md#dockerignore-file)。
    在这种情况下，`node_modules`第二个文件夹应省略`COPY`步骤，因为否则，它可能会覆盖由命令中创建的文件`RUN`步骤。
    有关为什么建议将其用于Node的更多详细信息。
    js应用程序和其他最佳实践，看看他们的指南 对节点进行[Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/){:target="_blank" rel="noopener" class="_"}网络应用。

3. 构建新镜像 `docker build`。

    ```console
    $ docker build -t getting-started .
    ```

    你应该看到这样的输出……

    ```plaintext
    Sending build context to Docker daemon  219.1kB
    Step 1/6 : FROM node:12-alpine
    ---> b0dc3a5e5e9e
    Step 2/6 : WORKDIR /app
    ---> Using cache
    ---> 9577ae713121
    Step 3/6 : COPY package.json yarn.lock ./
    ---> bd5306f49fc8
    Step 4/6 : RUN yarn install --production
    ---> Running in d53a06c9e4c2
    yarn install v1.17.3
    [1/4] Resolving packages...
    [2/4] Fetching packages...
    info fsevents@1.2.9: The platform "linux" is incompatible with this module.
    info "fsevents@1.2.9" is an optional dependency and failed compatibility check. Excluding it from installation.
    [3/4] Linking dependencies...
    [4/4] Building fresh packages...
    Done in 10.89s.
    Removing intermediate container d53a06c9e4c2
    ---> 4e68fbc2d704
    Step 5/6 : COPY . .
    ---> a239a11f68d8
    Step 6/6 : CMD ["node", "src/index.js"]
    ---> Running in 49999f68df8f
    Removing intermediate container 49999f68df8f
    ---> e709c03bc597
    Successfully built e709c03bc597
    Successfully tagged getting-started:latest
    ```

    您会看到所有图层都已重建。非常好，因为我们对 Dockerfile 进行了相当多的更改。

4. 现在，对 `src/static/index.html` 文件进行更改 (例如，更改 `<title>` 说 “令人敬畏的待办事项应用”)。

5. 再次构建镜像 `docker build -t getting-started .`。又来了。这一次，你的输出应该看起来有点不同。

    ```plaintext
    Sending build context to Docker daemon  219.1kB
    Step 1/6 : FROM node:12-alpine
    ---> b0dc3a5e5e9e
    Step 2/6 : WORKDIR /app
    ---> Using cache
    ---> 9577ae713121
    Step 3/6 : COPY package.json yarn.lock ./
    ---> Using cache
    ---> bd5306f49fc8
    Step 4/6 : RUN yarn install --production
    ---> Using cache
    ---> 4e68fbc2d704
    Step 5/6 : COPY . .
    ---> cccde25a3d9a
    Step 6/6 : CMD ["node", "src/index.js"]
    ---> Running in 2be75662c150
    Removing intermediate container 2be75662c150
    ---> 458e5c6f080c
    Successfully built 458e5c6f080c
    Successfully tagged getting-started:latest
    ```

    首先，您应该注意到构建速度要快得多！而且，您会看到步骤 1-4 都有 `Using cache`. 所以，万岁！我们正在使用构建缓存。推送和拉取此映像并对其进行更新也会更快。万岁！

## 多阶段构建

虽然我们不会在本教程中深入探讨它，但多阶段构建是一个非常强大的工具，可以帮助使用多个阶段来创建镜像。它们有几个优点：

- 将构建时依赖项与运行时依赖项分开
- 通过仅传送您的应用程序需要运行的内容来减少整体镜像大小

### Maven/Tomcat示例

在构建基于 Java 的应用程序时，需要 JDK 将源代码编译为 Java 字节码。但是，生产中不需要该 JDK。此外，您可能会使用 Maven 或 Gradle 等工具来帮助构建应用程序。在我们的最终镜像中也不需要这些。多阶段构建帮助。

```dockerfile
# syntax=docker/dockerfile:1
FROM maven AS build
WORKDIR /app
COPY . .
RUN mvn package

FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/webapps 
```

在此示例中，我们使用一个阶段（称为`build`）来使用 Maven 执行实际的 Java 构建。在第二阶段（`FROM tomcat`），我们从`build`阶段复制文件。最终镜像只是正在创建的最后一个阶段（可以使用`--target` 标志覆盖）。


### React 示例

在构建 React 应用程序时，我们需要一个 Node 环境来将 JS 代码（通常是 JSX）、SASS 样式表等编译成静态 HTML、JS 和 CSS。如果我们不进行服务器端渲染，我们甚至不需要用于生产构建的 Node 环境。为什么不在静态 nginx 容器中传送静态资源？

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12 AS build
WORKDIR /app
COPY package* yarn.lock ./
RUN yarn install
COPY public ./public
COPY src ./src
RUN yarn run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
```

在这里，我们使用 `node:12` 镜像来执行构建（最大化层缓存），然后将输出复制到 nginx 容器中。很酷吧？

## 回顾

通过稍微了解镜像的结构，我们可以更快地构建镜像并减少更改。扫描镜像让我们相信我们正在运行和分发的容器是安全的。多阶段构建还通过将构建时依赖项与运行时依赖项分开来帮助我们减小整体映像大小并提高最终容器的安全性。