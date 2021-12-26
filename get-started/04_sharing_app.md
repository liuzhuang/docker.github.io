---
title: "分享应用"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop, docker hub, sharing 
redirect_from:
- /get-started/part3/
description: Sharing our image we built for our example application so we can run it else where and other developers can use it
---

现在我们已经建立了一个镜像，让我们分享它！要共享 Docker 镜像，您必须使用 Docker registry。默认 registry 是 Docker Hub，是我们使用的所有镜像的来源。

> **Docker ID**
>
> Docker ID 允许您访问 Docker Hub，这是世界上最大的容器镜像库和社区。
> 如果你没有，请免费创建一个 [Docker ID](https://hub.docker.com/signup){:target="_blank" rel="noopener" class="_"}。

## 创建一个仓库

要推送镜像，我们首先需要在 Docker Hub 上创建存储库。

1. [注册或登录](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade){:target="_blank" rel="noopener" class="_"} 到 [Docker Hub](https://hub.docker.com){:target="_blank" rel="noopener" class="_"}。

2. 点击 **Create Repository** 按钮。

3. repo name 使用 `getting-started`，确保可见性是 `Public`。

    > **私有仓库**
    >
    > 您是否知道 Docker 提供私有存储库，允许您将内容限制为特定用户或团队？
    > 查看 [Docker pricing](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade){:target="_blank" rel="noopener" class="_"} 页面。

4. 点击**Create**按钮!

如果你看页面的右侧，你会看到一个名为 **Docker commands** 的部分。这提供了一个示例命令，您需要运行该命令才能推送到此存储库。

![Docker command with push example](images/push-command.png){: style=width:75% }
{: .text-center }

## Push 镜像

1. 在命令行中，尝试运行您在 Docker Hub 上看到的 push 命令。请注意，您的命令将使用您的命名空间，而不是“docker”。

    ```plaintext
    $ docker push docker/getting-started
    The push refers to repository [docker.io/docker/getting-started]
    An image does not exist locally with the tag: docker/getting-started
    ```

    为什么失败？push 命令正在寻找名为 docker/get-start 的镜像，但是没找到。如果你运行 `docker image ls`，你也不会看到一个。

    为了解决这个问题，我们需要 "tag" 我们已经构建的现有镜像，给它取另一个名字。

2. 使用 `docker login -u YOUR-USER-NAME` 命令登录到 Docker Hub。

3. 使用 `docker tag` 命令给 `getting-started` 镜像指定一个新名称。一定要把 `YOUR-USER-NAME` 更换为您的 Docker ID。

    ```console
    $ docker tag getting-started YOUR-USER-NAME/getting-started
    ```

4. 现在再次尝试您的 push 命令。如果要从 Docker Hub 复制值，则可以删除 `tagname` 部分，因为我们没有在镜像名称中添加标签。如果您没有指定标签，Docker 将使用 `latest`。

    ```console
    $ docker push YOUR-USER-NAME/getting-started
    ```

## 在新实例上运行镜像

现在我们的镜像已经建立并被推送到注册表中，让我们尝试在一个品牌上运行我们的应用程序从未见过此容器镜像的新实例!为此，我们将使用Play with Docker。

1. 在浏览器上打开 [Play with Docker](https://labs.play-with-docker.com/){:target="_blank" rel="noopener" class="_"}

2. 点击 **Login** 然后从下拉列表中选择 **docker**。

3. 与您的 Docker Hub 帐户连接。

4. 登录后，点击添加新实例左侧栏上的选项。如果你没有看到它，让你的浏览器宽一点。几秒钟后，浏览器中会打开一个终端窗口。

    ![Play with Docker add new instance](images/pwd-add-new-instance.png){: style=width:75% }

5. 在终端中，启动新推送的应用程序。

    ```console
    $ docker run -dp 3000:3000 YOUR-USER-NAME/getting-started
    ```

    你应该看到镜像被下拉并最终启动!

6. 当它出现时，单击 3000 徽章，您应该会看到带有修改的应用程序!万岁! 如果 3000 徽章没有出现，你可以点击 “Open Port”” 按钮，输入 3000。

## 回顾

在本节中，我们学习了如何通过将镜像推送到注册表来共享镜像。然后我们去了一个全新的实例，能够运行新推送的镜像。这在 CI 管道中很常见，管道将在其中创建镜像并将其推送到注册表，然后是生产环境可以使用最新版本的镜像。

现在我们已经弄明白了，让我们回到最后一次最后注意到的部分。提醒一下，我们注意到当我们重新启动应用程序时，我们丢失了所有待办事项列表项。这显然不是一个很好的用户体验，所以让我们学习如何在重启!
