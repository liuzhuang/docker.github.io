---
title: "为你的应用程序配置CI/CD"
keywords: Java, CI/CD, local, development
description: Learn how to Configure CI/CD for your application
---

{% include_relative nav.html selected="5" %}

本页面将指导您完成使用 Docker 容器设置 GitHub Action CI/CD 管道的过程。在设置新管道之前，我们建议您查看关于 CI/CD 最佳实践的[博客](https://www.docker.com/blog/best-practices-for-using-docker-hub-for-ci-cd/){:target="_blank" rel="noopener" class="_"}。

本指南包含有关如何执行以下操作的说明：

1. 使用示例 Docker 项目作为示例来配置 GitHub Actions
2. 设置 GitHub 操作工作流
3. 优化您的工作流程，以减少拉取请求的数量和总构建时间
4. 仅将特定版本推送到 Docker Hub
5. 使用多阶段构建优化您的镜像

## 建立一个 Docker 项目

让我们开始吧。本指南以一个简单的 Docker 项目为例。[SimpleWhaleDemo](https://github.com/usha-mandya/SimpleWhaleDemo){:target="_blank" rel="noopener" class="_"}  仓库包含 Nginx alpine 镜像。您可以克隆这个仓库，也可以使用自己的 Docker 项目。

![SimpleWhaleDemo](../../ci-cd/images/simplewhaledemo.png){:width="500px"}


在我们开始之前，请确保您创建的任何工作流中都可以访问 [Docker Hub](https://hub.docker.com/) 。要做到这一点:

1. 将您的 Docker ID 作为 `secret` 添加到 GitHub。跳转到您的 GitHub repository 并单击 **Settings** > **Secrets** > **New secret**。

2. 创建一个名为 `DOCKER_HUB_USERNAME` 的 secret，并把 Docker ID 作为值.

3. 创建一个 Personal Access Token (PAT)，跳转到 [Docker Hub Settings](https://hub.docker.com/settings/security) 然后点击 **New Access Token**。

4. 让我们给这个令牌起名 **simplewhaleci**.

    ![New access token](../../ci-cd/images/github-access-token.png){:width="500px"}

5. 现在，将 Personal Access Token (PAT) 作为第二个 second 添加到 GitHub secrets UI，命名为 `DOCKER_HUB_ACCESS_TOKEN`。

    ![GitHub Secrets](../../ci-cd/images/github-secrets.png){:width="500px"}

## 设置 GitHub Actions 工作流

在上一节中，我们创建了一个 PAT 并将其添加到 GitHub，以确保我们可以从任何工作流中访问 Docker Hub。
现在，让我们设置 GitHub Actions 工作流程，以在 Hub 中构建和存储我们的镜像。
我们可以通过创建两个 Docker 操作来实现这一点：

1. 第一个操作使我们能够使用存储在 GitHub 存储库中的 secrets 登录到 Docker Hub。
2. 第二个是构建和推送操作。

在这个例子中，让我们将推送标志设置为 `true`。然后我们将添加一个标签来指定始终转到最新版本。最后，我们将输出镜像摘要用于查看推送的内容。

要设置工作流程：

1. 进入 GitHub 仓库，然后点击 **Actions** > **New workflow**.
2. 点击 **set up a workflow yourself** 并添加下面的内容

首先，我们讲给工作流命名：

```yaml
name: CI to Docker Hub
 ```

然后，我们将选择何时运行此工作流。在我们的示例中，我们将针对项目主分支的每次推送执行此操作：

```yaml
on:
  push:
    branches: [ main ]
```

现在，我们需要指定我们实际想要在我们的操作中发生的事情（什么作业），我们将添加我们的构建版本并选择它在最新的可用 Ubuntu 实例上运行：

```yaml
jobs:

  build:
    runs-on: ubuntu-latest
```

现在，我们可以添加所需的步骤。
第一个在 下检出我们的存储库 `$GITHUB_WORKSPACE`，以便我们的工作流程可以访问它。
第二种是使用我们的 PAT 和用户名登录到 Docker Hub。
第三个是 Builder，该动作通过一个简单的 Buildx 动作在幕后使用 BuildKit，我们也将设置它

{% raw %}
```yaml
    steps:

      - name: Check Out Repo 
        uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest

      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}
```
{% endraw %}

现在，让工作流第一次运行，然后调整 Dockerfile 以确保 CI 正在运行并推送新的镜像更改：

![CI to Docker Hub](../../ci-cd/images/ci-to-hub.png){:width="500px"}

## Optimize the workflow

接下来，让我们看看如何通过构建缓存来优化 GitHub Actions 工作流。这有两个主要优点：

1. 构建缓存减少了构建时间，因为它不必重新下载所有镜像，并且
2. 它还减少了我们针对 Docker Hub 完成的拉取次数。我们需要利用 GitHub 缓存来利用它。

让我们设置一个带有构建缓存的构建器。首先，我们需要为构建器设置缓存。在此示例中，让我们为此使用 GitHub 缓存添加路径和键以将其存储在下面。

{% raw %}
```yaml

      - name: Cache Docker layers
        uses: actions/cache@v2
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-
```
{% endraw %}

最后，在将构建器和构建缓存片段添加到 Actions 文件的顶部之后，我们需要向构建和推送步骤添加一些额外的属性。这包括：

设置构建器以使用 buildx 步骤的输出，然后使用我们之前为其设置的缓存来存储和检索

{% raw %}
```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}
      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          context: ./
          file: ./Dockerfile
          builder: ${{ steps.buildx.outputs.name }}
          push: true
          tags:  ${{ secrets.DOCKER_HUB_USERNAME }}/simplewhale:latest
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache
      - name: Image digest
        run: echo ${{ steps.docker_build.outputs.digest }}
```
{% endraw %}

现在，再次运行工作流并验证它是否使用构建缓存。

## 将标记的版本推送到 Docker Hub

之前，我们学习了如何为 Docker 项目设置 GitHub Actions 工作流，如何通过设置具有构建缓存的构建器来优化工作流。现在让我们看看如何进一步改进它。我们可以通过添加标记版本对所有提交到 master 的行为不同的功能来做到这一点。这意味着，只推送特定版本，而不是每次提交更新 Docker Hub 上的最新版本。

您可以考虑使用这种方法将提交转到本地注册表，然后在夜间测试中使用。通过这样做，您可以始终测试最新版本，同时保留标记版本以发布到 Docker Hub。

这包括两个步骤：

1. 修改 GitHub 工作流程，只将带有特定标签的提交推送到 Docker Hub
2. 设置 GitHub 操作文件以将最新提交作为镜像存储在 GitHub 注册表中

首先，让我们修改现有的 GitHub 工作流程，仅在有特定标签时才推送到 Hub。例如：

{% raw %}
```yaml
on:
  push:
    tags:
      - "v*.*.*"
```
{% endraw %}

这确保了主 CI 仅在我们使用 `V.n.n.n.` 标记我们的提交时才会触发。例如，运行以下命令：

```console
$ git tag -a v1.0.2
$ git push origin v1.0.2
```

现在，转到 GitHub 并检查您的操作

![Push tagged version](../../ci-cd/images/push-tagged-version.png){:width="500px"}

现在，让我们设置第二个 GitHub 操作文件，将我们最新的提交作为镜像存储在 GitHub 注册表中。您可能希望这样做：

1. 运行您的夜间测试或重复测试，或
2. 与同事分享正在进行的工作镜像。

让我们克隆我们之前的 GitHub 操作，并为所有推送添加回我们之前的逻辑。这意味着我们有两个工作流文件，我们之前的一个和我们现在将处理的新的。接下来，将您的 Docker Hub 登录名更改为 GitHub 容器注册表登录名：

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

要针对 [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry) 进行身份验证，请使用[`GITHUB_TOKEN`]获得最佳安全性和体验。


您可能需要在容器设置中管理存储库的 [manage write and read access of GitHub Actions](https://docs.github.com/en/packages/managing-github-packages-using-github-actions-workflows/publishing-and-installing-a-package-with-github-actions#upgrading-a-workflow-that-accesses-ghcrio)。

您还可以使用具有[适当范围]((https://docs.github.com/en/packages/getting-started-with-github-container-registry/migrating-to-github-container-registry-for-docker-images#authenticating-with-the-container-registry))的[个人访问令牌 (PAT)](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)。请记住更改镜像的标记方式。以下示例将“最新”作为唯一标记。但是，如果您愿意，您可以为此添加任何逻辑：


{% raw %}
```yaml
  tags: ghcr.io/${{ github.repository_owner }}/simplewhale:latest
```
{% endraw %}

![Update tagged images](../../ci-cd/images/ghcr-logic.png){:width="500px"}

现在，我们将有两种不同的流程：一种用于我们对 master 的更改，另一种用于我们的拉取请求。接下来，我们需要修改之前的内容，以确保我们将 PR 推送到 GitHub 注册表而不是 Docker Hub。

## 使用多阶段构建优化您的镜像

现在，让我们来看看 Dockerfile，看看我们如何优化它以在开发中工作，以及获得更小的镜像以在生产中运行容器。由于这是文件中的最后一步，当您运行 docker build 而不指定目标时，默认情况下将使用它来构建映像：

```console
$ docker build --tag java-docker .
docker build --tag java-docker .

[+] Building 1.2s (15/15) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 37B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/openjdk:11-jre-slim
 => [internal] load metadata for docker.io/library/openjdk:16-alpine3.13
 => [internal] load build context
 => => transferring context: 11.48kB
 => [production 1/2] FROM docker.io/library/openjdk:11-jre-slim@sha256:85795599f4c765182c414a1eb4e272841e18e2f267ce5010ea6a266f7f26e7f6
 => [base 1/6] FROM docker.io/library/openjdk:16-alpine3.13@sha256:49d822f4fa4deb5f9d0201ffeec9f4d113bcb4e7e49bd6bc063d3ba93aacbcae
 => CACHED [base 2/6] WORKDIR /app
 => CACHED [base 3/6] COPY .mvn/ .mvn
 => CACHED [base 4/6] COPY mvnw pom.xml ./
 => CACHED [base 5/6] RUN ./mvnw dependency:go-offline
 => CACHED [base 6/6] COPY src ./src
 => CACHED [build 1/1] RUN ./mvnw package
 => CACHED [production 2/2] COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar
 => exporting to image
 => => exporting layers
 => => writing image sha256:c17469b9e2f30537060f48bbe5d9d22003dd35edef7092348824a2438101ab3a
 => => naming to docker.io/library/java-docker
```

第二个有趣的点是，此步骤不以基本目标或 JDK 映像作为参考。相反，它使用 Java 运行时环境映像。请注意，您不需要具有所有开发依赖项的大映像来在生产中运行您的应用程序。限制生产映像中的依赖项数量可以显着限制攻击面。

```dockerfile
FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

容器还将自动公开其 8080 端口并复制在构建步骤中生成的 Java 存档以在容器启动时使用它。

以这种方式构建的生产映像仅包含一个带有最终应用程序存档的运行时环境，这正是您启动 Spring Pet Clinic 应用程序所需的。

```console
$ docker build --tag java-docker:jdk . --target development
$ docker build --tag java-docker:jre .
$  docker images
REPOSITORY       TAG        IMAGE ID         CREATED        SIZE
java-docker      jre        c17469b9e2f3     3 hours ago    270MB
java-docker      jdk        4c15436d8ab7     5 hours ago    567MB
```

## 下一步

在本单元中，您学习了如何为现有 Docker 项目设置 GitHub Actions 工作流程，优化您的工作流程以缩短构建时间并减少拉取请求的数量，最后，我们学习了如何仅将特定版本推送到 Docker Hub。您还可以针对最新的标签设置夜间测试，测试每个 PR，或者对我们正在使用的标签做一些更优雅的事情，并在我们的图像中对相同的标签使用 Git 标签。

您还可以考虑部署您的应用程序。有关详细说明，请参阅：

[Deploy your application](deploy.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.

<br />
