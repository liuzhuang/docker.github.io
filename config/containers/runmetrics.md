---
description: Measure the behavior of running containers
keywords: docker, metrics, CPU, memory, disk, IO, run, runtime, stats
redirect_from:
- /articles/runmetrics/
- /engine/articles/run_metrics
- /engine/articles/runmetrics
- /engine/admin/runmetrics/
title: 运行时指标
---

## Docker 统计数据

您可以使用 `docker stats` 命令实时的查看容器的运行时指标。
该命令支持 CPU、内存使用、内存限制和网络 IO 指标。

以下是 `docker stats` 命令的示例输出

```console
$ docker stats redis1 redis2

CONTAINER           CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O
redis1              0.07%               796 KB / 64 MB        1.21%               788 B / 648 B       3.568 MB / 512 KB
redis2              0.07%               2.746 MB / 64 MB      4.29%               1.266 KB / 648 B    12.4 MB / 0 B
```

[docker stats](../../engine/reference/commandline/stats.md) 参考页面有更多关于 `docker stats` 命令的细节 。

## Control groups

Linux 容器依赖于[control groups](
https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt) ，这些 control groups 不仅跟踪进程组，还公开 CPU、内存和 I/O 使用情况指标。
您也可以访问这些指标并获取网络使用指标。这与 “纯” LXC 容器以及 Docker 容器相关。

Control groups 通过伪文件系统公开。
在最近的发行版中，您应该在 `/sys/fs/cgroup` 看到这个伪文件系统。
在该目录下，您会看到多个子目录，称为 devices、freezer、blkio 等；
每个子目录实际上对应于不同的 cgroup 层次结构。

在较旧的系统上，control groups 可能挂载在 `/cgroup` 上，没有明显的层次结构。
在这种情况下，您不会看到子目录，而是在该目录中看到一堆文件，可能还有一些与现有容器对应的目录。

要确定 control groups 的挂载位置，您可以运行：

```console
$ grep cgroup /proc/mounts
```

### Enumerate cgroups

cgroups 的文件布局在 v1 和 v2 之间有显着差异。

如果  `/sys/fs/cgroup/cgroup.controllers` 存在于您的系统上，则您使用的是 v2，否则您使用的是 v1。请参阅与您的 cgroup 版本相对应的小节。

cgroup v2 默认用于以下发行版：
- Fedora (since 31)
- Debian GNU/Linux (since 11)
- Ubuntu (since 21.10)

#### cgroup v1

您可以查看`/proc/cgroups`系统已知的不同控制组子系统、它们所属的层次结构以及它们包含的组数。

您还可以查看 `/proc/<pid>/cgroup` 进程属于哪些控制组。控制组显示为相对于层次结构挂载点根的路径。`/`表示该进程尚未分配到组，而`/lxc/pumpkin`表示该进程是名为 的容器的成员 `pumpkin`.。

#### cgroup v2

在 cgroup v2 主机上， `/proc/cgroups` 的内容没有意义。查看 `/sys/fs/cgroup/cgroup.controllers` 可用的控制器。

### Changing cgroup version

更改 cgroup 版本需要重新启动整个系统。

在基于 systemd 的系统上，可以通过添加`systemd.unified_cgroup_hierarchy=1` 到 kernel cmdline 来启用 cgroup v2 。要将 cgroup 版本恢复为 v1，您需要改为设置 `systemd.unified_cgroup_hierarchy=0`。

如果 `grubby` 命令在您的系统上可用（例如在 Fedora 上），则可以按如下方式修改 cmdline：

```console
$ sudo grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=1"
```

如果 `grubby` 命令不可用，请编辑 `/etc/default/grub` 文件的 `GRUB_CMDLINE_LINUX` 行并运行 `sudo update-grub`。

### Running Docker on cgroup v2

Docker 从 Docker 20.10 开始支持 cgroup v2。
在 cgroup v2 上运行 Docker 还需要满足以下条件：
* containerd: v1.4 或更高版本
* runc: v1.0.0-rc91 或更高版本
* Kernel: v4.15或更高版本（推荐v5.2 或更高版本）

请注意，cgroup v2 模式的行为与 cgroup v1 模式略有不同：

