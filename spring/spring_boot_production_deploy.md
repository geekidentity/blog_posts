---
categories: spring

tags: 
  - Spring Boot
  - Spring
  - Java


title: Spring Boot 生产环境部署

date: 2017-04-26
---

欢迎加群交流（Spring Boot技术交流）：490788638

除了使用java -jar运行Spring Boot应用程序外，还可以为Unix系统打包完全可执行的应用程序。 这使得在常见的生产环境中安装和管理Spring Boot应用程序非常容易。

Spring Boot 提供了一个tools工具，该工具可以方便的让我们将程序部署到生产环境，本文将结合官网与实际项目部署，给出一个完美的部署方案。

## 将Spring Boot应用程序打包为“完全可执行”的jar包

要使用Maven创建“完全可执行”的jar，请使用以下插件配置：

```XML
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

这里的“完全可执行”实际上是Spring Boot tools在打包的过程中将bash脚本及一些辅助进行启动的Java代码打包到我们的项目中，这样

使用Gradle：

```shell
springBoot {
    executable = true
}
```
然后，您可以通过键入./my-application.jar（其中my-application是您工程的artifact的名称）来运行应用程序。

> 完全可执行的jar通过在文件的前面嵌入一个额外的脚本来工作。 并不是所有的工具目前都接受这种格式，所以你可能并不总是能够使用这种技术。

> 默认脚本支持大多数Linux发行版，并在CentOS和Ubuntu上进行了测试。 其他平台，如OS X和FreeBSD，将需要使用自定义的embeddedLaunchScript。

> 当运行完全可执行的jar时，它将使用jar的目录作为工作目录。

### 59.1 Unix/Linux 服务

Spring Boot应用程序可以使用init.d或systemd轻松地作为Unix / Linux服务启动。

#### 59.1.1 作为init.d服务进行安装（System V）

如果您配置了Spring Boot的Maven或Gradle插件来生成[完全可执行的jar](http://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/htmlsingle/#deployment-install)，并且您没有使用自定义的embeddedLaunchScript，那么您的应用程序可以用作init.d服务。 简单地将jar链接到init.d以支持标准的start，stop，restart和status命令。

该脚本支持以下功能：

* 以拥有该jar文件的用户启动服务
* 使用/var/run/<appname>/<appname>.pid跟踪应用程序的PID
* 将控制台日志写入/var/log/<appname>.log

假设你有一个Spring Boot应用程序安装在 /var/myapp 中，安装Spring Boot应用程序作为init.d服务只需创建一个符号链接：

```shell
sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```
一旦安装，您可以按照Linux系统常用的方式启动和停止服务。 例如，在基于Debian的系统上：

```shell
$ service myapp start
```
> 如果您的应用程序无法启动，请检查写入/var/log/<appname>.log的日志文件是否有错误日志。

您还可以将应用程序标记为使用标准操作系统工具自动启动。 例如Debian：

```shell
$ update-rc.d myapp defaults <priority>
```
#### 保护init.d服务

> 以下是关于如何保护作为init.d服务运行的Spring Boot应用程序的一组指导。 它并不是为了强化应用程序和运行环境而应该做的所有事情的详尽列表。

当以root身份执行时，如使用root用于启动init.d服务的情况，默认可执行脚本将以拥有该jar文件的用户身份运行应用程序。 您不应该以root身份运行Spring Boot应用程序，因此您的应用程序的jar文件不应该由root拥有。 相反，创建一个特定的用户来运行应用程序，并使用chown将其作为jar文件的所有者。 例如：

```shell
$ chown bootapp:bootapp your-app.jar
```
在这种情况下，默认的可执行脚本将作为bootapp用户运行应用程序。

> 为了减少应用程序的用户帐户遭到入侵的机会，您应该考虑防止其使用登录shell。 例如，将帐户的shell设置为 /usr/sbin/nologin 。

您还应该采取措施来阻止修改jar文件。 首先，配置其权限，使其不能被写入，并且只能由其所有者读取或执行：

```shell
$ chmod 500 your-app.jar
```
其次，如果您的应用程序或运行它的帐户被泄露，您还应该采取措施限制jar包被损坏。如果攻击者获得访问权限，他们可以使jar文件可写，并更改其内容。防止这种情况的一种方法是使用chattr使其变得不可变：

```shell
$ sudo chattr +i your-app.jar
```
这将阻止任何用户（包括root）修改该jar。

如果使用root来控制应用程序的服务，并且使用.conf文件来自定义其启动，那么root用户将读取和评估该.conf文件。 应该保证相应的安全。 使用chmod，以便该文件只能由所有者读取，并使用chown使root成为所有者：

```shell
$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
```
#### 59.1.2 作为systemd服务进行安装

Systemd是System V init系统的后继者，现在被许多现代Linux发行版使用。尽管您可以继续使用systemd的init.d脚本，但也可以使用systemd'service'脚本启动Spring Boot应用程序。

假设您在 /var/myapp 中安装了一个Spring Boot应用程序，要将Spring Boot应用作为系统服务安装为使用以下示例创建名为myapp.service的脚本，并将其放在 /etc/systemd/system 目录中：

```shell
[Unit]
Description=myapp
After=syslog.target

[Service]
User=myapp
ExecStart=/var/myapp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

> 请记住更改应用程序的Description，User 和ExecStart 字段。

> 请注意，ExecStart字段不声明脚本操作命令，这意味着默认情况下使用run命令。

请注意，与运行init.d服务不同，运行应用程序，PID文件和控制台日志文件的用户由systemd本身管理，因此必须使用“service”脚本中的相应字段进行配置。 有关详细信息，请参阅服务单元配置手册页。

请注意，与运行init.d服务不同，运行应用程序，PID文件和控制台日志文件的用户由systemd本身管理，因此必须使用“service”脚本中的相应字段进行配置。 有关详细信息，请参阅[服务单元配置手册页](https://www.freedesktop.org/software/systemd/man/systemd.service.html)。

要将应用程序标记为在系统启动时自动启动，请使用以下命令：

```shell
$ systemctl enable myapp.service
```
有关详细信息，请参阅man systemctl。

#### 59.1.3 自定义启动脚本

由Maven或Gradle插件编写的默认嵌入式启动脚本可以通过多种方式进行自定义。 对于大多数人来说，使用默认脚本以及一些自定义项通常就足够了。 如果您发现无法自定义需要的内容，则可以随时使用embeddedLaunchScript选项来完全编写自己的文件。

##### 编写自定义脚本

在将起始脚本写入jar文件时，自定义元素是很有意义的。 例如，init.d脚本可以提供一个“描述”，因为你知道这一点（它不会改变），你可以在生成jar时提供它。
