---
title: "多容器应用"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Using more than one container in our application
---

到目前为止，我们一直在使用单容器应用程序。但是，我们现在想要将 MySQL 添加到应用程序堆栈中。经常会出现以下问题——“MySQL 将在哪里运行？安装在同一个容器中还是单独运行？” 一般来说，每个容器应该做一件事并且做好。几个原因：

- 您很有可能必须以不同于数据库的方式扩展 API 和前端
- 单独的容器让您可以隔离地版本和更新版本
- 虽然您可以在本地为数据库使用容器，但您可能希望在生产中为数据库使用托管服务。你不想在你的应用程序中发布你的数据库引擎。
- 运行多个进程会需要一个进程管理器（容器只启动一个进程），这增加了容器启动/关闭的复杂性

还有更多的原因。因此，我们将更新我们的应用程序以如下方式工作：

![Todo App connected to MySQL container](images/multi-app-architecture.png)
{: .text-center }

## 容器网络

请记住，默认情况下，容器是独立运行的，并且对其他进程一无所知或者同一台机器上的容器。那么，我们如何允许一个容器与另一个容器通信？答案是**网络**。现在，你不必是一名网络工程师 (万岁!)。只要记住这条规则……

> **Note**
>
> 如果两个容器在同一个网络上，它们可以相互通信。如果他们不是，他们就不能。

## 启动 MySQL

将容器放在网络上有两种方法: 1) 在开始时分配它 2) 连接现有的容器。现在，我们将首先创建网络，并在启动时附加MySQL容器。

1. 创建网络。

    ```console
    $ docker network create todo-app
    ```

