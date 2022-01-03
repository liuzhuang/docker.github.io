---
title: Logentries 日志驱动
description: Describes how to use the logentries logging driver.
keywords: logentries, docker, logging, driver
redirect_from:
- /engine/admin/logging/logentries/
---

`logentries` 日志驱动将发送容器日志发送到 [Logentries](https://logentries.com/) 服务器。

## 用法

`--log-opt`

 - `logentries-token`: 设置 logentries token
 - `line-only`: 只发送原始负载 payload


设置 `logentries` 驱动

```console
$ dockerd --log-driver=logentries
```

设置 `logentries` 驱动

```console
$ docker run --log-driver=logentries ...
```

在使用此日志驱动程序之前，您需要在 Logentries Web 界面中创建一个新的日志集并将该日志集的令牌传递给 Docker：

```console
$ docker run --log-driver=logentries --log-opt logentries-token=abcd1234-12ab-34cd-5678-0123456789ab
```

## 可选项

使用 `--log-opt NAME=VALUE` 添加额外的参数

### logentries-token

提供日志令牌

```console
$ docker run --log-driver=logentries --log-opt logentries-token=abcd1234-12ab-34cd-5678-0123456789ab
```

### line-only

发送日志消息还是原始日志行

```console
$ docker run --log-driver=logentries --log-opt logentries-token=abcd1234-12ab-34cd-5678-0123456789ab --log-opt line-only=true
```
