---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件和集成[5.5] -- 插件管理

date: 2017-07-13
---

# 插件管理

插件脚本用于安装，列出和删除插件。 默认情况下，它位于$ES_HOME/bin 目录中，但可能位于不同的位置，具体取决于您安装的Elasticsearch软件包：

* [.zip和.tar.gz文档的目录布局](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/zip-targz.html#zip-targz-layout)
* [Debian包的目录布局](Debian包的目录布局)
* [RPM包的目录布局](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/rpm.html#rpm-layout)

运行以下命令获取使用说明：

```Bash
sudo bin/elasticsearch-plugin -h
```

**以root身份运行**

如果使用deb或rpm软件包安装了Elasticsearch，则以root身份运行/usr/share/elasticsearch/bin/elasticsearch-plugin，以便它可以写入磁盘上的相应文件。 否则以拥有Elasticsearch文件的用户的身份运行bin/elasticsearch-plugin 。

## 安装插件

每个插件的文档通常包含该插件的特定安装说明，但下面我们记录了各种可用选项：

**ES核心插件**

ES核心插件可按如下方式安装：

```Bash
sudo bin/elasticsearch-plugin install [plugin_name]
```
例如，要安装核心[ICU插件](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/analysis-icu.html)，只需运行以下命令：

```Bash
sudo bin/elasticsearch-plugin install analysis-icu
```
此命令将安装与Elasticsearch版本匹配的插件版本，并在下载时显示进度条。

## 自定义URL或文件系统

也可以通过指定URL直接从自定义位置下载插件：

```Bash
sudo bin/elasticsearch-plugin install [url]
```
> 必须是一个有效的URL，插件名称由其描述符（descriptor）确定。

例如，要从本地文件系统安装插件，可以运行：

```Bash
sudo bin/elasticsearch-plugin install file:///path/to/plugin.zip
```

插件脚本将拒绝与具有不受信任证书的HTTPS URL进行通信。 要使用自签名HTTPS证书，您需要将CA证书添加到本地Java信任库，并将该位置传递给脚本，如下所示：

```Bash
sudo ES_JAVA_OPTS="-Djavax.net.ssl.trustStore=/path/to/trustStore.jks" bin/elasticsearch-plugin install https://....
```

## 列出，删除和更新已安装的插件

### 列出插件

可以使用list选项检索当前加载的插件的列表：

```Bash
sudo bin/elasticsearch-plugin list
```
或者，使用[node-info API](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/cluster-nodes-info.html)来查找群集中每个节点上安装的插件。

### 删除插件

可以手动删除插件，通过删除plugins/ 下的相应目录或使用公共脚本：

```Bash
sudo bin/elasticsearch-plugin remove [pluginname]
```

删除Java插件后，您将需要重新启动节点才能完成删除过程。

### 更新插件

插件是针对特定版本的Elasticsearch而构建的，因此每次更新Elasticsearch时都必须重新安装插件。

```Bash
sudo bin/elasticsearch-plugin remove [pluginname]
sudo bin/elasticsearch-plugin install [pluginname]
```
## 其他命令行参数

插件脚本支持许多其他命令行参数：

### Silent/Verbose 模式

--verbose参数输出更多调试信息，而--silent参数关闭所有输出，包括进度条。 脚本可能会返回以下退出代码：

code | 描述
---|---
0 | 一切正常
64 | IO错误
74 | IO error
70 | 其他错误

### Batch 模式

某些插件需要比Core Elasticsearch默认提供的更多权限。 这些插件将列出所需的权限，并要求用户进行确认，然后再继续安装。

当从另一个程序（例如安装自动化脚本）运行插件安装脚本时，插件脚本应检测到它未从控制台调用，并跳过确认响应，自动授予所有请求的权限。 如果检测是否控制控制台调用失败，则可以通过指定-b或--batch来强制使用批处理模式，如下所示：

```Bash
sudo bin/elasticsearch-plugin install --batch [pluginname]
```

### 自定义配置目录

如果您的elasticsearch.yml配置文件位于自定义位置，则需要在使用插件脚本时指定配置文件的路径。 你可以这样做：

```Bash
sudo CONF_DIR=/path/to/conf/dir bin/elasticsearch-plugin install <plugin name>
```

### 代理设置

要通过代理安装插件，可以使用Java设置http.proxyHost和http.proxyPort（或https.proxyHost和https.proxyPort）将代理详细信息添加到ES_JAVA_OPTS环境变量中：

```Bash
sudo ES_JAVA_OPTS="-Dhttp.proxyHost=host_name -Dhttp.proxyPort=port_number -Dhttps.proxyHost=host_name -Dhttps.proxyPort=https_port_number" bin/elasticsearch-plugin install analysis-icu

```

或在Windows上：

```Bash
set ES_JAVA_OPTS="-Dhttp.proxyHost=host_name -Dhttp.proxyPort=port_number -Dhttps.proxyHost=host_name -Dhttps.proxyPort=https_port_number"
bin\elasticsearch-plugin install analysis-icu
```

## 插件目录

plugins目录的默认位置取决于您安装的软件包：

* [.zip和.tar.gz文档的目录布局](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/zip-targz.html#zip-targz-layout)
* [Debian包的目录布局](Debian包的目录布局)
* [RPM包的目录布局](https://www.elastic.co/guide/en/elasticsearch/reference/5.4/rpm.html#rpm-layout)


### 强制插件

如果您依赖某些插件，则可以通过将plugin.mandatory设置添加到config/elasticsearch.yml 文件中来定义强制性插件，例如：

```
plugin.mandatory: analysis-icu,lang-js
```

出于安全考虑，如果节点缺少必需的插件，节点将无法启动。