* 默认的 cgroup 驱动程序 (`dockerd --exec-opt native.cgroupdriver`) 在 v2 上是 “systemd”，在 v1 上是 “cgroupfs”。
* 默认的 cgroup 命名空间模式 (`docker run --cgroupns`) 在 v2 上为 “private”，在 v1 上为 “host”。
* `docker run` 的标志 `--oom-kill-disable` 和 `--kernel-memory` 在 V2 上被丢弃。

### 查找给定容器的 cgroup

对于每一个容器，在每个层次结构中会创建一个 cgroup。
在旧系统上的旧版 LXC 用户空间工具上，cgroup 的名称是容器的名称。
在新版本的 LXC 工具上，cgroup 的名字是 `lxc/<container_name>`。

对于使用 cgroups 的 Docker 容器，容器的名称是 full ID 或 long ID。
如果容器在 `docker ps` 中显示为 `ae836c95b4c3` ，则其 long ID 可能类似于 `ae836c95b4c3c9e9179e0e91015512da89fdec91612f63cebae57df9a5444c79`。 
您可以使用 `docker inspect` 或 `docker ps --no-trunc` 查找它。

将所有内容放在一起以查看 Docker 容器的内存指标，请查看以下路径：
- `/sys/fs/cgroup/memory/docker/<longid>/` cgroup v1, `cgroupfs` 驱动
- `/sys/fs/cgroup/memory/system.slice/docker-<longid>.scope/` cgroup v1, `systemd` 驱动
- `/sys/fs/cgroup/docker/<longid/>` cgroup v2, `cgroupfs` 驱动
- `/sys/fs/cgroup/system.slice/docker-<longid>.scope/` cgroup v2, `systemd` 驱动

### 来自 cgroups 的指标: memory, CPU, block I/O

