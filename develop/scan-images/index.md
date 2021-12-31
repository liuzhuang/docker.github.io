---
title: 扫描图像的最佳实践
description: Scan images best practices guide
keywords: docker scan, scan, images, snyk, vulnerability
---

{% include sign-up-cta.html
  body="您知道吗，您现在每月可以获得 10 次免费扫描？登录 Docker 开始扫描您的镜像查找漏洞。"
  header-text="Scan your images for free"
  target-url="https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade_scan"
%}

此页面包含有关扫描和构建安全映像的建议和最佳做法。

Docker 和 Snyk 携手合作，通过为开发人员构建和部署安全容器提供一种简单而精简的方法，将安全性融入到开发工作流程中。容器安全跨越多个团队——开发人员、安全和运营。此外，还有适用于容器的多层安全性：

- 容器镜像和运行在里面的软件
- 容器、主机操作系统和同一主机上的其他容器之间的交互
- 主机操作系统
- 容器网络和存储


在 Docker 平台中包含漏洞扫描选项扩展了现有的、熟悉的漏洞检测过程，并允许在开发过程的早期修复漏洞。简单而持续的检查过程，例如，通过使用 [Snyk Advisor](https://snyk.io/advisor/docker){:target="_blank" rel="noopener" class="_"} 在后台检查图像，可以减少检查到 Docker Hub 的漏洞。这可以导致更短的 CI 周期和更可靠的生产部署。

![Developer's security journey](/images/dev-security-journey.png){:width="700px"}

## 扫描镜像

> **Log4j 2 CVE-2021-44228**
>
> Versions of `docker scan` earlier than `v0.11.0` are not able to detect [Log4j 2
> CVE-2021-44228](https://nvd.nist.gov/vuln/detail/CVE-2021-44228){:
> target="_blank" rel="noopener" class="_"}. You must update your Docker Desktop
> installation to version 4.3.1 or higher to fix this issue. For more information,
> see [Scan images for Log4j 2 CVE](../../engine/scan#scan-images-for-log4j-2-cve). 
{: .important}

您可以通过将映像推送到 Docker Hub 来自动触发扫描。您可以通过`docker scan` CLI 中的命令或通过 Docker Hub实现此目的。

### 使用 CLI 扫描

在构建映像之后，在将映像推送到 Docker Hub 之前，运行该`docker scan`命令。有关如何使用 CLI 扫描图像的详细说明，请参阅[docker scan](../../engine/scan/index.md)。

![Docker Scan CL](/images/docker-scan-cli.png){:width="700px"}

### 使用 Docker Hub 扫描

您可以通过 Docker Hub 触发扫描、查看和检查漏洞。有关详细信息，请参阅 [Hub Vulnerability Scanning](../../docker-hub/vulnerability-scanning.md)。

> **Note**
>
> Docker Hub 漏洞扫描适用于订阅 Docker Pro、Team 或 Business 层的开发人员。有关定价计划的更多信息，请参阅 [Docker Pricing](https://www.docker.com/pricing?utm_source=docker&utm_medium=webreferral&utm_campaign=docs_driven_upgrade){:target="_blank" rel="noopener" class="_"}。

![Hub Vulnerability scanning](/images/hub-hvs.png){:width="700px"}

### 在 Docker Desktop 中查看扫描摘要

Docker 桌面在 Docker 仪表板上为您提供漏洞状态的快照。将鼠标悬停在图像上并单击**View in Hub**以查看 Docker Hub 中的详细漏洞报告。

![Hub Vulnerability scan summary](/images/hvs-scan-summary.png){:width="700px"}

## 最佳实践

作为开发人员，您可以通过几个简单的步骤来提高容器的安全性。这包括：

- 从可信赖的来源中选择正确的基础映像并保持较小
- 使用多阶段构建
- 重建图像
- 在开发过程中扫描图像
- 在生产过程中扫描图像

现在，让我们详细看看这些最佳实践中的每一个：

### 选择正确的基础镜像

实现安全映像的第一步是选择正确的基础映像。选择映像时，请确保它是从可信来源构建的，并保持较小。

Docker Hub 拥有超过 830 万个存储库。其中一些镜像是[Official Images](../../docker-hub/official_images.md)，由 Docker 发布为一组精选的 Docker 开源和嵌入式解决方案存储库。Docker 还提供由[Verified Publishers](../../docker-hub/publish/index.md) 发布的镜像。这些高质量图像由与 Docker 合作的组织发布和维护，Docker 会验证其存储库中内容的真实性。选择基本图片时，请注意**Official Image**和**Verified Publisher**徽章。

![Docker Hub Official and Verified Publisher images](/images/hub-official-images.png){:width="700px"}

从 Dockerfile 构建自己的映像时，请确保选择符合您要求的最小基础映像。较小的基础映像不仅提供可移植性和快速下载，而且还缩小了映像的大小并最大限度地减少了通过依赖项引入的漏洞数量。

我们还建议您使用两种类型的基础映像：第一个映像用于开发和单元测试，第二个映像用于在开发和生产的最新阶段进行测试。在开发的后期阶段，您的映像甚至可能不需要一些构建工具，例如编译器、构建系统或任何调试工具。具有最小依赖性的小图像可以显着降低攻击面。

### 使用多阶段构建

多阶段构建旨在创建易于阅读和维护的优化 Dockerfile。通过多阶段构建，您可以使用多个图像并有选择地仅复制特定图像所需的工件。

您可以`FROM`在 Dockerfile 中使用多个语句，并且可以为每个FROM. 您还可以有选择地将工件从一个阶段复制到另一个阶段，从而在最终图像中留下您不需要的东西。这可以产生简洁的最终图像。

这种创建微小图像的方法不仅显着降低了复杂性，而且还改变了在图像中实施易受攻击的工件。因此，多阶段构建允许您“挑选”您的工件，而不是从它们所依赖的基础镜像中继承漏洞，而不是构建在镜像上的镜像，而后者再次构建在其他镜像上。

有关如何配置多阶段构建的详细信息，请参阅[multi-stage builds](../develop-images/multistage-build.md)。

### 重建镜像

Docker 镜像是从 Dockerfile 构建的。Dockerfile 包含一组指令，可让您自动执行通常（手动）创建映像所采取的步骤。此外，它还可以包含一些导入的库并安装自定义软件。这些在 Dockerfile 中显示为指令。

构建您的图像是该图像的快照，在那个时刻。当你依赖一个没有标签的基础镜像时，你每次重建都会得到一个不同的基础镜像。此外，当您使用软件包安装程序安装软件包时，重建会彻底改变映像。例如，包含以下条目的 Dockerfile 可能会在每次重建时具有不同的二进制文件。

```dockerfile
FROM ubuntu:latest
RUN apt-get -y update && apt-get install -y python
```

我们建议您定期重建 Docker 映像，以防止已解决的已知漏洞。重建时，使用该选项--no-cache避免缓存命中并确保全新下载。

例如：

```console
$ docker build --no-cache -t myImage:myTag myPath/
```

重建映像时请考虑以下最佳实践：

- 每个容器应该只有一个职责。
- 容器应该是不可变的、轻量级的和快速的。
- 不要将数据存储在您的容器中。请改用共享数据存储。
- 容器应该易于销毁和重建。
- 使用小型基础映像（例如 Linux Alpine）。较小的图像更容易分发。
- 避免安装不必要的软件包。这使图像保持干净和安全。
- 构建时避免缓存命中。
- 在部署之前自动扫描您的映像，以避免将易受攻击的容器推向生产环境。
- 在开发和生产期间每天扫描您的图像是否存在漏洞基于此，如有必要，自动重建图像。

有关构建高效镜像的详细最佳实践和方法，请参阅 [Dockerfile best practices](../develop-images/dockerfile_best-practices.md)。

### 在开发过程中扫描镜像

从 Dockerfile 创建映像甚至重建映像都可能在您的系统中引入新的漏洞。在开发过程中扫描 Docker 映像应该是您工作流程的一部分，以便在开发早期发现漏洞。您应该在开发周期的所有阶段扫描图像，最好考虑自动扫描。例如，考虑在构建过程中、将映像推送到 Docker Hub（或任何其他注册表）之前以及最后将其推送到生产环境之前配置自动扫描。

### 在制作镜像的过程中扫描镜像

主动检查容器可以在发现新漏洞时为您省去很多麻烦，否则会使您的生产系统面临风险。

通过使用容器的 Snyk 监视器功能，可以定期扫描 Docker 映像。Snyk 创建映像依赖项的快照以进行持续监控。此外，您还应该激活运行时监控。扫描运行时内未使用的模块和包可以深入了解如何缩小图像。删除未使用的组件可防止不必要的漏洞进入系统和应用程序库。这也使图像更易于维护。


## 结论

构建安全映像是一个持续的过程。考虑本指南中强调的建议和最佳实践，以规划和构建高效、可扩展且安全的映像。

让我们回顾一下我们在本指南中学到的东西：

- 从您信任的基本映像开始。选择基本图片时，请记住官方图片和经过验证的发布商徽章。
- 保护您的代码及其依赖项。
- 选择仅包含所需软件包的最小基础映像。
- 使用多阶段构建来优化您的图像。
- 确保仔细监视和管理添加到映像中的工具和依赖项。
- 确保在开发生命周期的多个阶段扫描图像。
- 经常检查您的图像是否存在漏洞。

## 延伸阅读

您还可以查看 Snyk 的以下文章：

- [容器安全指南](https://snyk.io/learn/container-security/){:target="_blank" rel="noopener" class="_"}
- [Docker 漏洞扫描备忘单](https://goto.docker.com/rs/929-FJL-178/images/cheat-sheet-docker-desktop-vulnerability-scanning-CLI.pdf){:target="_blank" rel="noopener" class="_"}
