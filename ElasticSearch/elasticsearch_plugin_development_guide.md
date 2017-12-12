---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件开发完全指南

date: 2017-07-17
---

# 前言

[ElasticSearch](https://www.elastic.co/cn/products/elasticsearch) （以下简称ES）是一个非常强大的基于Lucene构建的全文搜索引擎，同时ES插件特性使得ES更为强大，我们可以根据自己的需求自己开发ES，但官网并没有给出ES插件开发的文档，只有[ES插件使用和介绍](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/index.html)的文档，在[文档最后一章](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/plugin-authors.html)写道让我们自己根据demo 和官网插件找灵感，这种事是不能忍的，因在工作中正好要修改ES插件，所以整理了这个文档（基于5.4.0），如有任何疑问可[联系我](http://blog.geekidentity.com/about/)。

# 插件简介

插件是以自定义方式增强核心Elasticsearch 功能的一种方式。 通过插件可以添加自定义映射类型，自定义分析器，本地脚本，自定义发现等。

插件包含JAR文件，一般也会包含脚本和配置文件，并且必须安装在群集中的每个节点上。 安装完成后，必须重新启动每个节点才能使插件变得可用。

安装具有自定义集群状态元数据的插件（如X-Pack）需要完全集群重新启动。当然仍然可以通过逐个重新启动来升级此类插件。
ES插件可以分为两类：

* 核心插件

此类别标识作为Elasticsearch项目一部分的插件。 与Elasticsearch同时交付，其版本号与Elasticsearch本身的版本号一致。 这些插件由Elastic团队维护，并有惊人的社区成员（对于开源插件）的赞赏。可以在Github项目页面上报告问题和提交bug。

* 社区贡献

此类别标识Elasticsearch项目之外的插件。 它们由个人开发者或私人公司提供，并拥有自己的许可证以及自己的版本管理系统。问题和bug报告通常可以在社区插件的网站上报告。

# 创建Maven项目

ES是用gradle构建的，但国内服务器端开发用的最多的还是Maven，因此我们用Maven来构建插件项目。

首先创建一个Maven simple项目，项目名叫es_simple_plugin。并修改pom文件如下所示：

```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.geekidentity</groupId>
	<artifactId>es_simple_plugin</artifactId>
	<description>es plugin sample</description>
	<version>0.0.1</version>

	<properties>
		<elasticsearch.version>5.4.0</elasticsearch.version>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<elasticsearch.plugin.name>elasticsearch_sample_plugin</elasticsearch.plugin.name>
		<elasticsearch.plugin.site>true</elasticsearch.plugin.site>
		<elasticsearch.plugin.jvm>true</elasticsearch.plugin.jvm>
		<elasticsearch.plugin.java.version>1.8</elasticsearch.plugin.java.version>
		<!-- ES插件的类名 -->
		<elasticsearch.plugin.classname>com.geekidentity.es_plugin.EsSimplePlugin</elasticsearch.plugin.classname>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.elasticsearch</groupId>
			<artifactId>elasticsearch</artifactId>
			<version>${elasticsearch.version}</version>
			<scope>provided</scope>
		</dependency>

		<!-- Testing -->
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-api</artifactId>
			<version>2.7</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.logging.log4j</groupId>
			<artifactId>log4j-core</artifactId>
			<version>2.7</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.elasticsearch.test</groupId>
			<artifactId>framework</artifactId>
			<version>${elasticsearch.version}</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
	
	<build>
		<resources>
			<resource>
				<directory>src/main/resources</directory>
				<filtering>false</filtering>
				<excludes>
					<exclude>*.properties</exclude>
				</excludes>
			</resource>
		</resources>
		<plugins>
			<!-- 使用assembly 打包成zip格式 -->
			<plugin>
				<artifactId>maven-assembly-plugin</artifactId>
				<configuration>
					<outputDirectory>${project.build.directory}/releases/</outputDirectory>
					<descriptors>
						<descriptor>${basedir}/src/main/assembly/plugin.xml</descriptor>
					</descriptors>
				</configuration>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>single</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

创建文件 /src/main/assembly/plugin.xml，内容如下所示：

```XML
<?xml version="1.0"?>
<assembly  
    xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.0 http://maven.apache.org/xsd/assembly-1.1.0.xsd">
    <id>release</id>
    <formats>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>  
        <fileSet>
            <directory>${project.basedir}/config/dic</directory>
            <outputDirectory>/elasticsearch</outputDirectory>
        </fileSet>
        <fileSet>
            <directory>${project.basedir}/config</directory>
            <outputDirectory>/elasticsearch/config</outputDirectory>
            <excludes>
                <exclude>**/dic/**</exclude>
            </excludes>
        </fileSet>

        <fileSet>
            <directory>${project.basedir}/src/main/plugin-metadata</directory>
            <outputDirectory>/elasticsearch</outputDirectory>
        </fileSet>
    </fileSets>
    <files>
        <file>
            <source>${project.basedir}/src/main/resources/plugin-descriptor.properties</source>
            <filtered>true</filtered>
            <outputDirectory>/elasticsearch</outputDirectory>
        </file>
    </files>
    <dependencySets>
        <dependencySet>
            <outputDirectory>/elasticsearch</outputDirectory>
            <useProjectArtifact>true</useProjectArtifact>
            <excludes>
                <exclude>org.elasticsearch:elasticsearch</exclude>
            </excludes>
        </dependencySet>
    </dependencySets>
</assembly>
```

# plugin-descriptor.properties

所有ES插件必须在名为elasticsearch的文件夹中包含一个名为plugin-descriptor.properties的文件。在`src/main/resources`目录创建该文件，内容如下：

```
description=${project.description}

version=${project.version}

name=${elasticsearch.plugin.name}

site=${elasticsearch.plugin.site}

jvm=${elasticsearch.plugin.jvm}

classname=${elasticsearch.plugin.classname}

java.version=${elasticsearch.plugin.java.version}

elasticsearch.version=${elasticsearch.version}
```



# 继承Plugin类

类 `org.elasticsearch.plugins.Plugin` 是ES的一个扩展点（point），允许插入自定义功能，也就是说我们的插件必须先继承这个类。此类具有多个可用于所有插件的扩展，此外，还可以实现以下任何接口以进一步自定义Elasticsearch：

* ActionPlugin 
* AnalysisPlugin 
* ClusterPlugin 
* DiscoveryPlugin 
* IngestPlugin 
* MapperPlugin 
* NetworkPlugin 
* RepositoryPlugin 
* ScriptPlugin 
* SearchPlugin 

这个插件类必须有一个无参的默认构造方法或有一个以类 `org.elasticsearch.common.settings.Settings` 为参数的构造方法。

插件的加载就从这个类开始，

# 加载插件配置信息

每个插件都可以有自己的配置信息，代码如下：

```Java
package com.geekidentity.es_plugin;

import java.io.IOException;
import java.nio.file.Path;

import org.apache.logging.log4j.Logger;
import org.elasticsearch.common.logging.Loggers;
import org.elasticsearch.common.settings.Setting;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.env.Environment;

/**
 * 插件的配置
 * 
 * @author geekidentity
 *
 */
public class ExamplePluginConfiguration {
	
	private final Logger logger = Loggers.getLogger(ExamplePluginConfiguration.class);
	
	private final Settings customSettings;

	public static final Setting<String> TEST_SETTING = 
			new Setting<String>("test", "default_value", 
			(value) -> value, Setting.Property.Dynamic);

	public ExamplePluginConfiguration(Environment evn) {
		
		// 目录部分与此插件的artifactId 一致
		
		Path path = evn.configFile().resolve("elasticsearch-sample-plugin/example.yml");
		logger.info("loading config file: " + path);
		try {
			customSettings = Settings.builder().loadFromPath(path).build();
		} catch (IOException e) {
			throw new RuntimeException("Failed to load settings, giving up", e);
		}

		// asserts for tests
		assert customSettings != null;
		assert TEST_SETTING.get(customSettings) != null;
	}

	public String getTestConfig() {
		return TEST_SETTING.get(customSettings);
	}
}

```

