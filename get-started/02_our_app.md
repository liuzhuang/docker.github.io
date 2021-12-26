---
title: "示例应用"
keywords: get started, setup, orientation, quickstart, intro, concepts, containers, docker desktop
redirect_from:
- /get-started/part2/
description: overview of our simple application for learning docker
---


在本教程的其余部分，我们将使用一个在 Node.js 中运行的简单待办事项列表管理器。如果您不熟悉 Node.js，请不要担心。不需要真正的 JavaScript 经验。

此时，您的开发团队非常小，您只是在构建一个应用程序来证明您的 MVP（最小可行产品）。您想展示它的工作原理以及它的功能，而无需考虑它将如何适用于大型团队、多个开发人员等。

![Todo List Manager Screenshot](images/todo-list-sample.png){: style="width:50%;" }

## 获取应用程序

在我们可以运行应用程序之前，我们需要将应用程序源代码放到我们的机器上。对于实际项目，您通常会克隆 repo。但是对于本教程，我们创建了一个包含应用程序的 ZIP 文件。

1. [下载应用程序内容](https://github.com/docker/getting-started/tree/master/app){:target="_blank" rel="noopener" class="_"}.。您可以拉取整个项目或将其下载为 zip 文件并解压缩应用程序文件夹以开始使用

2. 提取后，使用您喜欢的代码编辑器打开项目。如果你需要一个编辑器，你可以使用 `Visual Studio Code`。你应该看见 `package.json` 和两个子目录 (`src` 和 `spec`)。

    ![Screenshot of Visual Studio Code opened with the app loaded](images/ide-screenshot.png){: style="width:650px;margin-top:20px;"}
    {: .text-center }

## 构建应用程序的容器镜像

为了构建应用程序，我们需要使用 `Dockerfile`。
Dockerfile 只是一个基于文本的指令脚本，用于创建容器镜像。
如果您以前创建过 `Dockerfile`，您可能会看出下面 `Dockerfile` 文件中的一些缺陷。但是别担心。我们会检查它们。


1. 在 `package.json` 同一个文件夹中创建一个名为 `Dockerfile` 文件，包含以下内容。 

   ```dockerfile
   # syntax=docker/dockerfile:1
   FROM node:12-alpine
   RUN apk add --no-cache python2 g++ make
   WORKDIR /app
   COPY . .
   RUN yarn install --production
   CMD ["node", "src/index.js"]
   ```

   请检查文件 `Dockerfile` 没有文件扩展名，例如 `.txt`。一些编辑器可能会自动附加此文件扩展名，这将导致下一步出错。

2. 如果您还没有这样做，请打开终端，然后转到有 `Dockerfile` 文件的 `app` 目录。现在使用 `docker build` 构建容器镜像。

   ```console
   $ docker build -t getting-started .
   ```

   此命令使用 `Dockerfile` 构建新的容器镜像。你可能注意到下载了很多 `layers` 。这是因为我们指示构建器我们想从 `node:12-alpine` 镜像开始。但是，既然我们我们的机器上没有，需要下载该镜像。

   下载镜像后，我们复制到我们的应用程序中并使用 `yarn` 安装应用程序的依赖项。`CMD` 指令指定从该镜像启动容器时要运行的默认命令。

   最后 `-t` 标志标记我们的镜像。把这个简单地想象成一个人类可读的名字最终镜像。因为我们给镜像命名为 `getting-started`，因此我们可以在运行容器时引用该镜像。

   `.` 告诉 Docker 在当前目录中查找 `Dockerfile`。

## 启动应用程序容器

现在我们有了一个镜像，让我们运行应用程序。为此，我们将使用 `docker run` (还记得之前的吗？)。

1. 使用 `docker run` 命令启动容器并指定刚刚创建的镜像的名称: 

   ```console
   $ docker run -dp 3000:3000 getting-started
   ```

   还记得 `-d` 和 `-p` 标志吗？我们在 “分离” 模式下运行新容器 (在后台) 并在主机的端口3000到容器的端口3000之间创建映射。没有端口映射，我们将无法访问应用程序。

2. 几秒钟后，打开您的 Web 浏览器[http://localhost:3000](http://localhost:3000)。您应该看看我们的应用程序。

   ![Empty Todo List](images/todo-list-empty.png){: style="width:450px;margin-top:20px;"}
   {: .text-center }

3. 继续添加一两个项目，看看它是否如您所愿。您可以将项目标记为完成并删除项目。您的前端已成功将条目存储在后端。非常快捷简单，是吧？


此时，您应该有一个运行的待办事项列表管理器，其中包含一些条目，所有条目都是由您构建的。现在，让我们做一些改变，学习如何管理我们的容器。

如果您快速查看Docker仪表板，您应该会看到两个容器现在正在运行(本教程和您新启动的应用容器)。

![Docker Dashboard with tutorial and app containers running](images/dashboard-two-containers.png)

## 回顾

在这个简短的部分中，我们学习了有关构建容器映像的基础知识，并为此创建了一个 Dockerfile。构建映像后，我们启动容器并查看正在运行的应用程序。

接下来，我们将对我们的应用程序进行修改，并学习如何使用新镜像更新我们正在运行的应用程序。在此过程中，我们将学习其他一些有用的命令。