> **Note**
>
> 此部分尚未针对 cgroup v2 进行更新。
> 有关 cgroup v2 的更多信息，请参阅 [the kernel documentation](https://www.kernel.org/doc/html/latest/admin-guide/cgroup-v2.html)。

对于每个子系统（内存、CPU 和块 I/O），存在一个或多个伪文件并包含统计信息。

#### 内存指标: `memory.stat`

内存指标可在 "memory" cgroup 中找到。
内存控制组会增加一些开销，因为它对主机上的内存使用情况进行了非常细粒度的统计。
因此，许多发行版选择默认不启用它。
通常，要启用它，您所要做的就是添加一些内核命令行参数： `cgroup_enable=memory swapaccount=1`。

指标在伪文件 `memory.stat` 中。这是它的样子：

    cache 11492564992
    rss 1930993664
    mapped_file 306728960
    pgpgin 406632648
    pgpgout 403355412
    swap 0
    pgfault 728281223
    pgmajfault 1724
    inactive_anon 46608384
    active_anon 1884520448
    inactive_file 7003344896
    active_file 4489052160
    unevictable 32768
    hierarchical_memory_limit 9223372036854775807
    hierarchical_memsw_limit 9223372036854775807
    total_cache 11492564992
    total_rss 1930993664
    total_mapped_file 306728960
    total_pgpgin 406632648
    total_pgpgout 403355412
    total_swap 0
    total_pgfault 728281223
    total_pgmajfault 1724
    total_inactive_anon 46608384
    total_active_anon 1884520448
    total_inactive_file 7003344896
    total_active_file 4489052160
    total_unevictable 32768

前半部分（不带 `total_` 前缀）包含与 cgroup 中的进程相关的统计信息，不包括子 cgroup。
后半部分（带有 `total_` 前缀）包括子 cgroup。


一些指标是 "gauges"，或者可以增加或减少的值。
例如， `swap` 是 cgroup 成员使用的交换空间量。
其他一些是 "counters"，或只能上升的值，因为它们代表特定事件的发生。
例如，`pgfault` 指示自 cgroup 创建以来的缺页次数。


<style>table tr > td:first-child { white-space: nowrap;}</style>

Metric                                | Description
--------------------------------------|-----------------------------------------------------------
**cache**                             | The amount of memory used by the processes of this control group that can be associated precisely with a block on a block device. When you read from and write to files on disk, this amount increases. This is the case if you use "conventional" I/O (`open`, `read`, `write` syscalls) as well as mapped files (with `mmap`). It also accounts for the memory used by `tmpfs` mounts, though the reasons are unclear.
**rss**                               | The amount of memory that *doesn't* correspond to anything on disk: stacks, heaps, and anonymous memory maps.
**mapped_file**                       | Indicates the amount of memory mapped by the processes in the control group. It doesn't give you information about *how much* memory is used; it rather tells you *how* it is used.
**pgfault**, **pgmajfault**           | Indicate the number of times that a process of the cgroup triggered a "page fault" and a "major fault", respectively. A page fault happens when a process accesses a part of its virtual memory space which is nonexistent or protected. The former can happen if the process is buggy and tries to access an invalid address (it is sent a `SIGSEGV` signal, typically killing it with the famous `Segmentation fault` message). The latter can happen when the process reads from a memory zone which has been swapped out, or which corresponds to a mapped file: in that case, the kernel loads the page from disk, and let the CPU complete the memory access. It can also happen when the process writes to a copy-on-write memory zone: likewise, the kernel preempts the process, duplicate the memory page, and resume the write operation on the process's own copy of the page. "Major" faults happen when the kernel actually needs to read the data from disk. When it just  duplicates an existing page, or allocate an empty page, it's a regular (or "minor") fault.
**swap**                              | The amount of swap currently used by the processes in this cgroup.
**active_anon**, **inactive_anon**    | The amount of *anonymous* memory that has been identified has respectively *active* and *inactive* by the kernel. "Anonymous" memory is the memory that is *not* linked to disk pages. In other words, that's the equivalent of the rss counter described above. In fact, the very definition of the rss counter is **active_anon** + **inactive_anon** - **tmpfs** (where tmpfs is the amount of memory used up by `tmpfs` filesystems mounted by this control group). Now, what's the difference between "active" and "inactive"? Pages are initially "active"; and at regular intervals, the kernel sweeps over the memory, and tags some pages as "inactive". Whenever they are accessed again, they are immediately retagged "active". When the kernel is almost out of memory, and time comes to swap out to disk, the kernel swaps "inactive" pages.
**active_file**, **inactive_file**    | Cache memory, with *active* and *inactive* similar to the *anon* memory above. The exact formula is **cache** = **active_file** + **inactive_file** + **tmpfs**. The exact rules used by the kernel to move memory pages between active and inactive sets are different from the ones used for anonymous memory, but the general principle is the same. When the kernel needs to reclaim memory, it is cheaper to reclaim a clean (=non modified) page from this pool, since it can be reclaimed immediately (while anonymous pages and dirty/modified pages need to be written to disk first).
**unevictable**                       | The amount of memory that cannot be reclaimed; generally, it accounts for memory that has been "locked" with `mlock`. It is often used by crypto frameworks to make sure that secret keys and other sensitive material never gets swapped out to disk.
**memory_limit**, **memsw_limit**     | These are not really metrics, but a reminder of the limits applied to this cgroup. The first one indicates the maximum amount of physical memory that can be used by the processes of this control group; the second one indicates the maximum amount of RAM+swap.


在页面缓存中计算内存非常复杂。
如果不同控制组中的两个进程都读取同一个文件（最终依赖于磁盘上的相同块），则相应的内存费用在控制组之间分配。
这很好，但这也意味着当一个 cgroup 终止时，它可能会增加另一个 cgroup 的内存使用量，因为它们不再分摊这些内存页面的成本。

### CPU 指标: `cpuacct.stat`

现在我们已经介绍了内存指标，相比之下，其他一切都很简单。
CPU 指标在 `cpuacct` 控制器中。


对于每一个容器，一个伪文件 `cpuacct.stat` 包含由容器的进程累积的 CPU 使用率，分解成 `user` 和 `system`时间。区别在于：

- `user` 时间是进程直接控制 CPU 执行进程代码的时间。
- `system` time 是内核代表进程执行系统调用的时间。

这些时间以 1/100 秒的刻度表示，也被称为 “user jiffies”。
每秒有 `USER_HZ` * "jiffies"* ，在 x86 系统上 `USER_HZ` 是 100。
从历史上看，这完全映射到每秒调度程序 "ticks" 的数量，但更高频率的调度和[tickless kernels](https://lwn.net/Articles/549580/) 使“ticks” 数变得无关紧要。

#### Block I/O metrics

Block I/O 在 `blkio` 控制器中计算。
不同的指标分散在不同的文件中。
然而您可以在[内核文档](https://www.kernel.org/doc/Documentation/cgroup-v1/blkio-controller.txt) 的文件中找到更深入的细节，
以下是最相关的简短列表：

Metric                      | Description
----------------------------|-----------------------------------------------------------
**blkio.sectors**           | Contains the number of 512-bytes sectors read and written by the processes member of the cgroup, device by device. Reads and writes are merged in a single counter.
**blkio.io_service_bytes**  | Indicates the number of bytes read and written by the cgroup. It has 4 counters per device, because for each device, it differentiates between synchronous vs. asynchronous I/O, and reads vs. writes.
**blkio.io_serviced**       | The number of I/O operations performed, regardless of their size. It also has 4 counters per device.
**blkio.io_queued**         | Indicates the number of I/O operations currently queued for this cgroup. In other words, if the cgroup isn't doing any I/O, this is zero. The opposite is not true. In other words, if there is no I/O queued, it does not mean that the cgroup is idle (I/O-wise). It could be doing purely synchronous reads on an otherwise quiescent device, which can therefore handle them immediately, without queuing. Also, while it is helpful to figure out which cgroup is putting stress on the I/O subsystem, keep in mind that it is a relative quantity. Even if a process group does not perform more I/O, its queue size can increase just because the device load increases because of other devices.

### Network metrics

control groups 不直接暴露网络指标。
对此有一个很好的解释：网络接口存在于 **网络命名空间** 的上下文中。
内核可能会累积一组进程发送和接收的数据包和字节的指标，但这些指标不会很有用。
您需要每个接口的指标（因为本地 `lo` 接口上发生的流量实际上并不重要）。
但是由于单个 cgroup 中的进程可以属于多个网络命名空间，这些指标将更难以解释：多个网络命名空间意味着多个 `lo` 接口，可能是多个 `eth0`  接口等；所以这就是为什么没有简单的方法可以通过 control groups 收集网络指标。

相反，我们可以从其他来源收集网络指标：

#### IPtables

IPtables（或者更确切地说，iptables 只是一个接口的 netfilter 框架）可以做一些严肃的计算。

例如，您可以设置规则来解释 Web 服务器上的出站 HTTP 流量：

```console
$ iptables -I OUTPUT -p tcp --sport 80
```

没有 `-j` 或 `-g`标志，所以规则只计算匹配的数据包并转到以下规则。

稍后，您可以使用以下命令检查计数器的值：

```console
$ iptables -nxvL OUTPUT
```

从技术上讲，`-n` 不是必需的，但它会阻止 iptables 进行 DNS 反向查找，这在这种情况下可能没用。

计数器包括数据包和字节。
如果您想为这样的容器流量设置指标，您可以执行一个 `for` 循环，在 `FORWARD` 链中为每个容器 IP 地址添加两个 `iptables` 规则（每个方向一个）。
这仅计量通过 NAT 层的流量；您还需要添加通过用户空间代理的流量。

然后，您需要定期检查这些计数器。如果您碰巧使用 `collectd`,，有一个很好的[插件](https://collectd.org/wiki/index.php/Table_of_Plugins) 可以自动收集 iptables 计数器。

#### 网卡级别计数器

由于每个容器都有一个虚拟以太网接口(virtual Ethernet interface)，您可能需要直接检查该接口的 TX 和 RX 计数器。
每个容器都与主机中的一个虚拟以太网接口相关联，名称类似于 `vethKk8Zqi`。
不幸的是，弄清楚哪个接口对应于哪个容器是很困难的。


但就目前而言，最好的方法是检查**容器内的指标**。
为此，您可以使用 **ip-netns magic** 从容器的网络命名空间内的主机环境中运行可执行文件。

`ip-netns exec` 命令允许您在当前进程可见的任何网络命名空间中执行任何程序（存在于主机系统中）。
这意味着您的主机可以进入容器的网络命名空间，但您的容器无法访问该主机或其他对等容器。
不过，容器可以与其子容器进行交互。

命令的格式是：

```console
$ ip netns exec <nsname> <command...>
```

例如：

```console
$ ip netns exec mycontainer netstat -i
```

`ip netns` 使用命名空间伪文件找到 "mycontainer" 容器。
每个进程属于一个网络命名空间、一个 PID 命名空间、一个 `mnt` 命名空间等，这些命名空间在 `/proc/<pid>/ns/`. 
例如，PID 42 的网络命名空间由伪文件 `/proc/42/ns/net` 具体化。


当您运行 `ip netns exec mycontainer ...` 时，`/var/run/netns/mycontainer` 应该是这些伪文件之一。（接受符号链接。）

换句话说，要在容器的网络命名空间内执行命令，我们需要：
In other words, to execute a command within the network namespace of a
container, we need to:

- 找出我们要调查的容器内任何进程的 PID；
- 创建一个符号链接，从 `/var/run/netns/<somename>` 到 `/proc/<thepid>/ns/net`
- 执行 `ip netns exec <somename> ....`

查看 [Enumerate Cgroups](#enumerate-cgroups) 以了解如何查找要测量其网络使用情况的容器内进程的 cgroup。
从那里，您可以检查名为 `tasks` 的伪文件，其中包含 cgroup 中（因此也包含在容器中）的所有 PID。选择任一 PID。

将所有内容放在一起，如果容器的 "short ID" 保存在环境变量  `$CID` 中，那么您可以这样做：

```console
$ TASKS=/sys/fs/cgroup/devices/docker/$CID*/tasks
$ PID=$(head -n 1 $TASKS)
$ mkdir -p /var/run/netns
$ ln -sf /proc/$PID/ns/net /var/run/netns/$CID
$ ip netns exec $CID netstat -i
```

## 高性能指标收集

如果在每次收集指标的时候都新建一个进程，是相对昂贵的。
如果您想以高频率或大量容器（例如单主机上运行的 1000 个容器）收集指标，您不想每次都创建一个新进程。

以下讲解如何从单个进程收集指标。
您需要用 C（或任何允许您执行 low-level 系统调用的语言）编写 metric 收集器。
您需要使用一个特殊的系统调用 `setns()`，它可以让当前进程进入任意命名空间。
但是，它需要一个指向命名空间伪文件的打开文件描述符（请记住：那是 `/proc/<pid>/ns/net` 中的伪文件）。

但是，有一个问题：您不能让这个文件描述符保持打开状态。
如果这样做，当 control group 的最后一个进程退出时，命名空间不会被破坏，它的网络资源（如容器的虚拟接口）将永远存在（直到您关闭该文件描述符）。

正确的方法是跟踪每个容器的第一个 PID，并且每次都重新打开命名空间伪文件。

## 当容器退出时收集指标

有时，您并不关心实时收集的指标，但是当容器退出时，您想知道它使用了多少 CPU、内存等。

Docker 使这变得困难，因为它依赖于 `lxc-start`，它会在自己之后仔细清理。
定期收集指标通常更容易，因为它是 LXC 插件 `collectd` 的工作方式。

但是，如果您仍想在容器停止时收集统计信息，请执行以下操作：

对于每个容器，启动一个收集进程，通过将其 PID 写入 cgroup 的 tasks 文件，将其移动到要监控的 control groups 中。
收集进程应该定期重新读取任务文件以检查它是否是控制组的最后一个进程。（如果您还想按照上一节中的说明收集网络统计信息，您还应该将该进程移至适当的网络命名空间。）

当容器退出时，`lxc-start` 尝试删除 control groups。
它失败了，因为 control groups 仍在使用中；
不过没关系。您的进程现在应该检测到它是组中唯一剩余的进程。现在是收集您需要的所有指标的好时机！

最后，您的进程应该将自身移动到 root control group，并删除容器的 control group。
要删除 control group，只需 `rmdir` 自己的目录。`rmdir` 目录是违反直觉的，因为它仍然包含文件；
但请记住，这是一个伪文件系统，因此通常的规则不适用。清理完成后，收集过程可以安全退出。