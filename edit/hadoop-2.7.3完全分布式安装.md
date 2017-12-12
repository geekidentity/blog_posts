---
categories: hadoop

tags: 
  - Hadoop
  - 大数据

title: hadoop-2.7.3完全分布式安装教程详解

date: 2017-01-09
---

本教程详细解释了在虚拟机上对Hadoop-2.7.3进行完全分布式安装，安装Hadoop要求对Linux比较熟悉。目前Hadoop2.x中最新的就2.7.3，希望对新人有帮。

## 服务器环境准备


IP（改成自己的IP） | 机器名 | 系统
---|---|---
192.168.0.1|master|CentOS7
192.168.0.2|slave1|CentOS7
192.168.0.3|slave2|CentOS7



用户：hadoop（最好不要使用root用户，所有的数据放到当前用户下防止新手出错）

JDK 1.8（用java –version查看）

Hadoop版本2.7.3

## 开始部署
==以下部署动作全部在master上执行，slave1和slave2通过scp命令传过去。==

Hadoop安装在当前用户下applications中。

设置用户环境变量，在当前用户home目录下，编辑.bashrc，在末尾添加Hadoop环境变量


```
HADOOP_INSTALL=/home/hadoop/applications/hadoop-2.7.3
PATH=$HADOOP_INSTALL/bin:$PATH
PATH=$HADOOP_INSTALL/sbin:$PATH
```


修改hosts，编辑/etc/hosts，末尾添加：

```
192.168.0.1	master
192.168.0.2	slave1
192.168.0.3	slave2
```


配置ssh免密登陆
修改hadoop配置文件
在$HADOOP_INSTALL/etc/hadoop下


对下面配置文件进行如下修改：

## 必须配置项配置
### slaves
集群中结点域名/IP，所有结点都要加上。

```
master
slave1
slave2
```


### core-site.xml

核心配置，也是全局配置，

```
<configuration>
        <property>
                <!-- 默认文件系统的名称 -->
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000</value>
        </property>
        <property>
                <!-- 其他临时文件的基目录 -->
                <name>hadoop.tmp.dir</name>
                <value>file:/home/hadoop/workspaces/hadoop/tmp</value>
        </property>
</configuration>
```


### hdfs-site.xml


```
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/home/hadoop/workspaces/hadoop/dfs/name</value>
        </property>
        <property>
                <name>dfs.namenode.data.dir</name>
                <value>file:/home/ hadoop /workspaces/hadoop/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
</configuration>
```


yarn-site.xml


```
<configuration>
        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>master </value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master:8032</value>
        </property>
</configuration>
```


mapred-site.xml


```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>master:19888</value>
        </property>
</configuration>
```


通过scp 将配置好的hadoop复制到其他结点上。

在master上启动hadoop,执行

```
start-all.sh
```


完成后进入master:8088查看集群状态

