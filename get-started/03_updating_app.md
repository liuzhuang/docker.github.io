---
title: "更新应用程序"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
description: Making changes to our example learning application
---


作为一个小的功能要求，产品团队要求我们当我们没有任何待办事项列表项时，更改 “空文本”。他们希望将其转换为以下内容:

> 你还没有待办事项!在上面添加一个!

很简单，对吧？让我们做出改变。

## 更新源代码

1. 在 `src/static/js/app.js` 文件中, 更新第56行以使用新的空文本.

    ```diff
    -                <p className="text-center">No items yet! Add one above!</p>
    +                <p className="text-center">你还没有待办事项!在上面添加一个!</p>
    ```

2. 让我们使用之前使用的相同命令构建更新版本的镜像。

    ```console
    $ docker build -t getting-started .
    ```

3. 让我们使用更新的代码启动一个新容器。

    ```console
    $ docker run -dp 3000:3000 getting-started
    ```

**Uh oh!** 您可能看到了这样的错误 (id会有所不同):

```console
docker: Error response from daemon: driver failed programming external connectivity on endpoint laughing_burnell 
(bb242b2ca4d67eba76e79474fb36bb5125708ebdabd7f45c8eaf16caaabde9dd): Bind for 0.0.0.0:3000 failed: port is already allocated.
```

那么，发生了什么？我们无法启动新容器，因为我们的旧容器仍然正在运行。出现问题的原因是因为该容器正在使用主机的 3000 端口和机器上只有一个进程 (包括容器) 可以监听特定端口。为了解决这个问题，我们需要移除旧容器。

## 更换旧容器

要删除容器，首先需要停止它。一旦停止，它就可以被移除。我们有两种方法可以删除旧容器。随意选择你最满意的方法。

### 使用CLI删除容器

1. 使用 `docker ps`命令获取容器 ID。

    ```console
    $ docker ps
    ```

2. 使用 `docker stop` 命令停止容器。

    ```console
    # Swap out <the-container-id> with the ID from docker ps
    $ docker stop <the-container-id>
    ```

3. 容器停止后，您可以使用 `docker rm` 命令将其删除。

    ```console
    $ docker rm <the-container-id>
    ```

>**Note**
>
> 您可以通过添加 `force` 标志在单个 `docker rm` 命令中停止和删除容器。例如：`docker rm -f <the-container-id>`
>

### 使用 Docker Dashboard 删除容器

如果您打开 Docker Dashboard，只需单击两次即可删除容器！当然比查找容器ID并将其删除要容易得多。

1. 打开仪表板后，将鼠标悬停在应用程序容器上，您会看到右侧出现一组操作按钮。
2. 单击垃圾桶图标以删除容器。
3. 确认删除，你就完成了！

![Docker Dashboard - removing a container](images/dashboard-removing-container.png)

### 启动更新的应用容器

1. 现在，启动更新的应用程序。

    ```console
    $ docker run -dp 3000:3000 getting-started
    ```

2. 在 [http://localhost:3000](http://localhost:3000) 上刷新浏览器，你应该看到你更新的帮助文本!

![Updated application with updated empty text](images/todo-list-updated-empty-text.png){: style="width:55%" }
{: .text-center }

## 回顾

虽然我们能够构建更新，但您可能已经注意到了两件事:

- 我们待办事项列表中的所有现有项目都消失了！这不是一个很好的应用程序！我们很快就会谈到这一点。
- 这样一个小的变更，就有很多步骤。在接下来的部分，我们将讨论每次进行更改时，如何查看代码更新，而无需重建和启动新容器。

在讨论持久性之前，我们将很快看到如何与他人共享这些图像。