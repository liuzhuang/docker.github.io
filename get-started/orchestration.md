---
title: "编排"
keywords: orchestration, deploy, kubernetes, swarm,
description: Get oriented on some basics of Docker and install Docker Desktop.
---

容器化流程的可移植性和可再现性为跨云和数据中心移动和扩展我们的容器化应用程序提供了机会。容器有效地保证了这些应用程序在任何地方都以相同的方式运行，使我们能够快速轻松地利用所有这些环境。此外，随着我​​们扩大应用程序的规模，我们需要一些工具来帮助自动维护这些应用程序，启用自动更换故障容器，并管理这些容器在其生命周期中的更新和重新配置的推出。

管理、扩展和维护容器化应用程序的工具称为编排器，其中最常见的示例是Kubernetes和Docker Swarm。这两个编排器的开发环境部署由 Docker Desktop 提供，我们将在本指南中使用它来创建我们的第一个编排的容器化应用程序。

高级模块教您如何：

- [在您的开发机器上设置和使用 Kubernetes 环境](kube-deploy.md)
- [在您的开发机器上设置和使用 Swarm 环境](swarm-deploy.md)

## 开启 Kubernetes

Docker Desktop 将为您快速轻松地设置 Kubernetes。按照适用于您的操作系统的设置和验证说明进行操作：

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" href="#kubeosx">Mac</a></li>
  <li><a data-toggle="tab" href="#kubewin">Windows</a></li>
</ul>
<div class="tab-content">
  <div id="kubeosx" class="tab-pane fade in active">
{% capture local-content %}

### Mac

1.  安装 Docker Desktop 后，您应该会在菜单栏中看到一个 Docker 图标。单击它，然后导航到 **Preferences** > **Kubernetes**。

2.  选中**Enable Kubernetes**复选框，然后单击**Apply & Restart**。Docker Desktop 会自动为你设置 Kubernetes。当您在“首选项”菜单中的“Kubernetes running ”旁边看到绿灯时，您就会知道 Kubernetes 已成功启用。

3.  为了确认 Kubernetes 已启动并正在运行，请创建一个名为的文本文件pod.yaml，内容如下：

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: demo
    spec:
      containers:
      - name: testpod
        image: alpine:3.5
        command: ["ping", "8.8.8.8"]
    ```

    这描述了一个带有单个容器的 pod，将一个简单的 ping 隔离到 8.8.8.8。

4.  在终端中，导航到您创建的位置pod.yaml并创建您的 pod：

    ```console
    $ kubectl apply -f pod.yaml
    ```

5.  检查您的 Pod 是否已启动并正在运行：

    ```console
    $ kubectl get pods
    ```

    你应该看到类似的东西：

    ```shell
    NAME      READY     STATUS    RESTARTS   AGE
    demo      1/1       Running   0          4s
    ```

6.  检查您是否获得了 ping 过程所需的日志：

    ```console
    $ kubectl logs demo
    ```

    您应该会看到正常 ping 过程的输出：

    ```shell
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    64 bytes from 8.8.8.8: seq=0 ttl=37 time=21.393 ms
    64 bytes from 8.8.8.8: seq=1 ttl=37 time=15.320 ms
    64 bytes from 8.8.8.8: seq=2 ttl=37 time=11.111 ms
    ...
    ```

7.  最后，拆掉你的测试 pod：

    ```console
    $ kubectl delete -f pod.yaml
    ```

{% endcapture %}
{{ local-content | markdownify }}

</div>
<div id="kubewin" class="tab-pane fade" markdown="1">
{% capture localwin-content %}

### Windows

1.  安装 Docker Desktop 后，您应该会在系统托盘中看到一个 Docker 图标。右键单击它，然后导航Settings > Kubernetes。

2.  选中标记为Enable Kubernetes的复选框，然后单击Apply & Restart。Docker Desktop 会自动为你设置 Kubernetes。当您在“设置”菜单中看到“Kubernetes running ”旁边的绿灯时，您就会知道 Kubernetes 已成功启用。

3.  为了确认 Kubernetes 已启动并正在运行，请创建一个名为的文本文件pod.yaml，内容如下：

    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: demo
    spec:
      containers:
      - name: testpod
        image: alpine:3.5
        command: ["ping", "8.8.8.8"]
    ```

    这描述了一个带有单个容器的 pod，将一个简单的 ping 隔离到 8.8.8.8。

4.  在 PowerShell 中，导航到您创建的位置pod.yaml并创建 pod：

    ```console
    $ kubectl apply -f pod.yaml
    ```

5.  检查您的 Pod 是否已启动并正在运行：

    ```console
    $ kubectl get pods
    ```

    你应该看到类似的东西：

    ```shell
    NAME      READY     STATUS    RESTARTS   AGE
    demo      1/1       Running   0          4s
    ```

6.  检查您是否获得了 ping 过程所需的日志：

    ```console
    $ kubectl logs demo
    ```

    您应该会看到正常 ping 过程的输出：

    ```shell
    PING 8.8.8.8 (8.8.8.8): 56 data bytes
    64 bytes from 8.8.8.8: seq=0 ttl=37 time=21.393 ms
    64 bytes from 8.8.8.8: seq=1 ttl=37 time=15.320 ms
    64 bytes from 8.8.8.8: seq=2 ttl=37 time=11.111 ms
    ...
    ```

