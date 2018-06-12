---
categories: linux
tags:
  - Linux

title: Linux目录结构
date: 2018-06-11
---

# Linux目录结构

初学Linux，首先需要弄清Linux 标准目录结构

```
/
├── bin # 二进制执行文件
├── boot
│   ├── efi
│   └── grub
├── cgroup
│   ├── blkio
│   ├── cpu
│   ├── cpuacct
│   ├── cpuset
│   ├── devices
│   ├── freezer
│   ├── hugetlb
│   ├── memory
│   └── perf_event
├── dev
│   ├── block
│   ├── char
│   ├── cpu
│   ├── disk
│   ├── fd -> /proc/self/fd
│   ├── input
│   ├── mapper
│   ├── net
│   ├── pts
│   ├── shm
│   ├── vfio
│   └── xen
├── etc
│   ├── acpi
│   ├── alertmanager-templates
│   ├── alternatives
│   ├── amazon
│   ├── audisp
│   ├── audit
│   ├── bash_completion.d
│   ├── blkid
│   ├── chkconfig.d
│   ├── cloud
│   ├── cron.d
│   ├── cron.daily
│   ├── cron.hourly
│   ├── cron.monthly
│   ├── cron.weekly
│   ├── dbus-1
│   ├── default
│   ├── depmod.d
│   ├── dhcp
│   ├── docker
│   ├── dracut.conf.d
│   ├── exports.d
│   ├── fonts
│   ├── gcrypt
│   ├── gnupg
│   ├── groff
│   ├── gss
│   ├── httpd
│   ├── init
│   ├── init.d -> rc.d/init.d
│   ├── iproute2
│   ├── java
│   ├── jvm
│   ├── jvm-commmon
│   ├── krb5.conf.d
│   ├── ld.so.conf.d
│   ├── libreport
│   ├── logrotate.d
│   ├── lvm
│   ├── mail
│   ├── maven
│   ├── modprobe.d
│   ├── NetworkManager
│   ├── ntp
│   ├── openldap
│   ├── opentsdb -> /usr/share/opentsdb/etc/opentsdb
│   ├── opt
│   ├── pam.d
│   ├── pkcs11
│   ├── pki
│   ├── pm
│   ├── popt.d
│   ├── ppp
│   ├── prelink.conf.d
│   ├── profile.d
│   ├── prometheus
│   ├── rc0.d -> rc.d/rc0.d
│   ├── rc1.d -> rc.d/rc1.d
│   ├── rc2.d -> rc.d/rc2.d
│   ├── rc3.d -> rc.d/rc3.d
│   ├── rc4.d -> rc.d/rc4.d
│   ├── rc5.d -> rc.d/rc5.d
│   ├── rc6.d -> rc.d/rc6.d
│   ├── rc.d
│   ├── request-key.d
│   ├── rpm
│   ├── rsyslog.d
│   ├── rwtab.d
│   ├── sasl2
│   ├── security
│   ├── selinux
│   ├── skel
│   ├── smrsh
│   ├── ssh
│   ├── ssl
│   ├── statetab.d
│   ├── sudoers.d
│   ├── sysconfig
│   ├── sysctl.d
│   ├── terminfo
│   ├── tmpfiles.d
│   ├── udev
│   ├── update-motd.d
│   ├── X11
│   ├── xdg
│   ├── xinetd.d
│   ├── yum
│   └── yum.repos.d
├── hbase_fs
│   ├── archive
│   ├── corrupt
│   ├── data
│   ├── MasterProcWALs
│   ├── mobdir
│   ├── oldWALs
│   ├── staging
│   └── WALs
├── hohott
│   ├── docker-demo
│   └── koa-demos
├── home # 存储普通用户的个人文件
├── lib
│   ├── firmware
│   ├── kbd
│   ├── modules
│   ├── terminfo
│   └── udev
├── lib64
│   ├── dbus-1
│   ├── device-mapper
│   ├── libnfsidmap
│   ├── rsyslog
│   ├── rtkaio
│   ├── security
│   ├── tls
│   └── xtables
├── local
├── lost+found
├── media
├── mnt
├── nexus-data
├── opt
│   ├── aws
│   ├── clink2-deploy
│   ├── data
│   ├── deploy
│   ├── heapster-1.5.3
│   ├── istio-0.7.1
│   └── istio-0.7.1.bak
├── proc
│   ├── 1
│   │...
│   ├── 9
│   ├── acpi
│   ├── bus
│   ├── driver
│   ├── fs
│   ├── irq
│   ├── net -> self/net
│   ├── scsi
│   ├── self -> 6516
│   ├── sys
│   ├── sysvipc
│   ├── thread-self -> 6516/task/6516
│   ├── tty
│   └── xen
├── root # root用户的home目录
├── run
│   ├── cloud-init
│   └── docker
├── sbin # 可执行程序的目录，但大多存放涉及系统管理的命令。只有root权限才能执行
├── selinux
├── srv
├── sys
│   ├── block
│   ├── bus
│   ├── class
│   ├── dev
│   ├── devices
│   ├── firmware
│   ├── fs
│   ├── hypervisor
│   ├── kernel
│   ├── module
│   └── power
├── tmp
│   ├── deploy
│   ├── hbase-root
│   ├── hsperfdata_root
│   ├── jetty-0.0.0.0-16010-master-_-any-6712533276056646004.dir
│   ├── jetty-0.0.0.0-16030-regionserver-_-any-8091681261346297363.dir
│   └── opentsdb
├── usr
│   ├── bin
│   ├── etc
│   ├── games
│   ├── include
│   ├── lib
│   ├── lib64
│   ├── libexec
│   ├── local
│   ├── sbin
│   ├── share
│   ├── src
│   └── tmp -> ../var/tmp
└── var
    ├── account
    ├── cache
    ├── db
    ├── empty
    ├── games
    ├── kerberos
    ├── lib
    ├── local
    ├── lock
    ├── log
    ├── mail -> spool/mail
    ├── nis
    ├── opt
    ├── preserve
    ├── run
    ├── spool
    ├── tmp
    ├── www
    └── yp
```

/

- root --- 启动Linux时使用的一些核心文件。如操作系统内核、引导程序Grub等。
- home --- 存储普通用户的个人文件
  - ftp --- 用户所有服务
  - httpd
  - samba
  - user1
  - user2
- bin --- 用户的二进制执行文件
- sbin --- 可执行程序的目录，但大多存放涉及系统管理的命令。只有root权限才能执行
- proc --- 虚拟，存在linux内核镜像；保存所有内核参数以及系统配置信息

参考：https://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=415328296&idx=1&sn=fa4d90b807f1f1552b8cf94a356009e8&scene=23&srcid=0205hM0eyERZuIwa901iMSKS#rd