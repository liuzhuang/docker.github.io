---
description: How to run more than one process in a container
keywords: docker, supervisor, process management
redirect_from:
- /articles/using_supervisord/
- /engine/admin/multi-service_container/
- /engine/admin/using_supervisord/
- /engine/articles/using_supervisord/
title: 在一个容器中运行多个服务
---

容器的主进程是 `Dockerfile` 结尾的 `ENTRYPOINT` 或 `CMD`。
通常建议您通过为每个容器使用一个服务来分隔不同的关注领域。
该服务可能会分成多个进程（例如，Apache Web 服务器启动多个工作进程）。
拥有多个进程是可以的，但是为了从 Docker 中获得最大收益，请避免一个容器负责整个应用程序的多个方面。
您可以使用用户定义的网络和共享存储卷连接多个容器。


容器的主进程负责管理它启动的所有进程。
在某些情况下，主进程没有设计好，并且在容器退出时不能优雅地处理 “reaping”（stopping）子进程。
如果您的进程属于此类别，则可以在运行容器时使用 `--init` 选项。
`--init` 标志将一个微小的 `init-process` 作为主进程插入到容器中，并在容器退出时处理所有进程的收割。
以这种方式处理此类进程优于使用成熟的 init 进程，例如 `sysvinit`, `upstart`, 或 `systemd` 处理容器内的进程生命周期。


如果您需要在一个容器中运行多个服务，您可以通过几种不同的方式来实现。

- 将所有命令放在一个包装脚本中，并附上测试和调试信息。
  将包装脚本作为您的 `CMD`. 这是一个非常幼稚的例子。
  首先，包装脚本：

  ```bash
  #!/bin/bash

  # Start the first process
  ./my_first_process &
  
  # Start the second process
  ./my_second_process &
  
  # Wait for any process to exit
  wait -n
  
  # Exit with status of process that exited first
  exit $?
  ```

  接下来，Dockerfile：

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM ubuntu:latest
  COPY my_first_process my_first_process
  COPY my_second_process my_second_process
  COPY my_wrapper_script.sh my_wrapper_script.sh
  CMD ./my_wrapper_script.sh
  ```

- 如果您有一个需要首先启动并保持运行的主进程，但您暂时需要运行一些其他进程（可能与主进程交互），
  那么您可以使用 bash 的作业控制来促进这一点。
  首先，包装脚本：

  ```bash
  #!/bin/bash
  
  # turn on bash's job control
  set -m
  
  # Start the primary process and put it in the background
  ./my_main_process &
  
  # Start the helper process
  ./my_helper_process
  
  # the my_helper_process might need to know how to wait on the
  # primary process to start before it does its work and returns
  
  
  # now we bring the primary process back into the foreground
  # and leave it there
  fg %1
  ```

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM ubuntu:latest
  COPY my_main_process my_main_process
  COPY my_helper_process my_helper_process
  COPY my_wrapper_script.sh my_wrapper_script.sh
  CMD ./my_wrapper_script.sh
  ```

- 使用像 supervisord. 这是一种中等重量级的方法，需要您在镜像中打包 `supervisord` 及其配置（或基于包含 `supervisord` 的镜像）及其管理的不同应用程序。
  接下来启动  `supervisord`，它会为您管理您的流程。
  下面是使用这种方法，它假定预先编写的一个例子 Dockerfile `supervisord.conf`, `my_first_process` 和 `my_second_process`

  ```dockerfile
  # syntax=docker/dockerfile:1
  FROM ubuntu:latest
  RUN apt-get update && apt-get install -y supervisor
  RUN mkdir -p /var/log/supervisor
  COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
  COPY my_first_process my_first_process
  COPY my_second_process my_second_process
  CMD ["/usr/bin/supervisord"]
  ```