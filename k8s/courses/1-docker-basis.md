
# Docker简介

Docker是DotCloud基于Go语言并遵循Apache 2.0协议开源的产品，于2013年3月发布首个版本，是一个世界领先的软件“集装箱化”平台。它基于 Linux 内核的 cgroup，namespace等技术对进程进行隔离，号称“build once, configure once and run anywhere“，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上。

[Docker官网](https://www.docker.com/)

![img](http://blog.geekidentity.com/images/k8s/courses/1-docker-basis/docker-logo.png)

# Docker架构

![img](http://blog.geekidentity.com/images/k8s/courses/1-docker-basis/docker-architecture.png)

完整的Docker有以下几个部分组成：

- Docker 客户端（Client）
- Docker 守护进程（Daemon）
- Docker 镜像（Image）
- Docker容器（Container）

 

# Docker应用场景

Docker的典型场景：

- 使应用的打包与部署自动化
- 创建轻量、私密的PAAS环境
- 实现自动化测试和持续的集成/部署
- 部署与扩展webapp、数据库和后台服务

# 容器与虚拟机比较

## 传统虚拟机技术

### Hypervisor

Hypervisor是一种运行在物理服务器和操作系统之间的中间软件层,可允许多个操作系统和应用共享一套基础物理硬件。Hypervisor是所有虚拟化技术的核心。非中断地支持多工作负载迁移的能力是Hypervisor的基本功能。当服务器启动并执行Hypervisor时，它会给每一台虚拟机分配适量的内存、CPU、网络和磁盘，并加载所有虚拟机的客户操作系统。（[IBM 虚拟化技术概述](https://www.ibm.com/developerworks/community/blogs/5144904d-5d75-45ed-9d2b-cf1754ee936a/entry/%25e8%2599%259a%25e6%258b%259f%25e5%258c%2596%25e6%258a%2580%25e6%259c%25af%25e6%25a6%2582%25e8%25bf%25b0?lang=en)）

**基于宿主操作系统的系统级虚拟化**

![基于宿主操作系统的系统级虚拟化](http://blog.geekidentity.com/images/k8s/courses/1-docker-basis/vm-base-system.png)

**基于硬件的系统级虚拟化**

![基于硬件的系统级虚拟化](http://blog.geekidentity.com/images/k8s/courses/1-docker-basis/vm-base-hard.png)

## 容器与虚拟机比较

- 传统虚拟机技术通过Hypervisor层抽象底层基础设施资源，提供相互隔离的虚拟机，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；
- 而容器内的应用进程运行于宿主的内核，容器内没有自己的内核， 没有客户机操作系统，而且也没有进行硬件虚拟，因此容器要比传统虚拟机更为轻便。
- 虚拟机是为提供系统环境而生的，容器是为提供应用环境而生的。

![img](http://blog.geekidentity.com/images/k8s/courses/1-docker-basis/docker-pk-vm.png)

# 运行一个容器

通过 docker run 运行一个容器：

```bash
$ docker run -itd busybox sleep 300
52eab400f2fa3324cc3a6772e334474251fe6c87a2cc3877310a711aba86da6b
```

通过 docker ps 查看容器：

```bash
$ docker ps | grep busybox
52eab400f2fa busybox "sleep 300" 28 seconds ago Up 27 seconds quizzical_fermi
```

通过 docker rm 删除容器：

```bash
$ docker rm -f 52eab400f2fa
52eab400f2fa
```



# 参考

1. [虚拟化技术的发展](https://www.ibm.com/developerworks/community/blogs/5144904d-5d75-45ed-9d2b-cf1754ee936a/entry/%25e8%2599%259a%25e6%258b%259f%25e5%258c%2596%25e6%258a%2580%25e6%259c%25af%25e6%25a6%2582%25e8%25bf%25b0?lang=en) 