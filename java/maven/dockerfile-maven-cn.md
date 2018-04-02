---
categories: Maven

tags: 
  - maven
  - docker

title: Dockerfile Maven 插件使用

date: 2018-03-22
---

# Dockerfile Maven 插件使用

这是一个将Docker与Maven无缝集成的Maven插件，可以方便地使用Maven打包Docker image（注意：原来的项目[docker-maven-plugin ](https://github.com/spotify/docker-maven-plugin) 已经不建议使用）。

设计目标：

- 不要试图做任何事情。 这个插件使用Dockerfiles构建Docker项目的而且是强制性的。
- 将Docker构建过程集成到Maven构建过程中。如果绑定默认phases，那么当你键入mvn package时，你会得到一个Docker镜像。 当你键入mvn deploy时，你的图像被push。
- 让goals记住你在做什么。 你可以输入 `mvn dockerfile:build`及后面的 `mvn dockerfile:build`和`mvn dockerfile:push` 都没有问题。这也消除了之前像 `mvn dockerfile:build -DalsoPush`这样的命令；相反，你可以只使用 `mvn dockerfile:build dockerfile:push`。
- 与Maven build reactor集成。你可以在一个项目中依赖另一个项目所构建的Docker image，Maven将按照正确的顺序构建项目。当你想要运行涉及多个服务的集成测试时，这非常有用。

该项目遵守 [Open Code of Conduct](https://github.com/spotify/code-of-conduct/blob/master/code-of-conduct.md).。 参与贡献代码，你需要遵守此代码规则。

[查看更新日志以获取发布列表](https://github.com/spotify/dockerfile-maven/blob/master/CHANGELOG.md)

## Set-up

该插件需要Java 7或更高版本以及Apache Maven 3或更高版本。要运行集成测试或在开发中使用该插件，需要有一个能正常工作的Docker。

## 例子

有关更多示例，请参阅[集成测试](https://github.com/spotify/dockerfile-maven/blob/master/plugin/src/it)目录。

特别是，[高级](https://github.com/spotify/dockerfile-maven/blob/master/plugin/src/it/advanced)测试展示了由两个微服务组成的一套服务，这些服务使用`helios-testing`进行集成测试。

这将配置插件以使用 `mvn package`构建映像，并使用 `mvn deploy`进行推送。 当然你也可以用 `mvn dockerfile:build`显式构建。

```xml
<plugin>
  <groupId>com.spotify</groupId>
  <artifactId>dockerfile-maven-plugin</artifactId>
  <version>${dockerfile-maven-version}</version>
  <executions>
    <execution>
      <id>default</id>
      <goals>
        <goal>build</goal>
        <goal>push</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <repository>spotify/foobar</repository>
    <tag>${project.version}</tag>
    <buildArgs>
      <JAR_FILE>${project.build.finalName}.jar</JAR_FILE>
    </buildArgs>
  </configuration>
</plugin>
```

相应的Dockerfile可能如下所示

```dockerfile
FROM openjdk:8-jre
MAINTAINER David Flemström <dflemstr@spotify.com>

ENTRYPOINT ["/usr/bin/java", "-jar", "/usr/share/myservice/myservice.jar"]

# Add Maven dependencies (not shaded into the artifact; Docker-cached)
ADD target/lib           /usr/share/myservice/lib
# Add the service itself
ARG JAR_FILE
ADD target/${JAR_FILE} /usr/share/myservice/myservice.jar
```

## 优点

使用这个插件进行项目构建有很多优点。

### 更快的构建时间

这个插件让你更好地利用Docker缓存，通过让你在你的image中缓存Maven依赖关系，极大地加速你的构建。 它还鼓励避免 `maven-shade-plugin`，这也大大加快了构建速度。

### 一致的构建生命周期

你不再需要像下面这样了：

```bash
mvn package
mvn dockerfile:build
mvn verify
mvn dockerfile:push
mvn deploy
```

用下面这一行命令就可以了：

```bash
mvn deploy
```

通过基本配置，这将确保image在正确的时间被构建和push。

### 依赖其他服务的Docker镜像

你可以依赖另一个项目的Docker信息，因为此插件会在构建Docker镜像时附加项目元数据。 只需将这些信息添加到任何项目中：

```xml
<dependency>
  <groupId>com.spotify</groupId>
  <artifactId>foobar</artifactId>
  <version>1.0-SNAPSHOT</version>
  <type>docker-info</type>
</dependency>
```

现在，你可以读取有关你依赖的项目的Docker镜像的信息：

```java
String imageName = getResource("META-INF/docker/com.spotify/foobar/image-name");
```

这对于需要另一个项目最新版本的Docker镜像的集成测试非常有用。

请注意，你必须在POM（或父POM）中注册Maven extension，才能支持docker-info类型：

```xml
<build>
  <extensions>
    <extension>
      <groupId>com.spotify</groupId>
      <artifactId>dockerfile-maven-extension</artifactId>
      <version>${version}</version>
    </extension>
  </extensions>
</build>
```

## 使用其他依赖Dockerfiles的Docker工具

你的项目如下所示：

```
a/
  Dockerfile
  pom.xml
b/
  Dockerfile
  pom.xml
```

你现在可以使用Fig或docker-compose或其他一些与Dockerfiles配合使用的系统来使用这些项目。 例如，一个`docker-compose.yml` 可能如下所示：

```
service-a:
  build: a/
  ports:
  - '80'

service-b:
  build: b/
  links:
  - service-a
```

现在， `docker-compose up`和`docker-compose build` 将按预期工作。

## 身份验证和私有Docker注册中心支持

从版本1.3.0开始，当你pulling, pushing, 或 building images 到private registries中时插件将自动使用 `~/.dockercfg` 或 `~/.docker/config.json`文件中的配置。

此外，如果插件能够成功加载[Google的“应用程序默认凭证”](https://developers.google.com/identity/protocols/application-default-credentials)，该插件将支持Google Container Registry。 如果已定义，该插件还将从环境变量`DOCKER_GOOGLE_CREDENTIALS` 指向的文件中加载Google凭据。 由于GCR认证需要为给定凭证检索短期访问代码，因此对此注册表的支持将被融入到基础的docker-client中，而不必在运行插件之前配置docker配置文件。

GCR用户可能需要通过gcloud初始化他们的应用程序默认凭证。 根据插件的运行位置，他们可能希望通过运行以下命令来使用他们的[Google identity ](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)。

```bash
gcloud auth application-default login
```

或者改为[创建一个service帐户](https://cloud.google.com/docs/authentication/getting-started#creating_a_service_account)。

## 使用maven settings.xml进行身份验证

从版本1.3.6开始，你可以使用maven的 settings.xml文件进行身份验证，而不是使用docker配置。 只需添加类似于以下配置：

```xml
<configuration>
  <repository>docker-repo.example.com:8080/organization/image</repository>
  <tag>latest</tag>
  <useMavenSettingsForAuth>true</useMavenSettingsForAuth>
</configuration>
```

你也可以在命令行上使用 `-Ddockerfile.useMavenSettingsForAuth=true`。

然后，在你的maven设置文件中，为服务器添加配置：

```xml
<servers>
  <server>
    <id>docker-repo.example.com:8080</id>
    <username>me</username>
    <password>mypassword</password>
  </server>
</servers>
```

与其他服务器配置完全相同。

## 使用maven pom.xml进行身份验证

从版本1.3.XX开始，你可以使用pom本身的配置进行身份验证。 只需添加类似于以下配置（经测试，1.4.0版本Windows10下向私有仓库push时会报错 denied: requested access to the resource is denied，所以建议使用maven settings.xml进行身份验证；但Linux环境该方案可行 ）：

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>${version}</version>
    <configuration>
        <username>repoUserName</username>
        <password>repoPassword</password>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

或更简单，

```xml
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>dockerfile-maven-plugin</artifactId>
    <version>${version}</version>
    <configuration>
        <repository>${docker.image.prefix}/${project.artifactId}</repository>
        <buildArgs>
            <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
        </buildArgs>
    </configuration>
</plugin>
```

用这个命令行调用：

```bash
mvn goal -Ddockerfile.username=... -Ddockerfile.password=...
```

## Maven Goals

可用于此插件的Goals ：

| Goal               | Description                    | Default Phase |
| ------------------ | ------------------------------ | ------------- |
| `dockerfile:build` | 从Dockerfile构建Docker镜像。   | `package`     |
| `dockerfile:tag`   | Tag Docker镜像。               | `package`     |
| `dockerfile:push`  | 将Docker镜像推送到repository。 | `deploy`      |

## 跳过Docker Goals 绑定到Maven phase

你可以将选项传递给maven以禁用 docker goals。

| Maven Option          | What Does *that thing* Do?                                   |
| --------------------- | ------------------------------------------------------------ |
| dockerfile.skip       | Disables the entire dockerfile plugin; all goals become no-ops. |
| dockerfile.build.skip | Disables the build goal; it becomes a no-op.                 |
| dockerfile.tag.skip   | Disables the tag goal; it becomes a no-op.                   |
| dockerfile.push.skip  | Disables the push goal; it becomes a no-op.                  |

例如，跳过整个dockerfile插件：

```bash
mvn clean package -Ddockerfile.skip
```

