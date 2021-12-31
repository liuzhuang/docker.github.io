---
description: Collecting Docker metrics with Prometheus
keywords: prometheus, metrics
title: Collect Docker metrics with Prometheus
redirect_from:
- /engine/admin/prometheus/
- /config/thirdparty/monitoring/
- /config/thirdparty/prometheus/
---

[Prometheus](https://prometheus.io/) 是一个开源系统监控和警报工具包。您可以将 Docker 配置为 Prometheus 目标。本主题向您展示如何配置 Docker、将 Prometheus 设置为作为 Docker 容器运行，以及如何使用 Prometheus 监控您的 Docker 实例。

> **Warning**: 
> 可用指标和这些指标的名称正在积极开发中，可能随时更改。

目前，您只能监控 Docker 本身。您当前无法使用 Docker 目标监控您的应用程序。

## 配置 Docker

要将 Docker 守护程序配置为 Prometheus 目标，您需要指定 metrics-address. 执行此操作的最佳方法是通过daemon.json，默认情况下它位于以下位置之一。如果该文件不存在，请创建它。

- **Linux**: `/etc/docker/daemon.json`
- **Windows Server**: `C:\ProgramData\docker\config\daemon.json`
- **Docker Desktop for Mac / Docker Desktop for Windows**: Click the Docker icon in the toolbar,
  select **Preferences**, then select **Daemon**. Click **Advanced**.

如果文件当前为空，请粘贴以下内容：

```json
{
  "metrics-addr" : "127.0.0.1:9323",
  "experimental" : true
}
```

如果文件不为空，请添加这两个键，确保生成的文件是有效的 JSON。请注意，,除最后一行外，每一行都以逗号 ( )结尾。

保存文件，或者对于 Mac 的 Docker 桌面或 Windows 的 Docker 桌面，保存配置。重启 Docker。

Docker 现在在端口 9323 上公开与 Prometheus 兼容的指标。


## 配置并运行 Prometheus

Prometheus 在 Docker swarm 上作为 Docker 服务运行。

> **Prerequisites**
>
> 1. 一个或多个 Docker 引擎加入一个 Docker 群，docker swarm init 在一个管理器和docker swarm join其他管理器和工作节点上使用。
> 
> 2. 您需要互联网连接才能拉取 Prometheus 映像。

复制以下配置文件之一并将其保存到 /tmp/prometheus.yml（Linux 或 Mac）或C:\tmp\prometheus.yml（Windows）。这是一个库存 Prometheus 配置文件，除了在文件底部添加了 Docker 作业定义。Docker Desktop for Mac 和 Docker Desktop for Windows 需要稍微不同的配置。

<ul class="nav nav-tabs">
<li class="active"><a data-toggle="tab" data-target="#linux-config" data-group="linux">Docker for Linux</a></li>
<li><a data-toggle="tab" data-target="#mac-config" data-group="mac">Docker Desktop for Mac</a></li>
<li><a data-toggle="tab" data-target="#win-config" data-group="win">Docker Desktop for Windows</a></li>
</ul>

<div class="tab-content">
<div id="linux-config" class="tab-pane fade in active" markdown="1">

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'docker'
         # metrics_path defaults to '/metrics'
         # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9323']
```

</div><!-- linux -->
<div id="mac-config" class="tab-pane fade" markdown="1">

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['host.docker.internal:9090'] # Only works on Docker Desktop for Mac

  - job_name: 'docker'
         # metrics_path defaults to '/metrics'
         # scheme defaults to 'http'.

    static_configs:
      - targets: ['docker.for.mac.host.internal:9323']
```

</div><!-- mac -->
<div id="win-config" class="tab-pane fade" markdown="1">

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['host.docker.internal:9090'] # Only works on Docker Desktop for Windows

  - job_name: 'docker'
         # metrics_path defaults to '/metrics'
         # scheme defaults to 'http'.

    static_configs:
      - targets: ['192.168.65.1:9323']
```

</div><!-- windows -->
</div><!-- tabs -->


接下来，使用此配置启动单副本 Prometheus 服务。

<ul class="nav nav-tabs">
<li class="active"><a data-toggle="tab" data-target="#linux-run" data-group="linux">Docker for Linux</a></li>
<li><a data-toggle="tab" data-target="#mac-run" data-group="mac">Docker Desktop for Mac</a></li>
<li><a data-toggle="tab" data-target="#win-run" data-group="win">Docker Desktop for Windows or Windows Server</a></li>
</ul>

<div class="tab-content">

<div id="linux-run" class="tab-pane fade in active" markdown="1">

```console
$ docker service create --replicas 1 --name my-prometheus \
    --mount type=bind,source=/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml \
    --publish published=9090,target=9090,protocol=tcp \
    prom/prometheus
```

</div><!-- linux -->
<div id="mac-run" class="tab-pane fade" markdown="1">

```console
$ docker service create --replicas 1 --name my-prometheus \
    --mount type=bind,source=/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml \
    --publish published=9090,target=9090,protocol=tcp \
    prom/prometheus
```

</div><!-- mac -->
<div id="win-run" class="tab-pane fade" markdown="1">

```powershell
PS C:\> docker service create --replicas 1 --name my-prometheus
    --mount type=bind,source=C:/tmp/prometheus.yml,destination=/etc/prometheus/prometheus.yml
    --publish published=9090,target=9090,protocol=tcp
    prom/prometheus
```

</div><!-- windows -->
</div><!-- tabs -->

验证 Docker 目标是否在 http://localhost:9090/targets/ 中列出。

![Prometheus targets page](images/prometheus-targets.png)

如果您使用 Docker Desktop for Mac 或 Docker Desktop for Windows，则无法直接访问端点 URL。

## 使用 Prometheus

创建图表。单击Prometheus UI 中的Graphs链接。从“执行”按钮右侧的组合框中选择一个指标，然后单击“ 执行”。下面的屏幕截图显示了 engine_daemon_network_actions_seconds_count.


![Prometheus engine_daemon_network_actions_seconds_count report](images/prometheus-graph_idle.png)

上图显示了一个非常空闲的 Docker 实例。如果您正在运行活动工作负载，您的图表可能会有所不同。

为了使图表更有趣，通过启动具有 10 个任务的服务来创建一些网络操作，这些任务只是不间断地 ping Docker（您可以将 ping 目标更改为您喜欢的任何内容）：

```console
$ docker service create \
  --replicas 10 \
  --name ping_service \
  alpine ping docker.com
```

等待几分钟（默认抓取间隔为 15 秒）并重新加载您的图表。

![Prometheus engine_daemon_network_actions_seconds_count report](images/prometheus-graph_load.png)

当您准备好时，停止并删除该ping_service服务，这样您就不会无缘无故地用 ping 淹没主机。

```console
$ docker service remove ping_service
```
等待几分钟，您应该会看到图形回落到空闲级别。


## 下一步

- 阅读 [Prometheus 文档](https://prometheus.io/docs/introduction/overview/){: target="_blank" rel="noopener" class="_" }
- 设置一些 [警报](https://prometheus.io/docs/alerting/overview/){: target="_blank" rel="noopener" class="_" }
