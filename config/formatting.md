---
description: CLI and log output formatting reference
keywords: format, formatting, output, templates, log
title: 格式化命令和日志输出
redirect_from:
- /engine/admin/formatting/
---

Docker 使用Go 模板，您可以使用它来操作某些命令和日志驱动程序的输出格式。

Docker 提供了一组基本函数来操作模板元素。所有这些示例都使用该docker inspect命令，但许多其他 CLI 命令都有一个--format标志，并且许多 CLI 命令参考包括自定义输出格式的示例。

>**Note**
>
> 使用该--format标志时，您需要观察您的 shell 环境。在 Posix shell 中，您可以使用单引号运行以下命令：
>
> {% raw %}
> ```console
> $ docker inspect --format '{{join .Args " , "}}'
> ```
> {% endraw %}
>
> 否则，在 Windows shell（例如 PowerShell）中，您需要使用单引号，但在 params 中转义双引号，如下所示：
>
> {% raw %}
> ```console
> $ docker inspect --format '{{join .Args \" , \"}}'
> ```
> {% endraw %}
>
{:.important}

## join

join连接字符串列表以创建单个字符串。它在列表中的每个元素之间放置一个分隔符。

{% raw %}
```console
$ docker inspect --format '{{join .Args " , "}}' container
```
{% endraw %}

## table

table 指定要查看其输出的字段。

{% raw %}
```console
$ docker image list --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}\t{{.Size}}"
```
{% endraw %}

## json

json 将元素编码为 json 字符串。

{% raw %}
```console
$ docker inspect --format '{{json .Mounts}}' container
```
{% endraw %}

## lower

lower 将字符串转换为其小写表示。

{% raw %}
```console
$ docker inspect --format "{{lower .Name}}" container
```
{% endraw %}

## split

split 将字符串切片为由分隔符分隔的字符串列表。

{% raw %}
```console
$ docker inspect --format '{{split .Image ":"}}'
```
{% endraw %}

## title

title 将字符串的第一个字符大写。

{% raw %}
```console
$ docker inspect --format "{{title .Name}}" container
```
{% endraw %}

## upper

upper 将字符串转换为其大写表示形式。

{% raw %}
```console
$ docker inspect --format "{{upper .Name}}" container
```
{% endraw %}


## println

println 在新行上打印每个值。

{% raw %}
```console
$ docker inspect --format='{{range .NetworkSettings.Networks}}{{println .IPAddress}}{{end}}' container
```
{% endraw %}

# Hint

要找出可以打印哪些数据，请将所有内容显示为 json：

{% raw %} 
```console
$ docker container ls --format='{{json .}}'
```
{% endraw %} 
