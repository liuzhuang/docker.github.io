---
title: "使用 Docker Compose"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Making our lives easier with Compose for our application
---

[Docker Compose](../compose/index.md) 是一种工具，旨在帮助定义和共享多容器应用程序。使用 Compose，我们可以创建一个 YAML 文件来定义服务，并且使用单个命令，可以启动或删除所有内容。

使用 Compose的一大优势是您可以在文件中定义 `application stack`，将其保存在项目存储库的根目录下（现在受版本控制），并且可以轻松地让其他人为您的项目做出贡献。有人只需要克隆您的存储库并启动 compose 应用程序。事实上，你现在可能会在 GitHub/GitLab 上看到很多项目就是这样做的。

那么，我们如何开始呢？

## 安装 Docker Compose

如果您为 Windows 或 Mac 安装了 Docker Desktop/Toolbox，那么您已经拥有 Docker Compose！
Play-with-Docker 实例也已经安装了 Docker Compose。
如果您使用的是 Linux 机器，则需要 [install Docker Compose](../compose/install.md)。

安装后，您应该能够运行以下命令并查看版本信息。

```console
$ docker-compose version
```

## 创建 Compose 文件

1. 在 app 项目的根目录下，创建一个名为 `docker-compose.yml`。

2. 在撰写文件中，我们将首先定义 schema 版本。在大多数情况下，最好使用最新的受支持版本。你可以查看 [Compose file reference](../compose/compose-file/index.md) 了解当前 schema 版本和兼容性。

    ```yaml
    version: "3.7"
    ```

3. 接下来，我们将定义要作为应用程序一部分运行的服务或容器列表。

    ```yaml
    version: "3.7"

    services:
    ```

现在，我们将开始迁移一个 service 到 Compose 文件中

## 定义 app service

请记住，这是我们用来定义 app 容器的命令。

```console
$ docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

如果您使用的是 PowerShell，请使用以下命令:

```powershell
PS> docker run -dp 3000:3000 `
  -w /app -v "$(pwd):/app" `
  --network todo-app `
  -e MYSQL_HOST=mysql `
  -e MYSQL_USER=root `
  -e MYSQL_PASSWORD=secret `
  -e MYSQL_DB=todos `
  node:12-alpine `
  sh -c "yarn install && yarn run dev"
```

1. 首先，让我们定义容器的 service 条目和镜像。我们可以为 service 选择任何名称。该名称将自动成为网络别名，这将在定义我们的 MySQL 服务时很有用。

    ```yaml
    version: "3.7"

    services:
      app:
        image: node:12-alpine
    ```

2. 通常，您会看到 command 挨着镜像定义，尽管对顺序没有要求。所以，让我们继续把它移到我们的文件中。

    ```yaml
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
    ```


3. 让我们迁移 service 的端口号 `-p 3000:3000`。我们将在这里使用[短语法](../compose/compose-file/compose-file-v3.md#short-syntax-1)，但也有更详细的 [长语法](../compose/compose-file/compose-file-v3.md#long-syntax-1) 也可用。

    ```yaml
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
    ```
 
4. 接下来，我们通过使用的`working_dir`和 `volumes` 定义迁移两个工作目录 (`-w /app`) 和卷映射 (`-v "$(pwd):/app"`) 。Volumes 也有[短语法]((../compose/compose-file/compose-file-v3.md#short-syntax-3))和[长语法](../compose/compose-file/compose-file-v3.md#long-syntax-3)。

    Docker Compose 卷定义的一个优点是我们可以使用当前目录的相对路径。

    ```yaml
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
    ```

5. 最后，我们需要使用 `environment` 迁移环境变量定义。

    ```yaml
    version: "3.7"

    services:
      app:
        image: node:12-alpine
        command: sh -c "yarn install && yarn run dev"
        ports:
          - 3000:3000
        working_dir: /app
        volumes:
          - ./:/app
        environment:
          MYSQL_HOST: mysql
          MYSQL_USER: root
          MYSQL_PASSWORD: secret
          MYSQL_DB: todos
    ```

### 定义 MySQL service

现在，是时候定义 MySQL 服务了。我们用于该容器的命令如下：

```console
$ docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

如果您使用的是 PowerShell，请使用以下命令:

```powershell
PS> docker run -d `
  --network todo-app --network-alias mysql `
  -v todo-mysql-data:/var/lib/mysql `
  -e MYSQL_ROOT_PASSWORD=secret `
  -e MYSQL_DATABASE=todos `
  mysql:5.7
