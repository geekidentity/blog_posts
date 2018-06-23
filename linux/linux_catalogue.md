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
/  # 根目录，一般根目录下只存放目录，不要存放文件，/etc、/bin、/dev、/lib、/sbin应该和根目录放置在一个分区中
├── bin # 二进制执行文件
├── boot # 放置linux系统启动时用到的一些文件。/boot/vmlinuz为linux的内核文件，以及/boot/gurb。建议单独分区，分区大小100M即可
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
├── dev # 存放linux系统下的设备文件，访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱mount /dev/cdrom /mnt
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
├── etc # 系统配置文件存放的目录，不建议在此目录下存放可执行文件，重要的配置文件有/etc/inittab、/etc/fstab、/etc/init.d、/etc/X11、/etc/sysconfig、/etc/xinetd.d修改配置文件之前记得备份。
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
│   ├── X11 # 存放与x windows有关的设置。
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
├── home # 系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下，~表示当前用户的家目录，~test表示用户test的家目录。建议单独分区，并设置较大的磁盘空间，方便用户存放数据
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
├── lost+found # 系统异常产生错误时，会将一些遗失的片段放置于此目录下，通常这个目录会自动出现在装置目录下。如加载硬盘于/disk 中，此目录下就会自动产生目录/disk/lost+found
├── media
├── mnt
├── nexus-data
├── opt # 给主机额外安装软件所摆放的目录。如：FC4使用的Fedora 社群开发软件，如果想要自行安装新的KDE 桌面软件，可以将该软件安装在该目录下。以前的 Linux 系统中，习惯放置在 /usr/local 目录下
│   ├── aws
│   ├── clink2-deploy
│   ├── data
│   ├── deploy
│   ├── heapster-1.5.3
│   ├── istio-0.7.1
│   └── istio-0.7.1.bak
├── proc # 此目录的数据都在内存中，如系统核心，外部设备，网络状态，由于数据都存放于内存中，所以不占用磁盘空间，比较重要的目录有/proc/cpuinfo、/proc/interrupts、/proc/dma、/proc/ioports、/proc/net/*等
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
├── root # 系统管理员root的家目录，系统第一个启动的分区为/，所以最好将/root和/放置在一个分区下。
├── run
│   ├── cloud-init
│   └── docker
├── sbin # 可执行程序的目录，但大多存放涉及系统管理的命令。只有root权限才能执行
├── selinux
├── srv # 服务启动之后需要访问的数据目录，如www服务需要访问的网页数据存放在/srv/www内
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
├── tmp # 一般用户或正在执行的程序临时存放文件的目录,任何人都可以访问,重要数据不可放置在此目录下
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

# FHS

维基百科：https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84%E6%A0%87%E5%87%86

FHS依据文件系统使用的频繁与否与是否允许使用者随意更动， 而将目录定义成为四种交互作用的形态，用表格来说有点像底下这样：

|                    | 可分享的(shareable)                                   | 不可分享的(unshareable)                    |
| ------------------ | ----------------------------------------------------- | ------------------------------------------ |
| 不变的(static)     | /usr (软件放置处)   /opt (第三方协力软件)             | /etc (配置文件)   boot (开机与核心档)      |
| 可变动的(variable) | /var/mail (使用者邮件信箱)   /var/spool/news (新闻组) | /var/run (程序相关)   /var/lock (程序相关) |

