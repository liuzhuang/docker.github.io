---
title: Networking with standalone containers
description: Tutorials for networking with standalone containers
keywords: networking, bridge, routing, ports, overlay
---

本系列教程涉及独立 Docker 容器的网络。有关使用 swarm 服务联网，请参阅 使用 swarm 服务联网。如果您需要了解有关 Docker 网络的更多信息，请参阅概述。

本主题包括三个不同的教程。您可以在 Linux、Windows 或 Mac 上运行它们中的每一个，但对于最后两个，您需要在其他地方运行第二个 Docker 主机。

- 使用默认桥接网络演示了如何使用bridgeDocker 自动为您设置的默认网络。该网络不是生产系统的最佳选择。

- 使用用户定义的桥接网络展示了如何创建和使用您自己的自定义桥接网络，以连接在同一 Docker 主机上运行的容器。推荐用于在生产中运行的独立容器。

尽管覆盖网络通常用于群服务，但您也可以将覆盖网络用于独立容器。这作为使用覆盖网络教程的一部分进行了介绍。

## 使用默认的桥接网络

在本例中，您alpine在同一个 Docker 主机上启动两个不同的容器，并进行一些测试以了解它们如何相互通信。您需要安装并运行 Docker。

1.  打开终端窗口。在你做任何其他事情之前列出当前的网络。如果您从未在此 Docker 守护程序上添加网络或初始化集群，那么您应该看到以下内容。
    您可能会看到不同的网络，但您至少应该看到这些（网络 ID 会有所不同）：

    ```console
    $ docker network ls

    NETWORK ID          NAME                DRIVER              SCOPE
    17e324f45964        bridge              bridge              local
    6ed54d316334        host                host                local
    7092879f2cc8        none                null                local
    ```

    The default `bridge` network is listed, along with `host` and `none`. The
    latter two are not fully-fledged networks, but are used to start a container
    connected directly to the Docker daemon host's networking stack, or to start
    a container with no network devices. **This tutorial will connect two
    containers to the `bridge` network.**

2.  启动两个alpine容器运行ash，这是 Alpine 的默认 shell 而不是bash. 
    这些-dit标志意味着启动容器是分离的（在后台）、交互的（能够输入）和带有 TTY（所以你可以看到输入和输出）。
    由于您将其分离启动，因此您不会立即连接到容器。相反，将打印容器的 ID。由于您没有指定任何 --network标志，容器连接到默认bridge网络。
    

    ```console
    $ docker run -dit --name alpine1 alpine ash

    $ docker run -dit --name alpine2 alpine ash
    ```

    检查两个容器是否实际启动：

    ```console
    $ docker container ls

    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
    602dbf1edc81        alpine              "ash"               4 seconds ago       Up 3 seconds                            alpine2
    da33b7aa74b0        alpine              "ash"               17 seconds ago      Up 16 seconds                           alpine1
    ```

3.  检查bridge网络以查看连接了哪些容器。

    ```console
    $ docker network inspect bridge

    [
        {
            "Name": "bridge",
            "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
            "Created": "2017-06-22T20:27:43.826654485Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "602dbf1edc81813304b6cf0a647e65333dc6fe6ee6ed572dc0f686a3307c6a2c": {
                    "Name": "alpine2",
                    "EndpointID": "03b6aafb7ca4d7e531e292901b43719c0e34cc7eef565b38a6bf84acf50f38cd",
                    "MacAddress": "02:42:ac:11:00:03",
                    "IPv4Address": "172.17.0.3/16",
                    "IPv6Address": ""
                },
                "da33b7aa74b0bf3bda3ebd502d404320ca112a268aafe05b4851d1e3312ed168": {
                    "Name": "alpine1",
                    "EndpointID": "46c044a645d6afc42ddd7857d19e9dcfb89ad790afb5c239a35ac0af5e8a5bc5",
                    "MacAddress": "02:42:ac:11:00:02",
                    "IPv4Address": "172.17.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {
                "com.docker.network.bridge.default_bridge": "true",
                "com.docker.network.bridge.enable_icc": "true",
                "com.docker.network.bridge.enable_ip_masquerade": "true",
                "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
                "com.docker.network.bridge.name": "docker0",
                "com.docker.network.driver.mtu": "1500"
            },
            "Labels": {}
        }
    ]
    ```

    在顶部附近，bridge列出了有关网络的信息，包括 Docker 主机和bridge 网络之间的网关的 IP 地址( 172.17.0.1)。在Containers键下，列出了每个连接的容器，以及有关其 IP 地址的信息（172.17.0.2for alpine1和172.17.0.3for alpine2）。

