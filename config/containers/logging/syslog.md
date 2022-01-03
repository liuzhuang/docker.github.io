---
description: Describes how to use the syslog logging driver.
keywords: syslog, docker, logging, driver
redirect_from:
- /engine/reference/logging/syslog/
- /engine/admin/logging/syslog/
title: Syslog 日志驱动
---

`syslog` 日志驱动将日志路由到 `syslog` 服务器。该 `syslog` 协议使用原始字符串作为日志消息并支持有限的元数据集。系统日志消息必须以特定方式格式化才能有效。从有效消息中，接收者可以提取以下信息：

- 优先级：日志级别，例如 `debug`, `warning`, `error`, `info`。
- 时间戳：事件发生的时间。
- 主机名：事件发生的地方。
- 设施：哪个子系统记录了消息，例如 `mail` 或 `kernel`。
- 进程名称和进程 ID (PID)：生成日志的进程的名称和 ID。

该格式在 [RFC 5424](https://tools.ietf.org/html/rfc5424) 中定义，Docker 的 syslog 驱动程序通过以下方式实现 [ABNF reference](https://tools.ietf.org/html/rfc5424#section-6)：

```none
                TIMESTAMP SP HOSTNAME SP APP-NAME SP PROCID SP MSGID
                    +          +             +           |        +
                    |          |             |           |        |
                    |          |             |           |        |
       +------------+          +----+        |           +----+   +---------+
       v                            v        v                v             v
2017-04-01T17:41:05.616647+08:00 a.vm {taskid:aa,version:} 1787791 {taskid:aa,version:}
```

## 用法

```json
{
  "log-driver": "syslog",
  "log-opts": {
    "syslog-address": "udp://1.2.3.4:1111"
  }
}
```

```console
$ docker run \
      --log-driver syslog --log-opt syslog-address=udp://1.2.3.4:1111 \
      alpine echo hello world
```

## 可选项

| Option | Description | Example value |
|:-------|:------------|:-----------------|
| `syslog-address`         | The address of an external `syslog` server. The URI specifier may be `[tcp|udp|tcp+tls]://host:port`, `unix://path`, or `unixgram://path`. If the transport is `tcp`, `udp`, or `tcp+tls`, the default port is `514`.                                                                                            | `--log-opt syslog-address=tcp+tls://192.168.1.3:514`, `--log-opt syslog-address=unix:///tmp/syslog.sock` |
| `syslog-facility`        | The `syslog` facility to use. Can be the number or name for any valid `syslog` facility. See the [syslog documentation](https://tools.ietf.org/html/rfc5424#section-6.2.1).                                                                                                                                      | `--log-opt syslog-facility=daemon`                                                                       |
| `syslog-tls-ca-cert`     | The absolute path to the trust certificates signed by the CA. **Ignored if the address protocol is not `tcp+tls`.**                                                                                                                                                                                              | `--log-opt syslog-tls-ca-cert=/etc/ca-certificates/custom/ca.pem`                                        |
| `syslog-tls-cert`        | The absolute path to the TLS certificate file. **Ignored if the address protocol is not `tcp+tls`**.                                                                                                                                                                                                             | `--log-opt syslog-tls-cert=/etc/ca-certificates/custom/cert.pem`                                         |
| `syslog-tls-key`         | The absolute path to the TLS key file. **Ignored if the address protocol is not `tcp+tls`.**                                                                                                                                                                                                                     | `--log-opt syslog-tls-key=/etc/ca-certificates/custom/key.pem`                                           |
| `syslog-tls-skip-verify` | If set to `true`, TLS verification is skipped when connecting to the `syslog` daemon. Defaults to `false`. **Ignored if the address protocol is not `tcp+tls`.**                                                                                                                                                 | `--log-opt syslog-tls-skip-verify=true`                                                                  |
| `tag`                    | A string that is appended to the `APP-NAME` in the `syslog` message. By default, Docker uses the first 12 characters of the container ID to tag log messages. Refer to the [log tag option documentation](log_tags.md) for customizing the log tag format.                                                       | `--log-opt tag=mailer`                                                                                   |
| `syslog-format`          | The `syslog` message format to use. If not specified the local UNIX syslog format is used, without a specified hostname. Specify `rfc3164` for the RFC-3164 compatible format, `rfc5424` for RFC-5424 compatible format, or `rfc5424micro` for RFC-5424 compatible format with microsecond timestamp resolution. | `--log-opt syslog-format=rfc5424micro`                                                                   |
| `labels`                 | Applies when starting the Docker daemon. A comma-separated list of logging-related labels this daemon accepts. Used for advanced [log tag options](log_tags.md).                                                                                                                                                 | `--log-opt labels=production_status,geo`                                                                 |
| `labels-regex`           | Applies when starting the Docker daemon. Similar to and compatible with `labels`. A regular expression to match logging-related labels. Used for advanced [log tag options](log_tags.md).                                                                                                                        | `--log-opt labels-regex=^(production_status\|geo)`                                                       |
| `env`                    | Applies when starting the Docker daemon. A comma-separated list of logging-related environment variables this daemon accepts. Used for advanced [log tag options](log_tags.md).                                                                                                                                  | `--log-opt env=os,customer`                                                                              |
| `env-regex`              | Applies when starting the Docker daemon. Similar to and compatible with `env`. A regular expression to match logging-related environment variables. Used for advanced [log tag options](log_tags.md).                                                                                                            | `--log-opt env-regex=^(os\|customer)`                                                                    |
