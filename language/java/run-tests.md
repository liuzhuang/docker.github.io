---
title: "运行你的测试"
keywords: Java, build, test
description: How to build and run your Tests
---

{% include_relative nav.html selected="4" %}

## 先决条件

[Use your container for development](develop.md).

## 简介

测试是现代软件开发的重要组成部分。
测试对于不同的开发团队来说意义重大。
有单元测试（unit tests）、集成测试（integration tests）和端到端测试（end-to-end testing）。
在本指南中，我们将了解在 Docker 中运行单元测试（unit tests）。

## 为了运行测试，重构 Dockerfile

**Spring Pet Clinic** 已经在测试目录中 `src/test/java/org/springframework/samples/petclinic` 定义了单元测试。
您只需要更新您的 JaCoCo 的 `pom.xml` 确保您的单元测试使用 JDK v15 或更高版本 `<jacoco.version>0.8.6</jacoco.version>`，
因此我们可以使用以下 Docker 命令启动容器并运行测试：

```console
$ docker run -it --rm --name springboot-test java-docker ./mvnw test
...
[INFO] Results:
[INFO]
[WARNING] Tests run: 40, Failures: 0, Errors: 0, Skipped: 1
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:49 min
```

### 用于测试的多阶段 Dockerfile

让我们来看看如何将 testing commands 加入到我们的 Dockerfile。
下面是一个多阶段 Dockerfile，我们将使用它来构建我们的 “生产环境镜像” 和 “测试环境镜像”。
将突出显示的行添加到 Dockerfile

```dockerfile
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13 as base

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src

FROM base as test
CMD ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

我们首先为 `FROM openjdk:16-alpine3.13` 语句添加了一个标签。这允许我们在其他构建阶段中引用这个构建阶段。
接下来，我们添加了一个标签为 `test` 的新构建阶段。我们将使用这个阶段来运行我们的 tests。

现在让我们重新构建镜像并运行我们的 tests。
我们将像上面一样运行 `docker build` 命令，但这次我们将添加 `--target test` 标志，以便我们运行 test 构建阶段。

```console
$ docker build -t java-docker --target test .
[+] Building 0.7s (6/6) FINISHED
...
 => => writing image sha256:967ac80cb7799a5d12a4bdfc67c37b5a6533c6e418c903907d3e86b7d4ebf89a
 => => naming to docker.io/library/java-docker
```

现在我们的测试镜像已经构建好了，我们将它作为一个容器运行，看看我们的测试是否通过。

```console
$ docker run -it --rm --name springboot-test java-docker
[INFO] Scanning for projects...
[INFO]
[INFO] ------------< org.springframework.samples:spring-petclinic >------------
[INFO] Building petclinic 2.4.2
...

[INFO] Results:
[INFO]
[WARNING] Tests run: 40, Failures: 0, Errors: 0, Skipped: 1
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:22 min
```

构建的输出被截断了，但您可以看到 Maven test runner 运行成功并且我们所有的测试都通过了。

这很棒。
但是，我们必须运行两个 Docker 命令来构建和运行我们的测试。
我们稍微改进一下，通过在测试阶段使用 `RUN` 语句而不是 `CMD` 语句。
`CMD` 语句在构建镜像期间不会被执行，而是在您在容器中运行镜像时执行。
当我们使用 `RUN` 语句时，我们的 tests 在构建镜像时运行，并在构建失败时停止构建。

将下面突出显示的行更新您的 Dockerfile。

```dockerfile
# syntax=docker/dockerfile:1

FROM openjdk:16-alpine3.13 as base

WORKDIR /app

COPY .mvn/ .mvn
COPY mvnw pom.xml ./
RUN ./mvnw dependency:go-offline
COPY src ./src

FROM base as test
RUN ["./mvnw", "test"]

FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]

FROM base as build
RUN ./mvnw package

FROM openjdk:11-jre-slim as production
EXPOSE 8080

COPY --from=build /app/target/spring-petclinic-*.jar /spring-petclinic.jar

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-petclinic.jar"]
```

现在，要运行我们的 tests，我们只需要运行和上面一样的 `docker build` 命令。

```console
$ docker build -t java-docker --target test .
[+] Building 27.6s (11/12)
 => CACHED [base 3/6] COPY .mvn/ .mvn
 => CACHED [base 4/6] COPY mvnw pom.xml ./
 => CACHED [base 5/6] RUN ./mvnw dependency:go-offline
 => CACHED [base 6/6] COPY src ./src
 => [test 1/1] RUN ["./mvnw", "test"]
 => exporting to image
 => => exporting layers
=> => writing image sha256:10cb585a7f289a04539e95d583ae97bcf8725959a6bd32c2f5632d0e7c1d16a0
=> => naming to docker.io/library/java-docker
```

为简单起见，构建的输出被截断，但您可以看到我们的 tests 成功运行并通过。
让我们中断其中一个测试并观察当我们的测试失败时的输出。

打开 `src/test/java/org/springframework/samples/petclinic/model/ValidatorTests.java` 文件并将 **第 57 行** 更改为以下内容。

```shell
55   ConstraintViolation<Person> violation = constraintViolations.iterator().next();
56   assertThat(violation.getPropertyPath().toString()).isEqualTo("firstName");
57   assertThat(violation.getMessage()).isEqualTo("must be empty");
58 }
```

现在，运行 `docker build` 上面的命令并观察到构建失败和失败的测试信息打印到控制台。

```console
$ docker build -t java-docker --target test .
 => [base 6/6] COPY src ./src
 => ERROR [test 1/1] RUN ["./mvnw", "test"]
...
------
executor failed running [./mvnw test]: exit code: 1
```

### 用于开发的多阶段 Dockerfile

新版本的 Dockerfile 会生成一个可用于生产的镜像（production image），但是你注意到的，您还有一个专门的步骤来生成开发容器（development container）。

```dockerfile
FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]
```

现在我们更新 `docker-compose.dev.yml` 使用特定的 target 来构建 `petclinic` 服务并删除 `command` 定义，如下所示：

```dockerfile
services:
 petclinic:
   build:
     context: .
     target: development
   ports:
     - 8000:8000
     - 8080:8080
   environment:
     - SERVER_PORT=8080
     - MYSQL_URL=jdbc:mysql://mysqlserver/petclinic
   volumes:
     - ./:/app
```

现在，让我们运行 Compose 应用程序。您现在应该看到该应用程序的行为与以前一样，您仍然可以对其进行调试。

```console
$ docker-compose -f docker-compose.dev.yml up --build
```

## 下一步

在本模块中，我们研究了运行测试作为 Docker 镜像构建过程的一部分。

在下一个模块中，我们将了解如何使用 GitHub Actions 设置 CI/CD 管道。看：

[Configure CI/CD](configure-ci-cd.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.

<br />
