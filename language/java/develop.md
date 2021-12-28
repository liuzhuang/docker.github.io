---
title: "使用容器进行开发"
keywords: Java, local, development, run,
description: Learn how to develop your application locally.
---

{% include_relative nav.html selected="3" %}

## 先决条件

[Run your image as a container](run-containers.md).

## 简介

在本模块中，我们将逐步为我们在前面模块中构建的应用程序设置本地开发环境。我们将使用 Docker 来构建我们的图像，并使用 Docker Compose 使一切变得更加容易。

## 在容器中运行数据库

首先，我们将看看在容器中运行数据库，以及我们如何使用卷和网络来持久化我们的数据并允许我们的应用程序与数据库对话。然后我们将把所有东西放在一个 Compose 文件中，它允许我们用一个命令设置和运行本地开发环境。最后，我们将看看将调试器连接到我们在容器内运行的应用程序。

First, we’ll take a look at running a database in a container and how we use volumes and networking to persist our data and allow our application to talk with the database. Then we’ll pull everything together into a Compose file which allows us to setup and run a local development environment with one command. Finally, we’ll take a look at connecting a debugger to our application running inside a container.

无需下载 MySQL、安装、配置，然后将 MySQL 数据库作为服务运行，我们可以使用 MySQL 的 Docker 官方镜像并在容器中运行它。
Instead of downloading MySQL, installing, configuring, and then running the MySQL database as a service, we can use the Docker Official Image for MySQL and run it in a container.

在容器中运行 MySQL 之前，我们将创建几个卷，Docker 可以管理这些卷来存储我们的持久数据和配置。让我们使用 Docker 提供的托管卷功能，而不是使用绑定挂载。您可以在我们的文档中阅读有关使用卷的所有信息。
Before we run MySQL in a container, we'll create a couple of volumes that Docker can manage to store our persistent data and configuration. Let’s use the managed volumes feature that Docker provides instead of using bind mounts. You can read all about [Using volumes](../../storage/volumes.md) in our documentation.


现在让我们创建我们的卷。我们将为数据创建一个，为 MySQL 的配置创建一个。
Let’s create our volumes now. We’ll create one for the data and one for configuration of MySQL.

```console
$ docker volume create mysql_data
$ docker volume create mysql_config
```

现在我们将创建一个网络，我们的应用程序和数据库将使用该网络相互通信。该网络称为用户定义的桥接网络，它为我们提供了一个很好的 DNS 查找服务，我们可以在创建连接字符串时使用它。
Now we’ll create a network that our application and database will use to talk to each other. The network is called a user-defined bridge network and gives us a nice DNS lookup service which we can use when creating our connection string.

```console
$ docker network create mysqlnet
```

现在，让我们在容器中运行 MySQL 并附加到我们上面创建的卷和网络。Docker 从 Hub 拉取镜像并在本地运行。
Now, let's run MySQL in a container and attach to the volumes and network we created above. Docker pulls the image from Hub and runs it locally.

```console
$ docker run -it --rm -d -v mysql_data:/var/lib/mysql \
-v mysql_config:/etc/mysql/conf.d \
--network mysqlnet \
--name mysqlserver \
-e MYSQL_USER=petclinic -e MYSQL_PASSWORD=petclinic \
-e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=petclinic \
-p 3306:3306 mysql:8.0.23
```

好的，现在我们有一个正在运行的 MySQL，让我们更新我们的 Dockerfile 以激活应用程序中定义的 MySQL Spring 配置文件，并从内存中的 H2 数据库切换到我们刚刚创建的 MySQL 服务器。
Okay, now that we have a running MySQL, let’s update our Dockerfile to activate the MySQL Spring profile defined in the application and switch from an in-memory H2 database to the MySQL server we just created.

我们只需要添加 MySQL 配置文件作为CMD定义的参数。
We only need to add the MySQL profile as an argument to the `CMD` definition.

```dockerfile
CMD ["./mvnw", "spring-boot:run", "-Dspring-boot.run.profiles=mysql"]
```

让我们建立我们的形象
Let's build our image

```console
$ docker build --tag java-docker .
```

现在，让我们运行我们的容器。这一次，我们需要设置MYSQL_URL环境变量，以便我们的应用程序知道使用什么连接字符串来访问数据库。我们将使用该docker run命令执行此操作。
Now, let’s run our container. This time, we need to set the `MYSQL_URL` environment variable so that our application knows what connection string to use to access the database. We’ll do this using the `docker run` command.

