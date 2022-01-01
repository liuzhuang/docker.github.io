<!-- This file is included in Docker Engine - Community or EE installation docs for Linux. -->

### 通过脚本安装

Docker provides a convenience script at [get.docker.com](https://get.docker.com/)
to install Docker into development environments quickly and non-interactively.
The convenience script is not recommended for production environments, but can be
used as an example to create a provisioning script that is tailored to your needs.
Also refer to the [install using the repository](#install-using-the-repository)
steps to learn about installation steps to install using the package repository.
The source code for the script is open source, and can be found in the
[`docker-install` repository on GitHub](https://github.com/docker/docker-install){:target="_blank" rel="noopener" class="_"}.

Always examine scripts downloaded from the internet before running them locally.
Before installing, make yourself familiar with potential risks and limitations
of the convenience script:
{:.warning}

- The script requires `root` or `sudo` privileges to run.
- The script attempts to detect your Linux distribution and version and
  configure your package management system for you, and does not allow you to
  customize most installation parameters.
- The script installs dependencies and recommendations without asking for
  confirmation. This may install a large number of packages, depending on the
  current configuration of your host machine.
- By default, the script installs the latest stable release of Docker, containerd,
  and runc. When using this script to provision a machine, this may result in
  unexpected major version upgrades of Docker. Always test (major) upgrades in
  a test environment before deploying to your production systems.
- The script is not designed to upgrade an existing Docker installation. When
  using the script to update an existing installation, dependencies may not be
  updated to the expected version, causing outdated versions to be used.

> Tip: preview script steps before running
>
> You can run the script with the `DRY_RUN=1` option to learn what steps the
> script will execute during installation:
>
> ```console
> $ curl -fsSL https://get.docker.com -o get-docker.sh
> $ DRY_RUN=1 sh ./get-docker.sh
> ```

This example downloads the script from [get.docker.com](https://get.docker.com/)
and runs it to install the latest stable release of Docker on Linux:

```console
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
Executing docker install script, commit: 7cae5f8b0decc17d6571f9f52eb840fbc13b2737
<...>
```

Docker is installed. The `docker` service starts automatically on Debian based
distributions. On `RPM` based distributions, such as CentOS, Fedora, RHEL or SLES,
you need to start it manually using the appropriate `systemctl` or `service` command.
As the message indicates, non-root users cannot run Docker commands by default.

> **Use Docker as a non-privileged user, or install in rootless mode?**
>
> The installation script requires `root` or `sudo` privileges to install and
> use Docker. If you want to grant non-root users access to Docker, refer to the
> [post-installation steps for Linux](/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).
> Docker can also be installed without `root` privileges, or configured to run
> in rootless mode. For instructions on running Docker in rootless mode, refer to
> [run the Docker daemon as a non-root user (rootless mode)](/engine/security/rootless/).

#### 安装预发布版本

Docker 还在 [test.docker.com](https://test.docker.com/) 上提供了一个方便的脚本来在 Linux 上安装 Docker 的预发布版本。
此脚本与 中的脚本等效`get.docker.com`，但将您的包管理器配置为从我们的包存储库启用“测试”通道，其中包括 Docker 的稳定版和预发布版（测试版、发布候选版）。使用此脚本可以提前访问新版本，并在它们稳定发布之前在测试环境中对其进行评估。

要从 "test" 频道在 Linux 上安装最新版本的 Docker，请运行：

```console
$ curl -fsSL https://test.docker.com -o test-docker.sh
$ sudo sh test-docker.sh
<...>
```

#### 使用脚本升级 Docker

如果您使用便利脚本安装 Docker，则应直接使用您的包管理器升级 Docker。重新运行便利脚本没有任何好处，如果它尝试重新添加已经添加到主机的存储库，它可能会导致问题。

If you installed Docker using the convenience script, you should upgrade Docker
using your package manager directly. There is no advantage to re-running the
convenience script, and it can cause issues if it attempts to re-add
repositories which have already been added to the host machine.
