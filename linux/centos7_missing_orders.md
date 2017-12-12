---
categories: linux
tags:
  - Linux

title: CentOS 7 中不见的命令
date: 2017-11-18
---

CentOS 7 之后，以前在CentOS 6 中的一些命令被淘汰了，默认不安装；这里记录那些被替换的命令，以供参考。

# ifconfig 替换为 ip addr


ip addr 相当于 ifconfig
```
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:3f:d4:7d brd ff:ff:ff:ff:ff:ff
    inet 192.168.13.128/24 brd 192.168.13.255 scope global dynamic ens33
       valid_lft 1174sec preferred_lft 1174sec
    inet6 fe80::baf:a9c7:248c:b461/64 scope link 
       valid_lft forever preferred_lft forever
```

# netstat 替換為 ss


```
#同 netstat -tunpl
$ ss -tunpl
 
# 查看 TCP 连接
$ ss -t
 
# 查看 UDP 连接
$ ss -u
```

# traceroute/traceroute6 替换为 tracepath

```
tracepath 8.8.8.8
```

# route 替换为 ip route

```
$ ip route
default via 192.168.13.2 dev ens33 proto static metric 100 
192.168.13.0/24 dev ens33 proto kernel scope link src 192.168.13.128 metric 100 
```

# arp 替換為 ip neighbor

```bash
$ ip neighbor
192.168.10.108 dev eth0 lladdr e0:ac:cb:66:2a:3c STALE
192.168.10.254 dev eth0 lladdr 10:7b:ef:47:ce:53 REACHABLE
```

# 如何安装旧的工具

这些被淘汰的工具默认是不安装的，如果想要使用需要自己进行安装。
在CentOS 6 之前的版本中，这些命令是在 net-tools 中，因此只需要安装 net-tools 就可以了。

```bash
$ yum install net-tools -y
```