```console
$ docker run --rm -d \
--name springboot-server \
--network mysqlnet \
-e MYSQL_URL=jdbc:mysql://mysqlserver/petclinic \
-p 8080:8080 java-docker
```

让我们测试一下我们的应用程序是否已连接到数据库并能够列出 Veterinarians。
Let’s test that our application is connected to the database and is able to list Veterinarians.

```console
$ curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```

您应该会从我们的服务中收到以下 json。
You should receive the following json back from our service.

```json
{"vetList":[{"id":1,"firstName":"James","lastName":"Carter","specialties":[],"nrOfSpecialties":0,"new":false},{"id":2,"firstName":"Helen","lastName":"Leary","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":3,"firstName":"Linda","lastName":"Douglas","specialties":[{"id":3,"name":"dentistry","new":false},{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":2,"new":false},{"id":4,"firstName":"Rafael","lastName":"Ortega","specialties":[{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":1,"new":false},{"id":5,"firstName":"Henry","lastName":"Stevens","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":6,"firstName":"Sharon","lastName":"Jenkins","specialties":[],"nrOfSpecialties":0,"new":false}]}
```

## 在本地使用 Compose 开发

在本节中，我们将创建一个 Compose 文件以java-docker使用单个命令启动我们的和 MySQL 数据库。我们还将设置 Compose 文件以java-docker在调试模式下启动应用程序，以便我们可以将调试器连接到正在运行的 Java 进程。
In this section, we’ll create a Compose file to start our `java-docker` and the MySQL database using a single command. We’ll also set up the Compose file to start the `java-docker` application in debug mode so that we can connect a debugger to the running Java process.

petclinic在 IDE 或文本编辑器中打开并创建一个名为docker-compose.dev.yml. 将以下命令复制并粘贴到文件中。
Open the `petclinic` in your IDE or a text editor and create a new file named `docker-compose.dev.yml`. Copy and paste the following commands into the file.

```yaml
version: '3.8'
services:
  petclinic:
    build:
      context: .
    ports:
      - 8000:8000
      - 8080:8080
    environment:
      - SERVER_PORT=8080
      - MYSQL_URL=jdbc:mysql://mysqlserver/petclinic
    volumes:
      - ./:/app
    command: ./mvnw spring-boot:run -Dspring-boot.run.profiles=mysql -Dspring-boot.run.jvmArguments="-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000"

  mysqlserver:
    image: mysql:8.0.23
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=
      - MYSQL_ALLOW_EMPTY_PASSWORD=true
      - MYSQL_USER=petclinic
      - MYSQL_PASSWORD=petclinic
      - MYSQL_DATABASE=petclinic
    volumes:
      - mysql_data:/var/lib/mysql
      - mysql_config:/etc/mysql/conf.d
volumes:
  mysql_data:
  mysql_config:
```

这个 Compose 文件非常方便，因为我们不必键入要传递给docker run命令的所有参数。我们可以使用 Compose 文件声明性地做到这一点。
This Compose file is super convenient as we do not have to type all the parameters to pass to the `docker run` command. We can declaratively do that using a Compose file.

我们公开端口 8000 并声明 JVM 的调试配置，以便我们可以附加调试器。
We expose port 8000 and declare the debug configuration for the JVM so that we can attach a debugger.

使用 Compose 文件的另一个非常酷的功能是我们将服务解析设置为使用服务名称。因此，我们现在可以mysqlserver在我们的连接字符串中使用。我们使用的原因mysqlserver是因为我们在 Compose 文件中命名了我们的 MySQL 服务。
Another really cool feature of using a Compose file is that we have service resolution set up to use the service names. Therefore, we are now able to use `mysqlserver` in our connection string. The reason we use `mysqlserver` is because that is what we've named our MySQL service as in the Compose file.

现在，启动我们的应用程序并确认它运行正常。
Now, to start our application and to confirm that it is running properly.

```console
$ docker-compose -f docker-compose.dev.yml up --build
```

我们传递--build标志，以便 Docker 编译我们的图像，然后启动容器。如果运行成功，您应该会看到类似的输出：
We pass the `--build` flag so Docker will compile our image and then starts the containers. You should see similar output if it runs successfully:

![Java Compose output](images/java-compose-output.png)