4.  容器在后台运行。使用docker attach 命令连接到alpine1.

    ```console
    $ docker attach alpine1

    / #
    ```

    提示变为 以#指示您是root容器内的用户。使用该ip addr show命令显示alpine1容器内的网络接口：

    ```console
    # ip addr show

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
        link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.2/16 scope global eth0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:acff:fe11:2/64 scope link
           valid_lft forever preferred_lft forever
    ```

    第一个接口是环回设备。暂时忽略它。请注意，第二个接口的 IP 地址与上一步中172.17.0.2显示的地址相同alpine1。

5.  从内部alpine1，确保您可以通过 ping 连接到 Internet google.com。该-c 2标志将命令限制为两次ping 尝试。

    ```console
    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.841 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.897 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.841/9.869/9.897 ms
    ```

6.  现在尝试 ping 第二个容器。首先，通过它的 IP 地址 ping 它， 172.17.0.3：

    ```console
    # ping -c 2 172.17.0.3

    PING 172.17.0.3 (172.17.0.3): 56 data bytes
    64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.086 ms
    64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.094 ms

    --- 172.17.0.3 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.086/0.090/0.094 ms
    ```

    这成功了。接下来，尝试alpine2按容器名称ping容器。这将失败。

    ```console
    # ping -c 2 alpine2

    ping: bad address 'alpine2'
    ```

7.  分离从alpine1没有通过使用分离的序列，停止它 CTRL+ p CTRL+ q（按住CTRL和类型p，随后q）。
    如果您愿意，请附加alpine2并重复步骤 4、5 和 6，替换alpine1为alpine2。

8.  停止并移除两个容器。

    ```console
    $ docker container stop alpine1 alpine2
    $ docker container rm alpine1 alpine2
    ```

请记住，`bridge`不建议将默认网络用于生产。要了解用户定义的桥接网络，请继续阅读[使用用户定义的桥接网络]。

## 使用用户定义的桥接网络

在这个例子中，我们再次启动两个`alpine`容器，但将它们附加到`alpine-net`我们已经创建的用户定义的网络上。这些容器根本没有连接到默认`bridge`网络。然后，我们启动`alpine`连接到`bridge`网络但未连接到 的第三个容器`alpine-net`，以及`alpine`连接到两个网络的第四个容器。

1.  创建`alpine-net`网络。您不需要该`--driver bridge`标志，因为它是默认设置，但此示例显示了如何指定它。

    ```console
    $ docker network create --driver bridge alpine-net
    ```

2.  列出 Docker 的网络：

    ```console
    $ docker network ls

    NETWORK ID          NAME                DRIVER              SCOPE
    e9261a8c9a19        alpine-net          bridge              local
    17e324f45964        bridge              bridge              local
    6ed54d316334        host                host                local
    7092879f2cc8        none                null                local
    ```

    检查`alpine-net`网络。这会显示它的 IP 地址以及没有容器连接到它的事实：

    ```console
    $ docker network inspect alpine-net

    [
        {
            "Name": "alpine-net",
            "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
            "Created": "2017-09-25T21:38:12.620046142Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {},
            "Options": {},
            "Labels": {}
        }
    ]
    ```

    请注意，此网络的网关是`172.18.0.1`，而不是默认网桥网络，其网关是`172.17.0.1`。您系统上的确切 IP 地址可能会有所不同。

3.  创建您的四个容器。注意`--network标志`。
    您只能在连接到一个网络`docker run`的命令，所以你需要使用 `docker network connect`之后连接`alpine4`到`bridge` 网络也是如此。

    ```console
    $ docker run -dit --name alpine1 --network alpine-net alpine ash

    $ docker run -dit --name alpine2 --network alpine-net alpine ash

    $ docker run -dit --name alpine3 alpine ash

    $ docker run -dit --name alpine4 --network alpine-net alpine ash

    $ docker network connect bridge alpine4
    ```

    验证所有容器都在运行：

    ```console
    $ docker container ls

    CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
    156849ccd902        alpine              "ash"               41 seconds ago       Up 41 seconds                           alpine4
    fa1340b8d83e        alpine              "ash"               51 seconds ago       Up 51 seconds                           alpine3
    a535d969081e        alpine              "ash"               About a minute ago   Up About a minute                       alpine2
    0a02c449a6e9        alpine              "ash"               About a minute ago   Up About a minute                       alpine1
    ```

