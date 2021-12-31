---
description: Configuring and troubleshooting the Docker daemon
keywords: docker, daemon, configuration, troubleshooting
redirect_from:
- /articles/chef/
- /articles/configuring/
- /articles/dsc/
- /articles/puppet/
- /config/thirdparty/
- /config/thirdparty/ansible/
- /config/thirdparty/chef/
- /config/thirdparty/dsc/
- /config/thirdparty/puppet/
- /engine/admin/
- /engine/admin/ansible/
- /engine/admin/chef/
- /engine/admin/configuring/
- /engine/admin/dsc/
- /engine/admin/puppet/
- /engine/articles/chef/
- /engine/articles/configuring/
- /engine/articles/dsc/
- /engine/articles/puppet/
- /engine/userguide/

title:  Docker daemon 的配置和故障排除
---

成功安装并启动 Docker 后，dockerd守护进程以其默认配置运行。本主题展示了如何自定义配置、手动启动守护程序以及在遇到问题时对守护程序进行故障排除和调试。

## 使用操作系统实用程序启动守护进程

在典型安装中，Docker 守护程序由系统实用程序启动，而不是由用户手动启动。这使得在机器重新启动时更容易自动启动 Docker。

启动 Docker 的命令取决于您的操作系统。检查 [Install Docker](../../engine/install/index.md)下的正确页面。要将 Docker 配置为在系统启动时自动启动，请参阅[Configure Docker to start on boot](../../engine/install/linux-postinstall.md#configure-docker-to-start-on-boot)。

## 手动启动守护进程

如果您不想使用系统实用程序来管理 Docker 守护程序，或者只想测试一下，您可以使用dockerd 命令手动运行它。您可能需要使用sudo，具体取决于您的操作系统配置。

当您以这种方式启动 Docker 时，它会在前台运行并将其日志直接发送到您的终端。

```console
$ dockerd

INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(unix:///var/run/docker.sock)
INFO[0000] Listening for HTTP on unix (/var/run/docker.sock)
```

要在手动启动时停止 Docker，请Ctrl+C在终端中发出 a 。

## 配置 Docker 守护进程

有两种方法可以配置 Docker 守护进程：

* 使用 JSON 配置文件。这是首选选项，因为它将所有配置保存在一个地方。
* 启动时使用标志dockerd。

您可以同时使用这两个选项，只要您不在标志和 JSON 文件中指定相同的选项即可。如果发生这种情况，Docker 守护程序将不会启动并打印错误消息。

要使用 JSON 文件配置 Docker 守护程序，请/etc/docker/daemon.json在 Linux 系统或C:\ProgramData\docker\config\daemon.json Windows上创建一个文件 。在 MacOS 上，转到任务栏 > 首选项 > 守护程序 > 高级中的鲸鱼。

这是配置文件的样子：

```json
{
  "debug": true,
  "tls": true,
  "tlscert": "/var/docker/server.pem",
  "tlskey": "/var/docker/serverkey.pem",
  "hosts": ["tcp://192.168.59.3:2376"]
}
```

使用此配置，Docker 守护程序以调试模式运行，使用 TLS，并侦听路由到192.168.59.3端口的流量2376。您可以在dockerd 参考文档中了解可用的配置选项

您还可以手动启动 Docker 守护程序并使用标志对其进行配置。这对于解决问题很有用。

以下是如何使用与上述相同的配置手动启动 Docker 守护程序的示例：


```console
$ dockerd --debug \
  --tls=true \
  --tlscert=/var/docker/server.pem \
  --tlskey=/var/docker/serverkey.pem \
  --host tcp://192.168.59.3:2376
```

您可以在[dockerd reference docs](../../engine/reference/commandline/dockerd.md)文档中了解哪些配置选项可用 ，或者通过运行：

```console
$ dockerd --help
```

Docker 文档中讨论了许多特定的配置选项。接下来要去的一些地方包括：

- [自动启动容器](../containers/start-containers-automatically.md)
- [限制容器的资源](../containers/resource_constraints.md)
- [配置存储驱动程序](../../storage/storagedriver/select-storage-driver.md)
- [容器安全](../../engine/security/index.md)

## Docker 守护进程目录

Docker 守护进程将所有数据保存在一个目录中。这会跟踪与 Docker 相关的所有内容，包括容器、图像、卷、服务定义和机密。

默认情况下，此目录为：

* `/var/lib/docker` on Linux.
* `C:\ProgramData\docker` on Windows.

您可以使用data-root配置选项将 Docker 守护程序配置为使用不同的目录 。

由于 Docker 守护程序的状态保存在此目录中，因此请确保为每个守护程序使用专用目录。如果两个守护程序共享同一个目录，例如 NFS 共享，您将遇到难以解决的错误。

## 对守护进程进行故障排除

您可以在守护程序上启用调试以了解守护程序的运行时活动并帮助进行故障排除。如果守护进程完全没有响应，您还 可以通过向Docker 守护进程发送信号来强制将所有线程的完整堆栈跟踪添加到守护进程日志中SIGUSR。

### 排查daemon.json和启动脚本之间的冲突

如果您使用daemon.json文件并dockerd 手动或使用启动脚本将选项传递给命令，并且这些选项发生冲突，Docker 将无法启动并显示错误，例如：

```none
unable to configure the Docker daemon with file /etc/docker/daemon.json:
the following directives are specified both as a flag and in the configuration
file: hosts: (from flag: [unix:///var/run/docker.sock], from file: [tcp://127.0.0.1:2376])
```

如果您看到与此类似的错误并且您正在使用标志手动启动守护程序，则可能需要调整标志或daemon.json以消除冲突。

> **Note**: 
> 如果您看到此特定错误，请继续阅读 下一部分以获取解决方法。

如果您使用操作系统的 init 脚本启动 Docker，您可能需要以特定于操作系统的方式覆盖这些脚本中的默认值。

#### 将 daemon.json 中的 hosts 密钥与 systemd 一起使用

一个难以解决的配置冲突的值得注意的例子是当您想要指定一个与默认值不同的守护程序地址时。默认情况下，Docker 侦听套接字。在使用 的 Debian 和 Ubuntu 系统上systemd，这意味着-H在启动时始终使用主机标志dockerd。如果您在 中指定一个 hosts条目daemon.json，这会导致配置冲突（如上面的消息所示）并且 Docker 无法启动。

要解决此问题，请创建一个/etc/systemd/system/docker.service.d/docker.conf包含以下内容的新文件，以删除-H默认情况下启动守护程序时使用的参数。

```none
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

有时您可能需要systemd使用 Docker进行配置，例如 [configuring a HTTP or HTTPS proxy](systemd.md#httphttps-proxy)。

> **Note**:
> 如果您覆盖此选项，然后hosts在手动启动 Docker 时未在daemon.json 或-H标志中指定条目，则 Docker 将无法启动。

sudo systemctl daemon-reload在尝试启动 Docker 之前运行。如果 Docker 成功启动，它现在正在侦听hosts键中 指定的 IP 地址daemon.json而不是套接字。

> **Important**:
> Windows 版 Docker 桌面或 Mac 版 Docker 桌面不支持hosts中的设置daemon.json。

{:.important}



### 内存不足异常 (OOME)

如果您的容器尝试使用比系统可用的内存更多的内存，您可能会遇到内存不足异常 (OOME)，并且容器或 Docker 守护进程可能会被内核 OOM 杀手杀死。为防止发生这种情况，请确保您的应用程序在具有足够内存的主机上运行，​​并参阅了解内存不足 的风险。


### 阅读日志

守护进程日志可以帮助您诊断问题。日志可以保存在几个位置之一，具体取决于操作系统配置和使用的日志子系统：

| Operating system                    | Location                                                                                                                                 |
|:------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------|
| Linux                               | Use the command `journalctl -xu docker.service` (or read `/var/log/syslog` or `/var/log/messages`, depending on your Linux Distribution) |
| macOS (`dockerd` logs)              | `~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log`                                                                         |
| macOS (`containerd` logs)           | `~/Library/Containers/com.docker.docker/Data/log/vm/containerd.log`                                                                      |
| Windows (WSL2) (`dockerd` logs)     | `AppData\Roaming\Docker\log\vm\dockerd.log`                                                                                              |
| Windows (WSL2) (`containerd` logs)  | `AppData\Roaming\Docker\log\vm\containerd.log`                                                                                           |
| Windows (Windows containers)        | Logs are in the Windows Event Log                                                                                                        |

要dockerd在 macOS 上查看日志，请打开一个终端窗口，然后使用tail 带有-f标志的命令来“跟踪”日志。将打印日志，直到您使用CTRL+c以下命令终止命令：

```console
$ tail -f ~/Library/Containers/com.docker.docker/Data/log/vm/dockerd.log
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.497642089Z" level=debug msg="attach: stdout: begin"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.497714291Z" level=debug msg="attach: stderr: begin"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.499798390Z" level=debug msg="Calling POST /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/wait?condition=removed"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.518403686Z" level=debug msg="Calling GET /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/json"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.527074928Z" level=debug msg="Calling POST /v1.41/containers/35fc5ec0ffe1ad492d0a4fbf51fd6286a087b89d4dd66367fa3b7aec70b46a40/start"
2021-07-28T10:21:21Z dockerd time="2021-07-28T10:21:21.528203579Z" level=debug msg="container mounted via layerStore: &{/var/lib/docker/overlay2/6e76ffecede030507fcaa576404e141e5f87fc4d7e1760e9ce5b52acb24
...
^C
```


### Enable debugging

有两种方法可以启用调试。推荐的方法是在文件中设置 debug密钥。此方法适用于每个 Docker 平台。truedaemon.json


1.  编辑daemon.json文件，该文件通常位于/etc/docker/. 如果此文件尚不存在，您可能需要创建它。在 macOS 或 Windows 上，请勿直接编辑文件。相反，转到 Preferences / Daemon / Advanced。

2.  如果文件为空，请添加以下内容：

    ```json
    {
      "debug": true
    }
    ```

    如果文件已经包含 JSON，只需添加 key "debug": true，如果它不是右括号前的最后一行，请小心在行尾添加逗号。还要验证是否log-level设置了密钥，是否将其设置为info或debug。info是默认值，可能的值为debug、info、warn、error、fatal。

3.  HUP向守护进程发送信号以使其重新加载其配置。在 Linux 主机上，使用以下命令。

    ```console
    $ sudo kill -SIGHUP $(pidof dockerd)
    ```

    在 Windows 主机上，重新启动 Docker。

您还可以停止 Docker 守护程序并使用调试标志手动重新启动它，而不是遵循此过程-D。但是，这可能会导致 Docker 在与主机启动脚本创建的环境不同的环境下重新启动，这可能会使调试更加困难。

### 强制记录堆栈跟踪

如果守护程序没有响应，您可以通过向SIGUSR1守护程序发送信号来强制记录完整的堆栈跟踪。

- **Linux**:

  ```console
  $ sudo kill -SIGUSR1 $(pidof dockerd)
  ```

- **Windows Server**:

  Download [docker-signal](https://github.com/moby/docker-signal).
  
  Get the process ID of dockerd `Get-Process dockerd`.

  Run the executable with the flag `--pid=<PID of daemon>`.


这会强制记录堆栈跟踪，但不会停止守护程序。守护进程日志显示堆栈跟踪或包含堆栈跟踪的文件的路径（如果它已记录到文件中）。

守护进程在处理SIGUSR1信号并将堆栈跟踪转储到日志后继续运行。堆栈跟踪可用于确定守护进程中所有 goroutine 和线程的状态。


### 查看堆栈跟踪

可以使用以下方法之一查看 Docker 守护进程日志：

- 通过journalctl -u docker.service在 Linux 系统上运行，使用systemctl
- /var/log/messages, /var/log/daemon.log, 或/var/log/docker.log在较旧的 Linux 系统上


> **Note**
> 
> 无法在 Docker Desktop for Mac 或 Docker Desktop for Windows 上手动生成堆栈跟踪。但是，如果遇到问题，您可以单击 Docker 任务栏图标并选择疑难解答以将信息发送到 Docker。

在 Docker 日志中查看类似如下的消息：

```none
...goroutine stacks written to /var/run/docker/goroutine-stacks-2017-06-02T193336z.log
...daemon datastructure dump written to /var/run/docker/daemon-data-2017-06-02T193336z.log
```

Docker 保存这些堆栈跟踪和转储的位置取决于您的操作系统和配置。有时，您可以直接从堆栈跟踪和转储中获得有用的诊断信息。否则，您可以将此信息提供给 Docker 以帮助诊断问题。

## 检查 Docker 是否正在运行

检查 Docker 是否正在运行的独立于操作系统的方法是使用docker info命令询问 Docker 。

您还可以使用操作系统实用程序，例如 sudo systemctl is-active docker或sudo status docker或 sudo service docker status，或使用 Windows 实用程序检查服务状态。

最后，您可以dockerd使用诸如ps或 之类的命令检查进程的进程列表top。