现在让我们测试我们的 API 端点。运行以下 curl 命令：
Now let’s test our API endpoint. Run the following curl commands:

```console
$ curl  --request GET \
  --url http://localhost:8080/vets \
  --header 'content-type: application/json'
```

您应该会收到以下回复：
You should receive the following response:

```json
{"vetList":[{"id":1,"firstName":"James","lastName":"Carter","specialties":[],"nrOfSpecialties":0,"new":false},{"id":2,"firstName":"Helen","lastName":"Leary","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":3,"firstName":"Linda","lastName":"Douglas","specialties":[{"id":3,"name":"dentistry","new":false},{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":2,"new":false},{"id":4,"firstName":"Rafael","lastName":"Ortega","specialties":[{"id":2,"name":"surgery","new":false}],"nrOfSpecialties":1,"new":false},{"id":5,"firstName":"Henry","lastName":"Stevens","specialties":[{"id":1,"name":"radiology","new":false}],"nrOfSpecialties":1,"new":false},{"id":6,"firstName":"Sharon","lastName":"Jenkins","specialties":[],"nrOfSpecialties":0,"new":false}]}
```

## 连接到 Debugger

我们将使用 IntelliJ IDEA 附带的调试器。您可以使用此 IDE 的社区版本。在 IntelliJ IDEA 中打开您的项目，然后转到“运行”菜单 >“编辑配置”。添加类似于以下内容的新远程 JVM 调试配置：
We’ll use the debugger that comes with the IntelliJ IDEA. You can use the community version of this IDE. Open your project in IntelliJ IDEA and then go to the **Run** menu > **Edit Configuration**. Add a new Remote JVM Debug configuration similar to the following:

![Java Connect a Debugger](images/connect-debugger.png)

让我们设置一个断点
Let's set a breakpoint

打开以下文件src/main/java/org/springframework/samples/petclinic/vet/VetController.java并在showResourcesVetList函数内添加断点，例如第 54 行。
Open the following file `src/main/java/org/springframework/samples/petclinic/vet/VetController.java` and add a breakpoint inside the `showResourcesVetList` function, line 54 for example.

启动您的调试会话，运行菜单，然后调试NameOfYourConfiguration
Start your debug session, **Run** menu and then **Debug _NameOfYourConfiguration_**

![Debug menu](images/debug-menu.png)

您现在应该会在 Compose 应用程序的日志中看到连接。
You should now see the connection in the logs of your Compose application.

![Compose log file ](images/compose-logs.png)

我们现在可以调用服务器端点。
We can now call the server endpoint.

```console
$ curl --request GET --url http://localhost:8080/vets
```

您应该已经在第 54 行看到了代码中断，现在您可以像平常一样使用调试器。您还可以检查和观察变量、设置条件断点、查看堆栈跟踪以及执行其他一些操作。
You should have seen the code break on line 54 and now you are able to use the debugger just like you would normally. You can also inspect and watch variables, set conditional breakpoints, view stack traces and a do bunch of other stuff.

![Debugger code breakpoint](images/debugger-breakpoint.png)

您还可以激活 SpringBoot Dev Tools 提供的实时重新加载选项。查看SpringBoot 文档以获取有关如何连接到远程应用程序的信息。
You can also activate the live reload option provided by SpringBoot Dev Tools. Check out the [SpringBoot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-devtools-remote){:target="_blank" rel="noopener" class="_"} for information on how to connect to a remote application.

## 下一步

在这个模块中，我们研究了创建一个通用的开发镜像，我们可以像使用普通命令行一样使用它。我们还设置了 Compose 文件以公开调试端口并配置 Spring Boot 以实时重新加载我们的更改。
In this module, we took a look at creating a general development image that we can use pretty much like our normal command line. We also set up our Compose file to expose the debugging port and configure Spring Boot to live reload our changes.

在下一个模块中，我们将看看如何在 Docker 中运行单元测试。
In the next module, we’ll take a look at how to run unit tests in Docker. See

[Run your tests](run-tests.md){: .button .primary-btn}

## Feedback

Help us improve this topic by providing your feedback. Let us know what you think by creating an issue in the [Docker Docs](https://github.com/docker/docker.github.io/issues/new?title=[Java%20docs%20feedback]){:target="_blank" rel="noopener" class="_"} GitHub repository. Alternatively, [create a PR](https://github.com/docker/docker.github.io/pulls){:target="_blank" rel="noopener" class="_"} to suggest updates.
