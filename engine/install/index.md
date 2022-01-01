---
title: Install Docker Engine
description: Lists the installation methods
keywords: docker, installation, install, Docker Engine, Docker Engine, docker editions, stable, edge
redirect_from:
- /cs-engine/
- /cs-engine/1.12/
- /cs-engine/1.12/upgrade/
- /cs-engine/1.13/
- /cs-engine/1.13/upgrade/
- /ee/docker-ee/oracle/
- /ee/supported-platforms/
- /en/latest/installation/
- /engine/installation/
- /engine/installation/frugalware/
- /engine/installation/linux/
- /engine/installation/linux/archlinux/
- /engine/installation/linux/cruxlinux/
- /engine/installation/linux/docker-ce/
- /engine/installation/linux/docker-ee/
- /engine/installation/linux/docker-ee/oracle/
- /engine/installation/linux/frugalware/
- /engine/installation/linux/gentoolinux/
- /engine/installation/linux/oracle/
- /engine/installation/linux/other/
- /engine/installation/oracle/
- /enterprise/supported-platforms/
- /install/linux/docker-ee/oracle/
---

> **适用于 Linux 的 Docker Desktop**
>
> Docker Desktop 可帮助您在 Mac 和 Windows 上轻松构建、共享和运行容器，就像在 Linux 上一样。
> Docker 处理复杂的设置并允许您专注于编写代码。
> 由于我们收到了 [subscription updates](https://www.docker.com/blog/updating-product-subscriptions/){: target="_blank" rel="noopener" class="_" id="dkr_docs_cta"} 的积极支持，我们已经开始开发 [Docker Desktop for Linux](https://www.docker.com/blog/accelerating-new-features-in-docker-desktop/){: target="_blank" rel="noopener" class="_" id="dkr_docs_cta"}，这是我们公共路线图中第二受欢迎的功能请求。
> 如果您对抢先体验感兴趣，请注册我们的 [Developer Preview program](https://www.docker.com/community/get-involved/developer-preview){: target="_blank" rel="noopener" class="_" id="dkr_docs_cta"} 计划。
>
{: .important}

## 支持的平台

Docker Engine 可 通过 Docker Desktop 在各种 [Linux platforms](#server),
[macOS](../../desktop/mac/install.md)、 [macOS](../../desktop/mac/install.md) 和 [Windows 10](../../desktop/windows/install.md) 上使用，并作为静态二进制安装。
在下面找到您喜欢的操作系统。

### 桌面操作系统

{% assign yes = '![yes](/images/green-check.svg){: .inline style="height: 14px; margin: 0 auto"}' %}

| Platform  | x86_64 / amd64 | arm64 (Apple Silicon) |
|:---------|:--------:|:---------:|
| [Docker Desktop for Mac (macOS)](../../desktop/mac/install.md)  | [{{ yes }}](../../desktop/mac/install.md)        | [{{ yes }}](../../desktop/mac/install.md)     |
| [Docker Desktop for Windows](../../desktop/windows/install.md) | [{{ yes }}](../../desktop/windows/install.md) |                                                  |

### 服务器

Docker 为下面的 Linux 发行版和硬件架构提供了 `.deb` 和 `.rpm` 包。

| Platform  | x86_64 / amd64 | arm64 / aarch64 | arm (32-bit) | s390x |
|:--------|:------|:----------|:------------|:-----------|
| [CentOS](centos.md)     | [{{ yes }}](centos.md) | [{{ yes }}](centos.md) |                        |                        |
| [Debian](debian.md)     | [{{ yes }}](debian.md) | [{{ yes }}](debian.md) | [{{ yes }}](debian.md) |                        |
| [Fedora](fedora.md)     | [{{ yes }}](fedora.md) | [{{ yes }}](fedora.md) |                        |                        |
| [Raspbian](debian.md)   |                        |                        | [{{ yes }}](debian.md) |                        |
| [RHEL](rhel.md)         |                        |                        |                        | [{{ yes }}](rhel.md)   |
| [SLES](sles.md)         |                        |                        |                        | [{{ yes }}](sles.md)   |
| [Ubuntu](ubuntu.md)     | [{{ yes }}](ubuntu.md) | [{{ yes }}](ubuntu.md) | [{{ yes }}](ubuntu.md) | [{{ yes }}](ubuntu.md) |
| [Binaries](binaries.md) | [{{yes}}](binaries.md) | [{{yes}}](binaries.md) | [{{yes}}](binaries.md) |                        |

### 其他 Linux 发行版

> **Note**
>
> 虽然以下说明可能有效，但 Docker 不会测试或验证衍生产品的安装。

- Users of Debian derivatives such as "BunsenLabs Linux", "Kali Linux" or 
  "LMDE" (Debian-based Mint) should follow the installation instructions for
  [Debian](debian.md), substituting the version of their distro for the
  corresponding Debian release. Refer to the documentation of your distro to find
  which Debian release corresponds with your derivative version.
- Likewise, users of Ubuntu derivatives such as "Kubuntu", "Lubuntu" or "Xubuntu"
  should follow the installation instructions for [Ubuntu](ubuntu.md),
  substituting the version of their distro for the corresponding Ubuntu release.
  Refer to the documentation of your distro to find which Ubuntu release
  corresponds with your derivative version.
- Some Linux distributions are providing a package of Docker Engine through their
  package repositories. These packages are built and maintained by the Linux
  distribution's package maintainers and may have differences in configuration
  or built from modified source code. Docker is not involved in releasing these
  packages and bugs or issues involving these packages should be reported in
  your Linux distribution's issue tracker.

Docker 提供了用于手动安装 Docker Engine 的[二进制文件](binaries.md)。这些二进制文件是静态链接的，可以在任何 Linux 发行版上使用。

## Release channels

Docker Engine 有 3 种更新渠道 **stable**, **test**, 和 **nightly**：

* **Stable**：为您提供全面上市的最新版本。
* **Test**：提供了预发布的，可用于一般的前可用性（GA）测试。
* **Nightly**：为您提供最新的构建工作，为下一个主要版本进展。

### Stable

Year-month 发布是从与主分支不同的发布分支发布的。
分支是用 `<year>.<month>` 格式创建的，例如 `20.10`。
year-month 名称表示预计该版本将普遍可用的最早可能日历月。
所有未来的的补丁发布都是从该分支执行的。例如，一旦 `v20.10.0` 发布，所有后续补丁版本都是从 `20.10` 分支构建的。

### Test

为了准备新的 year-month 发布，当 Docker 为发布所需的里程碑已实现功能完整时，会从 master 分支创建一个具有 `YY.mm` 格式的分支。
Beta 版和候选发布版等预发布是从各自的发布分支进行的。补丁发布和相应的预发布在相应的发布分支内执行。

### Nightly

Nightly 构建为您提供下一个主要版本正在进行的最新构建。它们每天从 master 分支创建一次，版本格式为：

    0.0.0-YYYYmmddHHMMSS-abcdefabcdef


其中 time 是 UTC 中的提交时间，最后一个后缀是提交哈希的前缀，例如 `0.0.0-20180720214833-f61e0f7`.

这些构建允许从主分支上的最新代码进行测试。对于每晚构建没有任何资格或保证。

## 支持

year-month 分支的 Docker Engine 版本在下一年月的全面可用性版本发布后的一个月内根据需要提供补丁支持。

这意味着错误报告和向后移植到发布分支的评估直到生命周期结束。

在 year-month 分支达到生命周期结束后，可以从存储库中删除该分支。

### 向后移植

Docker 公司优先考虑向后移植 Docker 产品。Docker 员工或存储库维护者将努力确保合理的错误修复使其成为有效版本。

如果有重要的修复应该考虑向后移植到活动发布分支，请务必在 PR 描述中突出显示这一点，或者通过向 PR 添加评论。

### 升级路径

补丁版本始终向后兼容其 year-month  版本。

### Licensing

Docker 在 Apache 许可下获得许可，版本 2.0。有关完整的许可证文本，请参阅 [LICENSE](https://github.com/moby/moby/blob/master/LICENSE)。

## 报告安全问题

Docker 维护者非常重视安全性。如果您发现安全问题，请立即通知他们！

请不要提交公开问题；而是将您的报告私下发送至 security@docker.com。

非常感谢安全报告，Docker 将公开感谢您。

## 开始

设置完 Docker 后，您可以通过  [Docker 入门学习基础知识](../../get-started/index.md)。
