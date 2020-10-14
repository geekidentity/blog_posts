---
categories: linux
tags:
  - Linux
  - Mac

title: Mac 根目录下无法挂载文件解决方案
date: 2020-10-14
---

由于Mac默认有系统文件保护，所以无法在 `/`  下创建文件 `Read-only file system` 。百度出来的文章是关闭系统文件保护，但其实并不是官方方案，官方建议使用 `synthetic.conf` 

`man synthetic.conf`看到这个文件说明

```
SYNOPSIS
     synthetic.conf -- synthetic symbolic link and directory manifest

DESCRIPTION
     synthetic.conf describes virtual symbolic links and empty directories to
     be created at the root mount point. Because the root mount point is read-
     only as of macOS 10.15, physical files may not be created at this loca-
     tion. All writeable paths must reside on the data volume, which is
     mounted at /System/Volumes/Data.

     synthetic.conf provides a mechanism for some limited, user-controlled
     file-creation at /.  The synthetic entities described in this file are
     synthesized by the kernel during early system boot. They are not physi-
     cally present on the disk, but when the system is booted, they behave as
     if they were within certain parameters.

     synthetic.conf is intended to be used for creating mount points at /
     (e.g. for use as NFS mount points in enterprise deployments) and symbolic
     links (e.g. for creating a package manager root without modifying the
     system volume).  synthetic.conf is read by apfs.util(8) during early sys-
     tem boot.
```

这里其实就是说在macOS 10.15之后物理文件是不能在root目录下面穿件的，所有的文件都放到了/System/Volumes/Data目录下面。synthetic.conf提供了用户希望把文件创建到/目录下面的机制，当然即使我们这样子看到，实际文件也不会真正放到根目录下面，配置参数之后重启会生效，其实这里就是提供一种映射机制。

synthetic.conf 的目的其实就是把目录挂载到我们的/下面，其实这个就是我们要。
读完之后我们可以了解到这个其实就是升级之后Mac提供给我们的官方做法。

```shell
sudo vim /etc/synthetic.conf
# 添加一行记录(如果有两列需要使用 tab 进行分割，注意空格分割是无效的)，然后重启即可
# 举例
app	/Users/houfc/app
# 将会在根目录下创建 app 软连接到根目录下的 /Users/houfc/app 目录
```

重启之后的目录：

```
lrwxr-xr-x  1 root  wheel  16 10 13 18:33 /app -> /Users/houfc/app
```

