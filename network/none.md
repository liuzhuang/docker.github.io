---
title: 禁用容器的网络
description: How to disable networking by using the none driver
keywords: network, none, standalone
---

如果要完全禁用容器上的网络堆栈，可以在启动容器时使用 `--network none` 标志。
在容器内，仅创建环回设备。
以下示例说明了这一点。

1.  创建容器。

    ```console
    $ docker run --rm -dit \
      --network none \
      --name no-net-alpine \
      alpine:latest \
      ash
    ```

2.  通过在容器内执行一些常见的网络命令来检查容器的网络堆栈。请注意，没有 `eth0` 被创建。

    ```console
    $ docker exec no-net-alpine ip link show

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN qlen 1
        link/ipip 0.0.0.0 brd 0.0.0.0
    3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN qlen 1
        link/tunnel6 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00 brd 00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
    ```

    ```console
    $ docker exec no-net-alpine ip route
    ```

    第二个命令返回空，因为没有路由表。

3.  停止容器。它会自动删除，因为它是使用 `--rm` 标志创建的。

    ```console
    $ docker stop no-net-alpine
    ```

## 下一步

- 浏览 [host networking tutorial](network-tutorial-host.md)
- 了解 [networking from the container's point of view](../config/containers/container-networking.md)
- 了解 [bridge networks](bridge.md)
- 了解 [overlay networks](overlay.md)
- 了解 [Macvlan networks](macvlan.md)