4.  再次检查`bridge`网络和`alpine-net`网络：

    ```console
    $ docker network inspect bridge

    [
        {
            "Name": "bridge",
            "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
            "Created": "2017-06-22T20:27:43.826654485Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": null,
                "Config": [
                    {
                        "Subnet": "172.17.0.0/16",
                        "Gateway": "172.17.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "156849ccd902b812b7d17f05d2d81532ccebe5bf788c9a79de63e12bb92fc621": {
                    "Name": "alpine4",
                    "EndpointID": "7277c5183f0da5148b33d05f329371fce7befc5282d2619cfb23690b2adf467d",
                    "MacAddress": "02:42:ac:11:00:03",
                    "IPv4Address": "172.17.0.3/16",
                    "IPv6Address": ""
                },
                "fa1340b8d83eef5497166951184ad3691eb48678a3664608ec448a687b047c53": {
                    "Name": "alpine3",
                    "EndpointID": "5ae767367dcbebc712c02d49556285e888819d4da6b69d88cd1b0d52a83af95f",
                    "MacAddress": "02:42:ac:11:00:02",
                    "IPv4Address": "172.17.0.2/16",
                    "IPv6Address": ""
                }
            },
            "Options": {
                "com.docker.network.bridge.default_bridge": "true",
                "com.docker.network.bridge.enable_icc": "true",
                "com.docker.network.bridge.enable_ip_masquerade": "true",
                "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
                "com.docker.network.bridge.name": "docker0",
                "com.docker.network.driver.mtu": "1500"
            },
            "Labels": {}
        }
    ]
    ```

    容器`alpine3`并`alpine4`连接到`bridge`网络。

    ```console
    $ docker network inspect alpine-net

    [
        {
            "Name": "alpine-net",
            "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
            "Created": "2017-09-25T21:38:12.620046142Z",
            "Scope": "local",
            "Driver": "bridge",
            "EnableIPv6": false,
            "IPAM": {
                "Driver": "default",
                "Options": {},
                "Config": [
                    {
                        "Subnet": "172.18.0.0/16",
                        "Gateway": "172.18.0.1"
                    }
                ]
            },
            "Internal": false,
            "Attachable": false,
            "Containers": {
                "0a02c449a6e9a15113c51ab2681d72749548fb9f78fae4493e3b2e4e74199c4a": {
                    "Name": "alpine1",
                    "EndpointID": "c83621678eff9628f4e2d52baf82c49f974c36c05cba152db4c131e8e7a64673",
                    "MacAddress": "02:42:ac:12:00:02",
                    "IPv4Address": "172.18.0.2/16",
                    "IPv6Address": ""
                },
                "156849ccd902b812b7d17f05d2d81532ccebe5bf788c9a79de63e12bb92fc621": {
                    "Name": "alpine4",
                    "EndpointID": "058bc6a5e9272b532ef9a6ea6d7f3db4c37527ae2625d1cd1421580fd0731954",
                    "MacAddress": "02:42:ac:12:00:04",
                    "IPv4Address": "172.18.0.4/16",
                    "IPv6Address": ""
                },
                "a535d969081e003a149be8917631215616d9401edcb4d35d53f00e75ea1db653": {
                    "Name": "alpine2",
                    "EndpointID": "198f3141ccf2e7dba67bce358d7b71a07c5488e3867d8b7ad55a4c695ebb8740",
                    "MacAddress": "02:42:ac:12:00:03",
                    "IPv4Address": "172.18.0.3/16",
                    "IPv6Address": ""
                }
            },
            "Options": {},
            "Labels": {}
        }
    ]
    ```

    容器`alpine1`、`alpine2`和`alpine4`连接到 `alpine-net`网络。