2. 启动 MySQL 容器并将其连接到网络。我们还将定义一些环境变量将用于初始化数据库 (参阅 [MySQL Docker Hub listing](https://hub.docker.com/_/mysql/) “环境变量” 部分))。

    ```console
    $ docker run -d \
        --network todo-app --network-alias mysql \
        -v todo-mysql-data:/var/lib/mysql \
        -e MYSQL_ROOT_PASSWORD=secret \
        -e MYSQL_DATABASE=todos \
        mysql:5.7
    ```

    如果您使用的是PowerShell，请使用此命令。

    ```powershell
    PS> docker run -d `
        --network todo-app --network-alias mysql `
        -v todo-mysql-data:/var/lib/mysql `
        -e MYSQL_ROOT_PASSWORD=secret `
        -e MYSQL_DATABASE=todos `
        mysql:5.7
    ```

    您还将看到我们指定了 `--network-alias` 标志。稍后我们会回到这个话题。

    > **Tip**
    >
    > 您会注意到我们正在使用一个名为 `todo-mysql-data` 在这里安装 `/var/lib/mysql`，这是 MySQL 存储其数据的地方。然而，我们从未运行过 `docker volume create` 。
    > Docker 识别我们想要使用 `named volume`，并自动为我们创建一个。

3. 要确认我们已启动并运行数据库，请连接到数据库并验证它已连接。

    ```console
    $ docker exec -it <mysql-container-id> mysql -u root -p
    ```

    当密码提示出现时，输入密码。在 MySQL shell 中，列出数据库并验证您是否看到了todos数据库。

    ```console
    mysql> SHOW DATABASES;
    ```

    您应该看到如下所示的输出:

    ```plaintext
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | todos              |
    +--------------------+
    5 rows in set (0.00 sec)
    ```

    万岁! 我们有我们的 `todos` 数据库，它已经准备好供我们使用了!

## 连接到 MySQL

现在我们知道 MySQL 已启动并运行，让我们使用它！但是，问题是……如何？如果我们在同一网络上运行另一个容器，我们如何找到该容器（记住每个容器都有自己的 IP 地址）？

为了解决这个问题，我们将利用 [nicolaka/netshoot](https://github.com/nicolaka/netshoot) 容器，它附带了许多可用于故障排除或调试网络问题的工具。


1. 使用 `nicolaka/netshoot` 镜像启动一个新容器。确保将其连接到同一网络。

    ```console
    $ docker run -it --network todo-app nicolaka/netshoot
    ```

2. 在容器内部，我们将使用 `dig` 命令，这是一个有用的 DNS 工具。我们将查找主机名为 `mysql` 的 IP 地址。

    ```console
    $ dig mysql
    ```

    你会得到这样的输出...

    ```text
    ; <<>> DiG 9.14.1 <<>> mysql
    ;; global options: +cmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

    ;; QUESTION SECTION:
    ;mysql.				IN	A

    ;; ANSWER SECTION:
    mysql.			600	IN	A	172.23.0.2

    ;; Query time: 0 msec
    ;; SERVER: 127.0.0.11#53(127.0.0.11)
    ;; WHEN: Tue Oct 01 23:47:24 UTC 2019
    ;; MSG SIZE  rcvd: 44
    ```

    在“ANSWER SECTION” 中，您将看到 `mysql` 解析为的 `A`记录 `172.23.0.2`（您的 IP 地址很可能具有不同的值），而 `mysql` 通常不是有效的主机名，Docker 能够将其解析为具有该网络别名的容器的 ip 地址 (请记住之前用过的参数？`--network-alias`)。

    这意味着……我们的应用程序只需要连接到一个名为 `mysql` 的主机，它就会与数据库对话！没有比这更简单的了！

## 使用 MySQL 运行您的应用程序

todo 应用程序支持设置一些环境变量以指定 MySQL 连接设置。它们是:

- `MYSQL_HOST` - 正在运行的MySQL服务器的主机名
- `MYSQL_USER` - 用于连接的用户名
- `MYSQL_PASSWORD` - 用于连接的密码
- `MYSQL_DB` - 连接后使用的数据库

> **通过 Env Vars 设置连接设置**
>
> 虽然使用 env vars 来设置连接设置对于开发来说通常是可以的，但 在生产中运行应用程序时，这是非常不推荐的。Docker 前安全主管 Diogo Monica 写了一篇精彩的博客文章解释了原因。
> [wrote a fantastic blog post](https://diogomonica.com/2017/03/27/why-you-shouldnt-use-env-variables-for-secret-data/){:target="_blank" rel="noopener" class="_"}
>
> 一种更安全的机制是使用容器编排框架提供的 `secret` 支持。在大多数情况下，这些 `secret` 作为文件装载在正在运行的容器中。您会看到许多应用程序 (包括 MySQL 镜像和 todo 应用程序) 还支持带有 `_FILE` 后缀指向包含变量的文件。
>
> 例如，设置 `MYSQL_PASSWORD_FILE` 将导致应用程序使用引用文件的内容作为连接密码。Docker 不采取任何措施来支持这些 env vars。您的应用程序需要知道寻找变量并获取文件内容。

解释完所有这些之后，让我们启动我们的开发就绪容器！

1. **注意**: : 适用于MySQL版本8。0及更高版本，请确保在mysql。
    ```console
    mysql> ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'secret';
    mysql> flush privileges;
    ```

2. 我们将指定上面的每个环境变量，并将容器连接到我们的应用程序网络。

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

    如果您使用的是PowerShell，请使用此命令。

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

3. 如果我们查看容器的日志 (`docker logs <container-id>`)，我们应该会看到一条消息，表明它正在使用 mysql 数据库。

    ```console
    $ nodemon src/index.js
    [nodemon] 1.19.2
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching dir(s): *.*
    [nodemon] starting `node src/index.js`
    Connected to mysql db at host mysql
    Listening on port 3000
    ```

4. 在浏览器中打开应用程序，然后将一些条目添加到待办事项列表中。

5. 连接到 mysql 数据库，并证明条目正在写入数据库。记住，密码是 **secret**。

    ```console
    $ docker exec -it <mysql-container-id> mysql -p todos
    ```

    在 mysql shell 中，运行以下命令:

    ```console
    mysql> select * from todo_items;
    +--------------------------------------+--------------------+-----------+
    | id                                   | name               | completed |
    +--------------------------------------+--------------------+-----------+
    | c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
    | 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
    +--------------------------------------+--------------------+-----------+
    ```

    显然，你的表格会看起来不一样，因为它有你的条目。但是，你应该看到它们储存在那里!

如果您快速查看 Docker Dashboard，您会发现我们正在运行两个应用程序容器。但是，有没有真正的迹象表明它们被组合在一个应用程序中。我们很快就会看到如何让它变得更好!

![Docker Dashboard showing two ungrouped app containers](images/dashboard-multi-container-app.png)

## 回顾

此时，我们有一个应用程序，它现在将其数据存储在运行在单独容器中的外部数据库中。我们了解了一些关于容器网络的知识，并了解了如何使用 DNS 执行服务发现。

但是，您很有可能开始对启动此应用程序所需执行的所有操作感到有些不知所措。我们必须创建网络、启动容器、指定所有环境变量、公开端口等等！要记住很多事情，而且肯定会让事情更难传递给其他人。

在下一节中，我们将讨论 Docker Compose。使用 Docker Compose，我们可以以更简单的方式共享我们的应用程序堆栈，并让其他人使用单个（且简单的）命令来启动它们！