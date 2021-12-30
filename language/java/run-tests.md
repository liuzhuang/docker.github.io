---
title: "Run your tests"
keywords: Java, build, test
description: How to build and run your Tests
---

{% include_relative nav.html selected="4" %}

## 先决条件

[Use your container for development](develop.md).

## 简介

测试是现代软件开发的重要组成部分。测试对于不同的开发团队来说意义重大。有单元测试、集成测试和端到端测试。在本指南中，我们将了解在 Docker 中运行单元测试。

## 重构 Dockerfile 为了运行测试

在 **Spring Pet Clinic** 的源代码已经在 `src/test/java/org/springframework/samples/petclinic` 测试目录中定义了单元测试。您只需要更新您的 JaCoCo 的 `pom.xml` 确保您的测试使用 JDK v15 或更高版本 <jacoco.version>0.8.6</jacoco.version>，因此我们可以使用以下 Docker 命令来启动容器并运行测试：


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

### Multi-stage Dockerfile 为了测试

让我们来看看将测试命令拉入我们的 Dockerfile。下面是一个多阶段 Dockerfile，我们将使用它来构建我们的生产镜像和我们的测试镜像。将突出显示的行添加到 Dockerfile

Let’s take a look at pulling the testing commands into our Dockerfile. 
Below is a multi-stage Dockerfile that we will use to build our production image and our test image. 
Add the highlighted lines to your Dockerfile

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

我们首先为 `FROM openjdk:16-alpine3.13` 语句添加一个标签。这允许我们在其他构建阶段中引用这个构建阶段。
接下来，我们添加了一个标记为 `test` 的新构建阶段。我们将使用这个阶段来运行我们的测试。
We first add a label to the `FROM openjdk:16-alpine3.13` statement. This allows us to refer to this build stage in other build stages. Next, we added a new build stage labeled `test`. We'll use this stage for running our tests.

现在让我们重建我们的镜像并运行我们的测试。我们将`docker build` 像上面一样运行命令，但这次我们将添加 `--target test` 标志，以便我们专门运行测试构建阶段。
Now let’s rebuild our image and run our tests. We will run the `docker build` command as above, but this time we will add the `--target test` flag so that we specifically run the test build stage.

```console
$ docker build -t java-docker --target test .
[+] Building 0.7s (6/6) FINISHED
...
 => => writing image sha256:967ac80cb7799a5d12a4bdfc67c37b5a6533c6e418c903907d3e86b7d4ebf89a
 => => naming to docker.io/library/java-docker
```

现在我们的测试镜像已经构建好了，我们可以将它作为一个容器运行，看看我们的测试是否通过。
Now that our test image is built, we can run it as a container and see if our tests pass.

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

构建输出被截断，但您可以看到 Maven 测试运行器成功并且我们所有的测试都通过了。
The build output is truncated, but you can see that the Maven test runner was successful and all our tests passed.

这很棒。但是，我们必须运行两个 Docker 命令来构建和运行我们的测试。我们可以通过在测试阶段使用RUN语句而不是语句来稍微改进这一点CMD。该CMD语句不会在构建镜像期间执行，而是在您在容器中运行镜像时执行。使用该RUN语句时，我们的测试在构建镜像时运行，并在构建失败时停止构建。
This is great. However, we'll have to run two Docker commands to build and run our tests. We can improve this slightly by using a `RUN` statement instead of the `CMD` statement in the test stage. The `CMD` statement is not executed during the building of the image, but is executed when you run the image in a container. When using the `RUN` statement, our tests run when the building the image, and stop the build when they fail.

使用下面突出显示的行更新您的 Dockerfile。
Update your Dockerfile with the highlighted line below.

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

现在，要运行我们的测试，我们只需要运行docker build上面的命令。
Now, to run our tests, we just need to run the `docker build` command as above.

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

为简单起见，构建输出被截断，但您可以看到我们的测试成功运行并通过。当我们的测试失败时，让我们中断其中一个测试并观察输出。
The build output is truncated for simplicity, but you can see that our tests ran succesfully and passed. Let’s break one of the tests and observe the output when our tests fail.

打开src/test/java/org/springframework/samples/petclinic/model/ValidatorTests.java文件并将第 57 行更改为以下内容。
Open the `src/test/java/org/springframework/samples/petclinic/model/ValidatorTests.java` file and change **line 57** to the following.

```shell
55   ConstraintViolation<Person> violation = constraintViolations.iterator().next();
56   assertThat(violation.getPropertyPath().toString()).isEqualTo("firstName");
57   assertThat(violation.getMessage()).isEqualTo("must be empty");
58 }
```

现在，docker build从上面运行命令并观察构建失败并且失败的测试信息打印到控制台。
Now, run the `docker build` command from above and observe that the build fails and the failing testing information is printed to the console.

```console
$ docker build -t java-docker --target test .
 => [base 6/6] COPY src ./src
 => ERROR [test 1/1] RUN ["./mvnw", "test"]
...
------
executor failed running [./mvnw test]: exit code: 1
```

### 用于开发的多阶段 Dockerfile

新版本的 Dockerfile 会生成可用于生产的最终映像，但正如您所注意到的，您还有一个专门的步骤来生成开发容器。
The new version of the Dockerfile produces a final image which is ready for production, but as you can notice, you also have a dedicated step to produce a development container.

```dockerfile
FROM base as development
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql", "-Dspring-boot.run.jvmArguments='-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000'"]
```

我们现在可以更新我们的docker-compose.dev.yml以使用此特定目标来构建petclinic服务并删除command定义，如下所示：
We can now update of our `docker-compose.dev.yml` to use this specific target to build the `petclinic` service and remove the `command` definition as follows:

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
Now, let's run the Compose application. You should now see that application behaves as previously and you can still debug it.

```console
$ docker-compose -f docker-compose.dev.yml up --build
```

## 下一步

在本模块中，我们研究了运行测试作为 Docker 镜像构建过程的一部分。
In this module, we took a look at running tests as part of our Docker image build process.

在下一个模块中，我们将了解如何使用 GitHub Actions 设置 CI/CD 管道。看：
In the next module, we’ll take a look at how to set up a CI/CD pipeline using GitHub Actions. See:

[Configure CI/CD](configure-ci-cd.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.

<br />
