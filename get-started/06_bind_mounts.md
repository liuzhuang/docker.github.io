---
title: "Use bind mounts"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Using bind mounts in our application
---

在上一章中，我们讨论并使用了 **named volume** 将数据保存在我们的数据库中。
如果我们只是想存储数据，**named volume** 是很棒的，因为我们不必担心数据被存储在哪里。

使用 **bind mounts**，我们可以控制主机上的确切挂载点。我们可以用它来持久化数据，但通常是用于向容器提供额的数据。
在处理应用程序时，我们可以使用绑定挂载将我们的源代码挂载到容器中，让它看到代码更改、响应，并让我们立即看到更改。

对于基于 Node 的应用程序，[nodemon](https://npmjs.com/package/nodemon){:target="_blank" rel="noopener" class="_"} 是一个很好的工具来监视文件更改然后重新启动应用程序。大多数其他语言和框架中都有等效的工具。

## volume 类型比较

`Bind mounts` 和 `named volumes` 是 Docker 引擎附带的两种主要类型的卷。
然而，额外的卷驱动程序可用于支持其他用例([SFTP](https://github.com/vieux/docker-volume-sshfs){:target="_blank" rel="noopener" class="_"}, [Ceph](https://ceph.com/geen-categorie/getting-started-with-the-docker-rbd-volume-plugin/){:target="_blank" rel="noopener" class="_"}, [NetApp](https://netappdvp.readthedocs.io/en/stable/){:target="_blank" rel="noopener" class="_"}, [S3](https://github.com/elementar/docker-s3-volume){:target="_blank" rel="noopener" class="_"}, 以及更多)。

|   | Named Volumes | Bind Mounts |
| - | ------------- | ----------- |
| 主机位置 | Docker chooses | You control |
| 挂载示例 (using `-v`) | my-volume:/usr/local/data | /path/to/data:/usr/local/data |
| Populates new volume with container contents | Yes | No |
| 支持卷驱动程序 | Yes | No |


## 启动一个开发模式容器

要运行容器以支持开发工作流，我们将执行以下操作:

- 将我们的源代码装载到容器中
- 安装所有依赖项，包括“dev”依赖项
- 启动 nodemon 以监视文件系统更改


所以，我们开始吧!

1. 确保你以前 `getting-started` 的容器没有运行。

2. 运行以下命令。之后我们将解释发生了什么:

    ```console
    $ docker run -dp 3000:3000 \
        -w /app -v "$(pwd):/app" \
        node:12-alpine \
        sh -c "yarn install && yarn run dev"
    ```

    如果您使用的是 PowerShell，请使用以下命令:

    ```powershell
    PS> docker run -dp 3000:3000 `
        -w /app -v "$(pwd):/app" `
        node:12-alpine `
        sh -c "yarn install && yarn run dev"
    ```

    - `-dp 3000:3000` - 和以前一样。在分离 (后台) 模式下运行并创建端口映射
    - `-w /app` - 设置 “工作目录” 或命令将从中运行的当前目录
    - `-v "$(pwd):/app"` - 绑定将容器中的主机中的当前目录绑定到 `/app` 目录
    - `node:12-alpine` - 要使用的镜像。请注意，这是 Dockerfile 中我们应用程序的基本镜像
    - `sh -c "yarn install && yarn run dev"` - the command. We're starting a shell using `sh` (alpine doesn't have `bash`) and
      running `yarn install` to install _all_ dependencies and then running `yarn run dev`. If we look in the `package.json`,
      we'll see that the `dev` script is starting `nodemon`.

3. 您可以使用`docker logs -f <container-id>`。当你看到这个的时候，你就会知道你已经准备好了: 

    ```console
    $ docker logs -f <container-id>
    nodemon src/index.js
    [nodemon] 1.19.2
    [nodemon] to restart at any time, enter `rs`
    [nodemon] watching dir(s): *.*
    [nodemon] starting `node src/index.js`
    Using sqlite database at /etc/todos/todo.db
    Listening on port 3000
    ```

    看完日志后，点击退出`Ctrl`+`C`。

4. 现在，让我们对应用程序进行更改。在 `src/static/js/app.js` 文件，让我们将 “Add Item” 按钮改为简单地说 “Add”。此更改将在109行:

    ```diff
    -                         {submitting ? 'Adding...' : 'Add Item'}
    +                         {submitting ? 'Adding...' : 'Add'}
    ```

5. 只需刷新页面 (或打开页面)，您就可以立即看到浏览器中反映的更改。Node 服务器可能需要几秒钟才能重新启动，因此如果出现错误，请在几秒钟后尝试刷新。

    ![Screenshot of updated label for Add button](images/updated-add-button.png){: style="width:75%;"}
    {: .text-center }

6. 随时进行您想进行的任何其他更改。完成后，停止容器并构建新映像使用 `docker build -t getting-started .`。

使用 `bind mounts` 是本地开发上非常常见。优点是开发机器不需要安装的所有构建工具和环境。
只需一个 `docker run` 命令，即可拉取开发环境并准备就绪。我们将在以后的步骤中讨论 Docker Compose，因为这将有助于简化我们的命令（我们已经获得了很多标志）。

## 回顾

在这一点上，我们可以保留我们的数据库，并迅速响应投资者和创始人的需求。万岁!但是，你猜怎么着？我们收到了好消息!

**您的项目已被选为未来的开发!** 

为了准备生产，我们需要将我们的数据库从 SQLite 迁移到可以更好扩展的东西。为简单起见，我们将保留关系数据库并将我们的应用程序切换为使用 MySQL。但是，我们应该如何运行 MySQL？我们如何让容器相互通信？我们接下来会谈到这个！