7.  最后，拆掉你的测试 pod：

    ```console
    $ kubectl delete -f pod.yaml
    ```

{% endcapture %}
{{ localwin-content | markdownify }}
</div>
<hr>
</div>

## 开启 Docker Swarm

Docker Desktop 主要在 Docker Engine 上运行，它内置了运行 Swarm 所需的一切。 按照适用于您的操作系统的设置和验证说明进行操作：

<ul class="nav nav-tabs">
  <li class="active"><a data-toggle="tab" href="#swarmosx">Mac</a></li>
  <li><a data-toggle="tab" href="#swarmwin">Windows</a></li>
</ul>
<div class="tab-content">
  <div id="swarmosx" class="tab-pane fade in active">
{% capture local-content %}

### Mac

1.  打开终端，并初始化 Docker Swarm 模式：

    ```console
    $ docker swarm init
    ```

    如果一切顺利，您应该会看到类似于以下内容的消息：

    ```shell
    Swarm initialized: current node (tjjggogqpnpj2phbfbz8jd5oq) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-3e0hh0jd5t4yjg209f4g5qpowbsczfahv2dea9a1ay2l8787cf-2h4ly330d0j917ocvzw30j5x9 192.168.65.3:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

2.  运行一个简单的 Docker 服务，它使用基于 alpine 的文件系统，并将 ping 隔离到 8.8.8.8：

    ```console
    $ docker service create --name demo alpine:3.5 ping 8.8.8.8
    ```

3.  检查您的服务是否创建了一个正在运行的容器：

    ```console
    $ docker service ps demo
    ```

    你应该看到类似的东西：

    ```shell
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
    463j2s3y4b5o        demo.1              alpine:3.5          docker-desktop      Running             Running 8 seconds ago
    ```

4.  检查您是否获得了 ping 过程所需的日志：

    ```console
    $ docker service logs demo
    ```

    您应该会看到正常 ping 过程的输出：

    ```shell
    demo.1.463j2s3y4b5o@docker-desktop    | PING 8.8.8.8 (8.8.8.8): 56 data bytes
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=0 ttl=37 time=13.005 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=1 ttl=37 time=13.847 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=2 ttl=37 time=41.296 ms
    ...
    ```

5.  最后，拆除您的测试服务：

    ```console
    $ docker service rm demo
    ```

{% endcapture %}
{{ local-content | markdownify }}

</div>
<div id="swarmwin" class="tab-pane fade" markdown="1">
{% capture localwin-content %}

### Windows

1.  Open a powershell, and initialize Docker Swarm mode:

    ```console
    $ docker swarm init
    ```

    If all goes well, you should see a message similar to the following:

    ```shell
    Swarm initialized: current node (tjjggogqpnpj2phbfbz8jd5oq) is now a manager.

    To add a worker to this swarm, run the following command:

        docker swarm join --token SWMTKN-1-3e0hh0jd5t4yjg209f4g5qpowbsczfahv2dea9a1ay2l8787cf-2h4ly330d0j917ocvzw30j5x9 192.168.65.3:2377

    To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
    ```

2.  Run a simple Docker service that uses an alpine-based filesystem, and isolates a ping to 8.8.8.8:

    ```console
    $ docker service create --name demo alpine:3.5 ping 8.8.8.8
    ```

3.  Check that your service created one running container:

    ```console
    $ docker service ps demo
    ```

    You should see something like:

    ```shell
    ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
    463j2s3y4b5o        demo.1              alpine:3.5          docker-desktop      Running             Running 8 seconds ago
    ```

4.  Check that you get the logs you'd expect for a ping process:

    ```console
    $ docker service logs demo
    ```

    You should see the output of a healthy ping process:

    ```shell
    demo.1.463j2s3y4b5o@docker-desktop    | PING 8.8.8.8 (8.8.8.8): 56 data bytes
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=0 ttl=37 time=13.005 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=1 ttl=37 time=13.847 ms
    demo.1.463j2s3y4b5o@docker-desktop    | 64 bytes from 8.8.8.8: seq=2 ttl=37 time=41.296 ms
    ...
    ```

5.  Finally, tear down your test service:

    ```console
    $ docker service rm demo
    ```

{% endcapture %}
{{ localwin-content | markdownify }}
</div>
<hr>
</div>

## 结论

此时，您已经确认可以在 Kubernetes 和 Swarm 中运行简单的容器化工作负载。下一步将是编写 Kubernetes yaml，描述如何在 Kubernetes 上运行和管理这些容器。

[On to deploying to Kubernetes >>](kube-deploy.md){: class="button primary-btn" style="margin-bottom: 30px; margin-right: 200%"}

要了解如何编写堆栈文件以帮助您在 Swarm 上运行和管理容器，请参阅 [Deploying to Swarm](swarm-deploy.md)。

## CLI references

Further documentation for all CLI commands used in this article are available here:

- [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)
- [`kubectl get`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get)
- [`kubectl logs`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs)
- [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete)
- [`docker swarm init`](https://docs.docker.com/engine/reference/commandline/swarm_init/)
- [`docker service *`](https://docs.docker.com/engine/reference/commandline/service/)
