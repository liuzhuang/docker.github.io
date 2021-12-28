<!-- This text will be included in Build images topic in the Get started guides -->

### 启用 BuildKit

在我们开始构建映像之前，请确保您已在您的机器上启用 BuildKit。BuildKit 允许您高效地构建 Docker 镜像。有关更多信息，请参阅使用 [BuildKit 构建镜像]((/develop/develop-images/build_enhancements/))。

默认情况下，Docker 桌面上的所有用户都启用了 BuildKit。如果您已经安装了 Docker Desktop，则无需手动启用 BuildKit。如果您在 Linux 上运行 Docker，则可以通过使用环境变量或将 BuildKit 设为默认设置来启用 BuildKit。

要在运行 `docker build` 命令时设置 BuildKit 环境变量，请运行：

```console
$ DOCKER_BUILDKIT=1 docker build .
```

要默认启用 docker BuildKit，请将 `/etc/docker/daemon.json` 功能中的守护程序配置设置为 `true` 并重新启动守护程序。
如果该 `daemon.json` 文件不存在，请创建文件 `daemon.json`，然后将以下内容添加到该文件中。

```json
{
  "features":{"buildkit" : true}
}
```

重新启动 Docker 守护进程。