```

1. 我们将首先定义一个新 service 并命名它为 `mysql` 因此，它会自动获取网络别名。我们将继续并指定要使用的镜像。

    ```yaml
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
    ```

2. 接下来，我们将定义卷映射。当我们使用 运行容器时 `docker run`，会自动创建命名卷。但是，使用 Compose 运行时不会发生这种情况。我们需要在顶级 `volumes:` 部分定义卷 ，然后在服务配置中指定挂载点。通过仅提供卷名称，使用默认选项。有[更多可用选项](../compose/compose-file/compose-file-v3.md#volume-configuration-reference)不过。

    ```yaml
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
        volumes:
          - todo-mysql-data:/var/lib/mysql

    volumes:
      todo-mysql-data:
    ```

3. 最后，我们只需要指定环境变量。

    ```yaml
    version: "3.7"

    services:
      app:
        # The app service definition
      mysql:
        image: mysql:5.7
        volumes:
          - todo-mysql-data:/var/lib/mysql
        environment:
          MYSQL_ROOT_PASSWORD: secret
          MYSQL_DATABASE: todos

    volumes:
      todo-mysql-data:
    ```

此时，我们完整的 `docker-compose.yml` 应该是这样的:


```yaml
version: "3.7"

services:
  app:
    image: node:12-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes:
      - ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos

  mysql:
    image: mysql:5.7
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```

## 运行 application stack

现在我们有了我们的`docker-compose.yml`文件，我们可以启动它!

1. 首先确保没有其他 app/mysql 的副本运行 (`docker ps` 和 `docker rm -f <ids>`)。

2. 使用 `docker-compose up` 命令启动 `application stack`。我们将添加 `-d` 标志以在后台运行所有内容。

    ```console
    $ docker-compose up -d
    ```

    当我们运行它时，我们应该看到如下输出:

    ```plaintext
    Creating network "app_default" with the default driver
    Creating volume "app_todo-mysql-data" with default driver
    Creating app_app_1   ... done
    Creating app_mysql_1 ... done
    ```

    您会注意到卷和网络一样被创建! 默认情况下，Docker Compose 会自动为应用 `application stack` 创建一个网络 (这就是为什么我们没有在 compose 文件中定义网络的原因)。

3. 让我们使用 `docker-compose logs -f` 命令查看日志。您将看到来自每个服务的日志交错到一个流中。当你想观察与时间相关的问题时，这是非常有用的。该 `-f` 标记 “跟随” 日志，因此将在生成时提供实时输出。

    如果您已经运行了该命令，您将看到如下所示的输出:

    ```plaintext
    mysql_1  | 2019-10-03T03:07:16.083639Z 0 [Note] mysqld: ready for connections.
    mysql_1  | Version: '5.7.27'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
    app_1    | Connected to mysql db at host mysql
    app_1    | Listening on port 3000
    ```

    服务名称显示在行的开头 (通常是彩色的)，以帮助区分消息。如果你想查看特定服务的日志，您可以将服务名称添加到 logs 命令的末尾 (例如 `docker-compose logs -f app`)。

    > **Tip: 在启动应用程序之前等待数据库**
    >
    > 当应用程序启动时，它实际上会等待 MySQL 启动并准备好，然后再尝试连接到它。Docker 没有任何内置支持在启动另一个容器之前等待另一个容器完全启动、运行和准备就绪。
    > 对于基于 Node 的项目，您可以使用的 [wait-port](https://github.com/dwmkerr/wait-port){:target="_blank" rel="noopener" class="_"} 依赖。其他语言/框架也存在类似的项目。

4. 此时，您应该能够打开 app 并看到它正在运行。嘿!我们只剩下一个命令了!

## 在 Docker Dashboard 中查看 app stack

如果我们查看 Docker Dashboard，我们会看到有一个名为 app 的组。这是来自 Docker Compose 的 “项目名称”，用于将容器组合在一起。默认情况下，项目名称是 `docker-compose.yml` 所在目录的名称。

![Docker Dashboard with app project](images/dashboard-app-project-collapsed.png)


如果你展开 app 组，你会看到我们在 Compose 文件中定义的两个容器。名字也有点更具描述性，因为它们遵循 `<project-name>_<service-name>_<replica-number>`。所以，这很容易快速查看我们的 app 是什么容器，哪个容器是 mysql 数据库。


![Docker Dashboard with app project expanded](images/dashboard-app-project-expanded.png)

## Tear it all down

当你准备好把它全部拆掉时，只需运行 `docker-compose down` 或者点击 Docker Dashboard 上 app 组上的垃圾桶。容器将停止，网络将被移除。


>**Warning**
>
> 删除卷
>
> 默认情况下，运行时不会删除 Compose 文件中的 named volumes `docker-compose down`。如果你想删除卷，您需要添加 `--volumes` 标志。
{: .warning}

移除后，您可以切换到另一个项目，运行` docker-compose up` 并准备好为这个项目做出贡献! 真的没有比这更简单的了!


## 回顾

在本节中，我们了解了 Docker Compose 以及它如何帮助我们显着简化多服务应用程序的定义和共享。我们通过将我们使用的命令转换为适当的撰写格式来创建一个 Compose 文件。

在这一点上，我们开始结束本教程。但是，我们想介绍一些有关镜像构建的最佳实践，因为我们一直在使用的 Dockerfile 存在一个大问题。那么，让我们来看看吧！