5.  在用户定义的网络上，如`alpine-net`，容器不仅可以通过 IP 地址进行通信，还可以将容器名称解析为 IP 地址。
    此功能称为自动服务发现。
    让我们连接`alpine1`并测试一下。`alpine1`应该能够解析 `alpine2`和`alpine4`（和`alpine1`本身）​到 IP 地址。

    ```console
    $ docker container attach alpine1

    # ping -c 2 alpine2

    PING alpine2 (172.18.0.3): 56 data bytes
    64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.085 ms
    64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.090 ms

    --- alpine2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.085/0.087/0.090 ms

    # ping -c 2 alpine4

    PING alpine4 (172.18.0.4): 56 data bytes
    64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.076 ms
    64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.091 ms

    --- alpine4 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.076/0.083/0.091 ms

    # ping -c 2 alpine1

    PING alpine1 (172.18.0.2): 56 data bytes
    64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.026 ms
    64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.054 ms

    --- alpine1 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.026/0.040/0.054 ms
    ```

6.  从`alpine1`，您应该根本无法连接到`alpine3`，因为它不在`alpine-net`网络上。

    ```console
    # ping -c 2 alpine3

    ping: bad address 'alpine3'
    ```

    不仅如此，但你无法连接到`alpine3`来自`alpine1`它的IP地址要么。
    查看网络的`docker network inspect`输出 `bridge`并找到`alpine3`的 IP 地址：172.17.0.2尝试 ping 它。

    ```console
    # ping -c 2 172.17.0.2

    PING 172.17.0.2 (172.17.0.2): 56 data bytes

    --- 172.17.0.2 ping statistics ---
    2 packets transmitted, 0 packets received, 100% packet loss
    ```

    用分离序列从`alpine1`分离， `CTRL` + `p` `CTRL` + `q` (按住`CTRL`，然后输入`p`和`q`).

7.  请记住，`alpine4`它同时连接到默认`bridge`网络和`alpine-net`. 它应该能够到达所有其他容器。
    但是，您需要`alpine3`通过其 IP 地址进行寻址。附加到它并运行测试。

    ```console
    $ docker container attach alpine4

    # ping -c 2 alpine1

    PING alpine1 (172.18.0.2): 56 data bytes
    64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.074 ms
    64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.082 ms

    --- alpine1 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.074/0.078/0.082 ms

    # ping -c 2 alpine2

    PING alpine2 (172.18.0.3): 56 data bytes
    64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.075 ms
    64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.080 ms

    --- alpine2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.075/0.077/0.080 ms

    # ping -c 2 alpine3
    ping: bad address 'alpine3'

    # ping -c 2 172.17.0.2

    PING 172.17.0.2 (172.17.0.2): 56 data bytes
    64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.089 ms
    64 bytes from 172.17.0.2: seq=1 ttl=64 time=0.075 ms

    --- 172.17.0.2 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.075/0.082/0.089 ms

    # ping -c 2 alpine4

    PING alpine4 (172.18.0.4): 56 data bytes
    64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.033 ms
    64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.064 ms

    --- alpine4 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 0.033/0.048/0.064 ms
    ```

8.  作为最终测试，请确保您的容器都可以通过 ping 连接到 Internet `google.com`。
    你已经附加了 `alpine4` 所以从那里开始尝试。接下来，断开`alpine4`并连接到`alpine3` （仅连接到`bridge`网络）并重试。
    最后，连接到`alpine1`（仅连接到`alpine-net`网络）并重试。

    ```console
    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.778 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.634 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.634/9.706/9.778 ms

    CTRL+p CTRL+q

    $ docker container attach alpine3

    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.706 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.851 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.706/9.778/9.851 ms

    CTRL+p CTRL+q

    $ docker container attach alpine1

    # ping -c 2 google.com

    PING google.com (172.217.3.174): 56 data bytes
    64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.606 ms
    64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.603 ms

    --- google.com ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max = 9.603/9.604/9.606 ms

    CTRL+p CTRL+q
    ```

9.  停止并移除所有容器和 `alpine-net` 网络。

    ```
    $ docker container stop alpine1 alpine2 alpine3 alpine4

    $ docker container rm alpine1 alpine2 alpine3 alpine4

    $ docker network rm alpine-net
    ```


## 其他网络资源

现在您已经完成了独立容器的网络教程，您可能想要浏览这些其他网络教程：

- [Host networking tutorial](network-tutorial-host.md)
- [Overlay networking tutorial](network-tutorial-overlay.md)
- [Macvlan networking tutorial](network-tutorial-macvlan.md)

