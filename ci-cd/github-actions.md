---
description: Configure GitHub Actions
keywords: CI/CD, GitHub Actions,
title: 配置 GitHub Actions
---

本页面将指导您完成使用 Docker 设置 GitHub Action CI/CD 管道的过程。在设置新管道之前，我们建议您查看Ben 的 关于 CI/CD 最佳实践的[博客][Ben's blog](https://www.docker.com/blog/best-practices-for-using-docker-hub-for-ci-cd/){:target="_blank" rel="noopener" class="_"}。

本指南包含有关如何执行以下操作的说明：

1. 以示例Docker项目为例，配置GitHub Actions。
2. 设置 GitHub 操作工作流程。
3. 优化您的工作流程以减少构建时间。
4. 仅将特定版本推送到 Docker Hub。

## 建立一个 Docker 项目

让我们开始吧。本指南以一个简单的 Docker 项目为例。该 [SimpleWhaleDemo](https://github.com/usha-mandya/SimpleWhaleDemo){:target="_blank" rel="noopener" class="_"} 库包含Nginx的高山图像。您可以克隆此存储库，也可以使用您自己的 Docker 项目。

![SimpleWhaleDemo](images/simplewhaledemo.png){:width="500px"}

在我们开始之前，请确保您可以 从您创建的任何工作流访问Docker Hub。去做这个：

1. 将您的 Docker ID 作为秘密添加到 GitHub。导航到您的 GitHub 存储库并单击Settings > Secrets > New secret。

2. 使用名称DOCKER_HUB_USERNAME和您的 Docker ID 作为值创建一个新机密。

3. 创建一个新的个人访问令牌 (PAT)。要创建新令牌，请转到 Docker Hub 设置，然后单击 新建访问令牌。

4. 让我们将此令牌称为 simplewhaleci。

    ![New access token](images/github-access-token.png){:width="500px"}

5. 现在，将此个人访问令牌 (PAT) 作为第二个机密添加到 GitHub 机密 UI 中，名称为DOCKER_HUB_ACCESS_TOKEN。

    ![GitHub Secrets](images/github-secrets.png){:width="500px"}

## 设置 GitHub Actions 工作流程

在上一节中，我们创建了一个 PAT 并将其添加到 GitHub，以确保我们可以从任何工作流访问 Docker Hub。现在，让我们设置 GitHub Actions 工作流程，以在 Hub 中构建和存储我们的图像。

在这个例子中，让我们将推送标志设置true为我们也想推送。然后我们将添加一个标签来指定始终转到最新版本。最后，我们将回显图像摘要以查看推送的内容。

要设置工作流程：

1. 转到您在 GitHub 中的存储库，然后单击操作>新建工作流。
2. 单击自行设置工作流并添加以下内容：

首先，我们将命名此工作流：

{% raw %}
```yaml
name: ci
```
{% endraw %}

然后，我们将选择何时运行此工作流。在我们的示例中，我们将针对项目主分支的每次推送执行此操作：

{% raw %}
```yaml
on:
  push:
    branches:
      - 'main'
```
{% endraw %}

现在，我们需要指定我们在工作流程中实际想要发生的事情（哪些作业），我们将添加我们的构建版本并选择它在可用的最新 Ubuntu 实例上运行：

{% raw %}
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```
{% endraw %}

现在，我们可以添加所需的步骤：

* 第一个在 下检出我们的存储库 `$GITHUB_WORKSPACE` ，以便我们的工作流程可以访问它。
* 第二个将使用我们的 PAT 和用户名登录到 Docker Hub。
* 第三个将设置 Docker Buildx 以在后台使用 BuildKit 容器创建构建器实例。

{% raw %}
```yaml
    steps:
      -
        name: Checkout 
        uses: actions/checkout@v2
      -
        name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1
      -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest
```
{% endraw %}

现在，让工作流第一次运行，然后调整 Dockerfile 以确保 CI 正在运行并推送新的图像更改：

![CI to Docker Hub](images/ci-to-hub.png){:width="500px"}

## 优化工作流程

接下来，让我们看看如何通过使用注册表构建缓存来优化 GitHub Actions 工作流程。这可以减少构建时间，因为它不必运行不受 Dockerfile 或源代码更改影响的指令，还可以减少我们针对 Docker Hub 完成的拉取次数。

在这个例子中，我们需要向构建和推送步骤添加一些额外的属性：

{% raw %}
```yaml
      -
        name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
      -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          builder: ${{ steps.buildx.outputs.name }}
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest
          cache-from: type=registry,ref=${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:buildcache
          cache-to: type=registry,ref=${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:buildcache,mode=max
```
{% endraw %}

如您所见，我们使用type=registry缓存导出器从缓存清单或（特殊）图像配置导入/导出缓存。在这里，它将作为buildcache为我们的镜像构建命名的特定标签推送。

现在，再次运行工作流并验证它是否使用构建缓存。

## 推送标记版本并处理拉取请求

之前，我们学习了如何为 Docker 项目设置 GitHub Actions 工作流，如何通过设置缓存来优化工作流。现在让我们看看如何进一步改进它。我们可以通过添加标记版本对所有提交到 master 的行为不同的功能来做到这一点。这意味着，只推送特定版本，而不是每次提交更新 Docker Hub 上的最新版本。

您可以考虑使用这种方法将提交作为边缘标签推送，然后在夜间测试中使用它。通过这样做，您可以始终测试活动分支的最后更改，同时保留标记版本以发布到 Docker Hub。

首先，让我们修改现有的 GitHub 工作流程以考虑推送的标签和拉取请求：

{% raw %}
```yaml
on:
  push:
    branches:
      - 'main'
    tags:
      - 'v*'
```
{% endraw %}

这确保 CI 将在推送事件（分支和标签）上触发您的工作流。如果我们用类似的东西标记我们的提交v1.0.2：

{% raw %}
```console
$ git tag -a v1.0.2
$ git push origin v1.0.2
```
{% endraw %}

现在，转到 GitHub 并检查您的操作

![Push tagged version](images/push-tagged-version.png){:width="500px"}

让我们重用我们当前的工作流程来处理拉取请求以进行测试，同时将我们的图像推送到 GitHub 容器注册表中。

首先，我们必须处理拉取请求事件：


{% raw %}
```yaml
on:
  push:
    branches:
      - 'main'
    tags:
      - 'v*'
  pull_request:
    branches:
      - 'main'
```
{% endraw %}


要针对GitHub Container Registry 进行身份验证，请使用 以GITHUB_TOKEN 获得最佳安全性和体验。

现在让我们使用 GitHub Container Registry one 更改 Docker Hub 登录：

{% raw %}
```yaml
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
```
{% endraw %}

请记住更改图像的标记方式。以下示例将“最新”作为唯一标记。但是，如果您愿意，您可以为此添加任何逻辑：

{% raw %}
```yaml
  tags: ghcr.io/<username>/simplewhale:latest
```
{% endraw %}

> **Note**: Replace `<username>` with the repository owner. We could use
> {% raw %}`${{ github.repository_owner }}`{% endraw %} but this value can be mixed-case, so it could
> fail as [repository name must be lowercase](https://github.com/docker/build-push-action/blob/master/TROUBLESHOOTING.md#repository-name-must-be-lowercase){:target="_blank" rel="noopener" class="_"}.

![Update tagged images](images/ghcr-logic.png){:width="500px"}

现在，我们将有两种不同的流程：一种用于我们对 master 的更改，另一种用于我们推送的标签。接下来，我们需要修改之前的内容，以确保我们将 PR 推送到 GitHub 注册表而不是 Docker Hub。

## 结论

在本指南中，您学习了如何为现有 Docker 项目设置 GitHub Actions 工作流程，优化您的工作流程以缩短构建时间并减少拉取请求的数量，最后，我们学习了如何仅将特定版本推送到 Docker Hub。

## 下一步

您现在可以考虑设置夜间构建、在推送之前测试您的图像、设置机密、在作业之间共享图像或自动处理标签和 OCI 图像格式规范标签生成。

要了解如何执行其中一项操作，或获取有关如何设置我们今天已完成的工作的完整示例，请查看我们的高级示例 ，这些示例将引导您完成此操作和更多详细信息。
