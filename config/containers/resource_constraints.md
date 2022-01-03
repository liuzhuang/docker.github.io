---
redirect_from:
- /engine/articles/systemd/
- /engine/admin/resource_constraints/
title: "带有内存、CPU 和 GPU 的运行时选项"
description: "Specify the runtime options for a container"
keywords: "docker, daemon, configuration, runtime"
---

默认情况下，容器没有资源限制，可以使用主机的内核调度程序允许的尽可能多的给定资源。
Docker 提供了控制容器可以使用多少内存或 CPU 的方法，设置 `docker run` 命令的运行时配置标志。
本节提供有关何时应该设置此类限制以及设置这些限制的可能影响的详细信息。

其中许多功能需要您的内核支持 Linux 功能。
要检查支持，您可以使用该 [`docker info`](../../engine/reference/commandline/info.md) 命令。
如果您的内核中禁用了某个功能，您可能会在输出的末尾看到如下警告：

```console
WARNING: No swap limit support
```

请查阅操作系统的文档以启用它们。
了解 [Learn more](../../engine/install/linux-postinstall.md#your-kernel-does-not-support-cgroup-swap-limit-capabilities)。

## 内存

### 了解内存溢出的风险

重要的是不要让正在运行的容器消耗过多的主机内存。
在 Linux 主机上，如果内核检测到没有足够的内存来执行重要的系统功能，它会抛出 `OOME` 或 `Out Of Memory Exception`，并开始杀死进程以释放内存。任何进程都会被杀死，包括 Docker 和其他重要的应用程序。
如果错误的进程被杀死，这将整个系统停机。

Docker 试图通过调整 Docker daemon 的 OOM 优先级来降低这些风险，使其比系统上的其他进程更不可能被杀死。
容器的 OOM 优先级没有调整。这使得单个容器更有可能被杀死，而不是 Docker 守护进程或其他系统进程被杀死。
您不应试图通过 `--oom-score-adj` 在守护程序或容器上手动设置为极端负数，或通过 `--oom-kill-disable` 在容器上设置来规避这些保护措施。

有关 Linux 内核的 OOM 管理的更多信息，请参阅 [内存不足管理](https://www.kernel.org/doc/gorman/html/understand/understand016.html){: target="_blank" rel="noopener" class="_" }。

您可以通过以下方式降低 OOME 导致的系统不稳定风险：

- 在将应用程序投入生产之前，执行测试以了解应用程序的内存要求。
- 确保您的应用程序仅在具有足够资源的主机上运行。
- 限制容器可以使用的内存量，如下所述。
- 在 Docker 主机上配置交换时要注意。Swap 比内存更慢且性能更差，但可以提供缓冲以防止系统内存耗尽。
- 考虑将您的容器转换为[service](../../engine/swarm/services.md)，并使用服务级别约束和节点标签来确保应用程序仅在具有足够内存的主机上运行

### 限制容器对内存的访问

Docker 可以强制执行硬内存限制，允许容器使用不超过给定数量的用户或系统内存，或软限制，允许容器使用尽可能多的内存，除非满足某些条件，例如内核检测到主机上的内存不足或争用。
当单独使用或设置多个选项时，其中一些选项会产生不同的效果。

这些选项中的大多数采用正整数，后跟 b`, `k`, `m`, `g`, 表示字节、千字节、兆字节或千兆字节。

| Option | Description |
|:-------|:------------|
| `-m` or `--memory=`    | The maximum amount of memory the container can use. If you set this option, the minimum allowed value is `6m` (6 megabytes). That is, you must set the value to at least 6 megabytes.  |
| `--memory-swap`*       | The amount of memory this container is allowed to swap to disk. See [`--memory-swap` details](#--memory-swap-details). |
| `--memory-swappiness`  | By default, the host kernel can swap out a percentage of anonymous pages used by a container. You can set `--memory-swappiness` to a value between 0 and 100, to tune this percentage. See [`--memory-swappiness` details](#--memory-swappiness-details).|
| `--memory-reservation` | Allows you to specify a soft limit smaller than `--memory` which is activated when Docker detects contention or low memory on the host machine. If you use `--memory-reservation`, it must be set lower than `--memory` for it to take precedence. Because it is a soft limit, it does not guarantee that the container doesn't exceed the limit.                                      |
| `--kernel-memory`      | The maximum amount of kernel memory the container can use. The minimum allowed value is `4m`. Because kernel memory cannot be swapped out, a container which is starved of kernel memory may block host machine resources, which can have side effects on the host machine and on other containers. See [`--kernel-memory` details](#--kernel-memory-details).            |
| `--oom-kill-disable`   | By default, if an out-of-memory (OOM) error occurs, the kernel kills processes in a container. To change this behavior, use the `--oom-kill-disable` option. Only disable the OOM killer on containers where you have also set the `-m/--memory` option. If the `-m` flag is not set, the host can run out of memory and the kernel may need to kill the host system's processes to free memory. |

有关 cgroup 和内存的更多信息，请参阅 [Memory Resource Controller](https://www.kernel.org/doc/Documentation/cgroup-v1/memory.txt) 文档。

### `--memory-swap` 详解

`--memory-swap` 是一个修饰符标志，只有在设置 `--memory` 时才有意义。
当容器耗尽所有可用的 RAM 时，使用 swap 允许容器将多余的内存需求写入磁盘。
经常将内存交换到磁盘的应用程序会降低性能。

它的设置可能会产生复杂的影响：

- 如果 `--memory-swap` 设置为正整数，则 `--memory` 和 `--memory-swap` 都必须设置。
  `--memory-swap`  表示可以使用的内存和 swap 总量，`--memory` 控制非 swap 内存使用量。
  所以如果设置 `--memory="300m"` 和 `--memory-swap="1g"`，容器可以使用 300m 的内存和 700m (`1g - 300m`) swap。

- 如果 `--memory-swap` 设置为 `0`，则忽略该设置，并将该值视为未设置。

- 如果 `--memory-swap` 设置为与 `--memory` 相同的值，并且 `--memory` 设置为正整数，则容器无权访问 swap。
  请参阅[防止容器使用 swap](#prevent-a-container-from-using-swap)。

- 如果 `--memory-swap` 未设置，但设置了 `--memory`，则容器可以使用与 `--memory` 设置一样多的交换，如果主机容器配置了交换内存。
  例如，设置了 `--memory="300m"`， 没有设置 `--memory-swap`，该容器总共可以使用600m的内存和交换。

- 如果 `--memory-swap` 显式设置为 `-1`，则允许容器使用无限交换，最多可达主机系统上可用的数量。

- 在容器内部，诸如 `free` 命名报告主机可用交换之类的工具，而不是容器内部可用的内容。
  不要依赖于 `free` 或类似工具的输出来确定是否存在交换。

#### 防止容器使用 swap

如果 `--memory` 和 `--memory-swap` 被设置为相同的值，这会阻止容器使用任何 swap。
这是因为 `--memory-swap` 是可以使用的组合内存和 swap 的数量，然而 `--memory` 是可以使用的物理内存量。

### `--memory-swappiness` 详解

- 值 0 关闭匿名页面交换。
- 值 100 将所有匿名页面设置为可交换页面。
- 默认情况下，如果您不设置 `--memory-swappiness`，则该值是从主机继承的。

### `--kernel-memory` 详解

内核内存限制以分配给容器的总内存表示。考虑以下场景：

- 无限内存，无限内核内存：
  这是默认行为。
- 无限内存，有限内核内存：
  当所有 cgroup 所需的内存量大于主机上实际存在的内存量时，这是合适的。
  您可以将内核内存配置为永远不会超过主机上可用的内存，并且需要更多内存的容器需要等待它。
- 有限内存，无限内核内存：
  整体内存是有限的，但内核内存不是。
- 有限的内存，有限的内核内存：
  限制用户和内核内存对于调试与内存相关的问题很有用。
  如果容器使用了意外数量的任一类型的内存，它就会耗尽内存而不会影响其他容器或主机。
  在此设置中，如果内核内存限制低于用户内存限制，则内核内存不足会导致容器遇到 OOM 错误。
  如果内核内存限制高于用户内存限制，则内核限制不会导致容器出现 OOM。

当您打开任何内核内存限制时，主机会在每个进程的基础上跟踪“高水位线”统计信息，因此您可以跟踪哪些进程（在本例中为容器）使用了过多的内存。这可以通过 `/proc/<PID>/status` 在主机上查看来查看每个进程。

## CPU

默认情况下，每个容器对主机 CPU 周期的访问是无限制的。
您可以设置各种约束来限制给定容器对主机 CPU 周期的访问。
大多数用户使用和配置 默认的 [CFS 调度程序](#configure-the-default-cfs-scheduler)。
您还可以配置[实时调度程序](#configure-the-realtime-scheduler)。

### 配置默认的 CFS scheduler

CFS 是用于正常 Linux 进程的 Linux 内核 CPU 调度程序。
多个运行时标志允许您配置对容器拥有的 CPU 资源的访问量。
当您使用这些设置时，Docker 会修改主机上容器的 cgroup 的设置。

| Option | Description |
|:-------|:------------|
| `--cpus=<value>`       | Specify how much of the available CPU resources a container can use. For instance, if the host machine has two CPUs and you set `--cpus="1.5"`, the container is guaranteed at most one and a half of the CPUs. This is the equivalent of setting `--cpu-period="100000"` and `--cpu-quota="150000"`.                                                                                                                                                                                                                                                                                                |
| `--cpu-period=<value>` | Specify the CPU CFS scheduler period, which is used alongside  `--cpu-quota`. Defaults to 100000 microseconds (100 milliseconds). Most users do not change this from the default. For most use-cases, `--cpus` is a more convenient alternative.                                                                                                                                                                                                                                                                                                                                                     |
| `--cpu-quota=<value>`  | Impose a CPU CFS quota on the container. The number of microseconds per `--cpu-period` that the container is limited to before throttled. As such acting as the effective ceiling. For most use-cases, `--cpus` is a more convenient alternative.                                                                                                                                                                                                                                                                                                                                                    |
| `--cpuset-cpus`        | Limit the specific CPUs or cores a container can use. A comma-separated list or hyphen-separated range of CPUs a container can use, if you have more than one CPU. The first CPU is numbered 0. A valid value might be `0-3` (to use the first, second, third, and fourth CPU) or `1,3` (to use the second and fourth CPU).                                                                                                                                                                                                                                                                          |
| `--cpu-shares`         | Set this flag to a value greater or less than the default of 1024 to increase or reduce the container's weight, and give it access to a greater or lesser proportion of the host machine's CPU cycles. This is only enforced when CPU cycles are constrained. When plenty of CPU cycles are available, all containers use as much CPU as they need. In that way, this is a soft limit. `--cpu-shares` does not prevent containers from being scheduled in swarm mode. It prioritizes container CPU resources for the available CPU cycles. It does not guarantee or reserve any specific CPU access. |

如果您有 1 个 CPU，则以下每个命令都保证容器每秒最多使用 50% 的 CPU。

```console
$ docker run -it --cpus=".5" ubuntu /bin/bash
```

这相当于手动指定 `--cpu-period` 和 `--cpu-quota`;

```console
$ docker run -it --cpu-period=100000 --cpu-quota=50000 ubuntu /bin/bash
```

### 配置 realtime scheduler

对于不能使用 CFS 调度程序的任务，您可以将容器配置为使用实时调度程序。
您需要[确保主机的内核配置正确](#configure-the-host-machines-kernel)， 然后才能[配置 Docker 守护程序](#configure-the-docker-daemon)或[配置单个容器](#configure-individual-containers)。

> **Warning**
>
> CPU 调度和优先级是高级内核级功能。
> 大多数用户不需要更改这些默认值。
> 错误地设置这些值可能会导致您的主机系统变得不稳定或无法使用。
{:.warning}

#### 配置主机内核

通过运行 `zcat /proc/config.gz | grep CONFIG_RT_GROUP_SCHED` 或检查`/sys/fs/cgroup/cpu.rt_runtime_us` 文件是否存在来验证在 Linux 内核中是否已启用 `CONFIG_RT_GROUP_SCHED`。
有关配置内核实时调度程序的指导，请参阅您的操作系统的文档。

#### 配置 Docker daemon

要使用实时调度程序运行容器，请运行 Docker daemon，并将 `--cpu-rt-runtime` 标志设置为每个运行时为实时任务保留的最大微秒数。
例如，默认周期为 1000000 微秒（1 秒），设置 `--cpu-rt-runtime=950000` 可确保使用实时调度程序的容器每 1000000 微秒周期可以运行 950000 微秒，留下至少 50000 微秒可用于非实时任务。
要使此配置在使用的系统上永久存在 `systemd`，请参阅[使用 systemd 控制和配置 Docker](../daemon/systemd.md)。

#### 配置单个 containers

当您使用 `docker run` 传递几个标志来控制一个容器的 CPU 优先级，有关 `ulimit` 值的信息，请查阅操作系统的文档或命令。

| Option | Description   |
|:-------|:--------------|
| `--cap-add=sys_nice`       | Grants the container the `CAP_SYS_NICE` capability, which allows the container to raise process `nice` values, set real-time scheduling policies, set CPU affinity, and other operations. |
| `--cpu-rt-runtime=<value>` | The maximum number of microseconds the container can run at realtime priority within the Docker daemon's realtime scheduler period. You also need the `--cap-add=sys_nice` flag.          |
| `--ulimit rtprio=<value>`  | The maximum realtime priority allowed for the container. You also need the `--cap-add=sys_nice` flag.                                                                                     |

以下示例命令在 `debian:jessie` 容器上设置这三个标志中的每一个。

The following example command sets each of these three flags on a `debian:jessie`
container.

```console
$ docker run -it \
    --cpu-rt-runtime=950000 \
    --ulimit rtprio=99 \
    --cap-add=sys_nice \
    debian:jessie
```

如果内核或 Docker daemon 没有正确配置，就会发生错误。

## GPU

### 使用 NVIDIA GPU

#### 先决条件

访问官方[NVIDIA 驱动程序页面](https://www.nvidia.com/Download/index.aspx) 以下载并安装正确的驱动程序。完成后重新启动系统。

验证您的 GPU 是否正在运行且可访问。

#### 安装 nvidia-container-runtime

按照 (https://nvidia.github.io/nvidia-container-runtime/) 上的说明操作，然后运行以下命令：

```console
$ apt-get install nvidia-container-runtime
```

确保 `nvidia-container-runtime-hook` 可从 `$PATH` 访问。

```console
$ which nvidia-container-runtime-hook
```

重启 Docker daemon。

#### Expose GPUs for use

在启动容器以访问 GPU 资源时包含 `--gpus` 标志。指定要使用的 GPU 数量。例如：

```console
$ docker run -it --rm --gpus all ubuntu nvidia-smi
```

公开所有可用的 GPU 并返回类似于以下内容的结果：

```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.130            	Driver Version: 384.130               	|
|-------------------------------+----------------------+----------------------+
| GPU  Name 	   Persistence-M| Bus-Id    	Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GRID K520       	Off  | 00000000:00:03.0 Off |                  N/A |
| N/A   36C	P0    39W / 125W |  	0MiB /  4036MiB |      0%  	Default |
+-------------------------------+----------------------+----------------------+
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU   	PID   Type   Process name                         	Usage  	|
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

使用该 `device` 选项指定 GPU。例如：

```console
$ docker run -it --rm --gpus device=GPU-3a23c669-1f69-c64e-cf85-44e9b07e7a2a ubuntu nvidia-smi
```

公开特定的 GPU。

```console
$ docker run -it --rm --gpus '"device=0,2"' ubuntu nvidia-smi
```

暴露第一个和第三个 GPU。

> **Note**
>
> NVIDIA GPU 只能由运行单个引擎的系统访问。

#### Set NVIDIA capabilities

您可以手动设置功能。例如，在 Ubuntu 上，您可以运行以下命令：

```console
$ docker run --gpus 'all,capabilities=utility' --rm ubuntu nvidia-smi
```

This enables the `utility` driver capability which adds the `nvidia-smi` tool to
the container.

Capabilities as well as other configurations can be set in images via
environment variables. More information on valid variables can be found at the
[nvidia-container-runtime](https://github.com/NVIDIA/nvidia-container-runtime)
GitHub page. These variables can be set in a Dockerfile.

You can also utitize CUDA images which sets these variables automatically. See
the [CUDA images](https://github.com/NVIDIA/nvidia-docker/wiki/CUDA) GitHub page
for more information.
