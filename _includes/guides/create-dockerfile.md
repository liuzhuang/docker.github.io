
Dockerfile 是一个文本文档，其中包含组装 Docker 镜像的说明。当我们通过执行 `docker build` 命令告诉 Docker 构建我们的镜像时，Docker 会读取这些指令，执行它们，并因此创建一个 Docker 镜像。


让我们来看看为我们的应用程序创建 Dockerfile 的过程。在项目的根目录中，创建一个名为`Dockerfile`的文件并在文本编辑器中打开该文件。


> **Dockerfile 的名称是什么？**
>
> 默认文件名是 `Dockerfile`（没有扩展名）。使用默认名称允许您运行 `docker build` 命令而无需指定其他命令标志。
>
> 某些项目可能需要不同的 Dockerfiles 用于特定目的。
> 一个常见的约定是将这些命名为 `Dockerfile.<something>` 或 `<something>.Dockerfile`。
> 通过命令上 `--file`（或 `-f`）选项使用。请参阅 ["Specify a Dockerfile" section](/engine/reference/commandline/build/#specify-a-dockerfile--f)
>
> 我们建议使用 `Dockerfile`，我们将在本指南中的大多数示例中使用默认值。

添加到 Dockerfile 的第一行是 [`# syntax` parser directive](/engine/reference/builder/#syntax)。
虽然是可选的，但该指令指示 Docker 构建器在解析 Dockerfile 时使用什么语法，并允许启用 BuildKit 的旧 Docker 版本在开始构建之前升级解析器。
[Parser directives](/engine/reference/builder/#parser-directives) 必须出现在 Dockerfile 中的任何其他注释、空格或 Dockerfile 指令之前，并且应该是 Dockerfiles 的第一行。

```dockerfile
# syntax=docker/dockerfile:1
```

我们建议使用 `docker/dockerfile:1`，它始终指向 version 1  语法的最新版本。BuildKit 在构建之前会自动检查语法的更新，确保您使用的是最新版本。