---
title: "持久化到数据库"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Making our DB persistent in our application
---

如果您没有注意到，每次启动容器时，我们的待办事项列表都会被清除。为什么是这样？让我们深入了解容器是如何工作的。

## 容器的文件系统

当容器运行时，它会将镜像中的各个层用于其文件系统。每个容器也有自己的 “scratch space” 来创建/更新/删除文件。任何更改都不会在另一个容器中看到，即使它们使用相同的镜像。

### 在实践中看到这个

为了看到这一点，我们将启动两个容器，并在每个容器中创建一个文件。您将看到在一个容器中创建的文件在另一个容器中不可用。

1. 启动一个 `ubuntu` 容器，该容器将创建一个名为 `/data.txt`的文件，文件内容是 1 到 10000之 间的随机数。

    ```console
    $ docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
    ```

    如果您对命令感到好奇，我们将启动一个 `bash shell` 并调用两个命令 (为什么我们有 `&&`)。第一部分选择一个随机数并写入 `/data.txt` 。第二个命令只是监视文件以保持容器运行。

2. 我们可以通过 `exec` 进入容器验证。为此，打开 Dashboard 并单击第一个运行 `ubuntu` 镜像容器。

    ![Dashboard open CLI into ubuntu container](images/dashboard-open-cli-ubuntu.png){: style=width:75% }

    您将看到一个在 ubuntu 容器中运行 shell 的终端。运行以下命令以查看 `/data.txt`。之后再次关闭此终端。

    ```console
    $ cat /data.txt
    ```

    如果您喜欢命令行，可以使用 `docker exec` 命令执行相同的操作。你需要得到容器的ID (使用`docker ps` 获取) 并使用以下命令获取内容。

    ```console
    $ docker exec <container-id> cat /data.txt
    ```

    你应该看到一个随机数!

3. 现在，让我们开始另一个`ubuntu` 容器 (相同的镜像)，我们会看到我们没有相同的文件。

    ```console
    $ docker run -it ubuntu ls /
    ```

    快看! 没有 `data.txt` 文件在那里! 那是因为它被写入到暂存空间只有第一个容器。

4. 继续使用 `docker rm -f <container-id>` 命令删除第一个容器。

## 容器卷

在前面的实验中，我们看到每个容器每次启动时都是从镜像定义开始的。虽然容器可以创建、更新和删除文件，但是当容器被移除并且所有更改都与该容器隔离时，这些更改将丢失。有了卷，我们可以改变这一切。

[Volumes](../storage/volumes.md) 提供了将容器的特定文件系统路径连接回主机的能力。如果挂载了容器中的目录，则主机上也会看到该目录中的更改。如果我们在容器重新启动时挂载相同的目录，我们会看到相同的文件。

有两种主要类型的卷。我们最终将同时使用两者，但我们将从 **named volumes** 开始。

## 持久化待办事项数据

默认情况下，待办事项应用程序将其数据存储在 [SQLite Database](https://www.sqlite.org/index.html){:target="_blank" rel="noopener" class="_"}，
`/etc/todos/todo.db` 在容器的文件系统中。
如果你不熟悉SQLite，不用担心! 它只是一个关系数据库所有数据都存储在一个文件中。虽然这不是大规模应用的最佳选择，它适用于小型演示。稍后我们将讨论将其切换到其他数据库引擎。

数据库是一个单独的文件，如果我们可以将该文件保留在主机上并使其对下一个容器，它应该能够从最后一个容器停止的地方继续。通过创建卷和附加(通常称为 “挂载”) 将其发送到数据存储的目录中，我们可以保留数据。作为我们的容器写入`todo.db` 文件，它将被保存到卷中的主机。

如前所述，我们将使用 **named volume**。将 **named volume** 视为简单的数据桶。
Docker 维护磁盘上的物理位置，您只需要记住卷的名称。每次使用该卷时，Docker都会确保提供了正确的数据。


1. 使用 `docker volume create` 创建卷。

    ```console
    $ docker volume create todo-db
    ```

2. 在 Dashboard 中再次停止并删除 todo 应用程序容器 (或者 `docker rm -f <id>`)，因为它仍然在不使用持久卷的情况下仍在运行。

3. 启动 todo 应用程序容器，但添加 `-v` 用于指定卷装载的标志。我们将使用 `named volume` 和挂载到 `/etc/todos`, 它将捕获在路径上创建的所有文件。

    ```console
    $ docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
    ```

4. 容器启动后，打开应用程序并将一些条目添加到待办事项列表中。

    ![Items added to todo list](images/items-added.png){: style="width: 55%; " }
    {: .text-center }

5. 停止并删除 todo 应用程序的容器。使用D ashboard 或 `docker ps` 获取ID，然后 `docker rm -f <id>` 移除它。

6. 使用上面的相同命令启动新容器。

7. 打开应用程序。你应该会看到你的物品还在你的清单上!

8. 签出清单后，请继续并删除容器。

万岁! 您现在已经学会了如何保存数据!

>**Note**
>
> 虽然 `named volumes` 和 `bind mounts` (我们将在一分钟内讨论）是默认 Docker 引擎安装支持的两种主要卷类型，
> 但有许多卷驱动程序插件可用于支持 NFS、SFTP、NetApp 等！一旦您开始在具有 Swarm、Kubernetes 等的集群环境中的多台主机上运行容器，这将尤其重要。

## 深入理解 volume

很多人经常问 “当我使用命名卷时，Docker 实际上将我的数据存储在哪里？“ 如果你想知道，您可以使用 `docker volume inspect` 命令。

```console
$ docker volume inspect todo-db
[
    {
        "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

`Mountpoint` 是磁盘上存储数据的实际位置。请注意，在大多数机器上，你会需要具有 root 访问权限才能从主机访问此目录。但是，它就是这样!


>**直接在 Docker Desktop 上访问卷数据**
>
> 在 Docker Desktop 中运行时，Docker 命令实际上在计算机上的一个小型虚拟机中运行。
> 如果要查看 Mountpoint 目录的实际内容，则需要首先进入虚拟机的。

## 回顾

在这一点上，我们有一个运行良好的应用程序，可以在重启后存活下来! 我们可以向投资者炫耀希望他们能抓住我们的愿景!

但是，我们之前看到，为每个更改重建映像需要相当长的时间。一定有更好的做出改变的方法，对吗？有了 `bind mounts` (我们之前提到过)，还有更好的方法!现在让我们看看!
