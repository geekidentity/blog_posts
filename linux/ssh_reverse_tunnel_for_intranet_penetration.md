---
categories: linux
tags:
  - Linux
  - SSH

title: 使用SSH反向隧道进行内网穿透
date: 2017-11-19
---

# 应用背景

主要介绍了如何利用SSH 反向隧道穿透NAT，并演示了如何维持一条稳定的SSH 隧道。

假设有机器 A 和 B，A 有公网IP，B 位于 NAT 之后并无可用的端口转发，现在想由 A 主动向 B 发起 SSH 连接。由于 B 在 NAT 后端，无可用公网IP + 端口 这样一个组合，所以A 无法穿透NAT，这篇文章应对的就是这种情况。

假设有这么三台机器

机器代号 | 机器位置 | 地址 | 账户 | ssh/sshd 端口 | 是否需要运行sshd
---|---|---|---|---|---
A | 位于公网 | a.site | usera | 22 | 是
B | 位于NAT 之后 | localhost | userb | 22 | 是
C | 位于NAT 之后 | localhost | userc | 22 | 否

> 这里默认你的系统init 程序为systemd，如果你使用其他的init 程序，如果没有特殊理由还是换到一个现代化的GNU/Linux 系统吧……

# SSH 反向隧道

这种手段实质上是由B 向A 主动地建立一个SSH 隧道，将A 的[6766 端口](http://www.adminsub.net/tcp-udp-port-finder/6766)转发到B 的22 端口上，只要这条隧道不关闭，这个转发就是有效的。有了这个端口转发，只需要访问A 的6766 端口反向连接B 即可。

首先在B 上建立一个SSH 隧道，将A 的6766 端口转发到B 的22 端口上：

```bash
B $ ssh -p 22 -qngfNTR 6766:localhost:22 usera@a.site
```
然后在A 上利用6766 端口反向SSH 到B：

```bash
A $ ssh -p 6766 userb@localhost
```

# 隧道的维持

## 稳定性维持

然而不幸的是SSH 连接是会超时关闭的，如果连接关闭，隧道无法维持，那么A 就无法利用反向隧道穿透B 所在的NAT 了，为此我们需要一种方案来提供一条稳定的SSH 反向隧道。

一个最简单的方法就是autossh，这个软件会在超时之后自动重新建立SSH 隧道，这样就解决了隧道的稳定性问题，如果你使用 CentOS，你可以这样获得它：

```bash
$ yum install autossh -y
```
下面在B 上做之前类似的事情，不同的是该隧道会由autossh 来维持：

```bash
B $ autossh -p 22 -M 6777 -NR 6766:localhost:22 usera@a.site
```
-M 参数指定的端口用来监听隧道的状态，与端口转发无关。

之后你可以在A 上通过6766 端口访问B 了：

```bash
A $ ssh -p 6766 userb@localhost
```

## 隧道的自动建立

然而这又有了另外一个问题，如果B 重启隧道就会消失。那么需要有一种手段在B 每次启动时使用autossh 来建立SSH 隧道。很自然的一个想法就是做成服务，之后会给出在systemd 下的一种解决方案。

# 打洞

之所以标题这么起，是因为自己觉得这件事情有点类似于UDP 打洞，即通过一台在公网的机器，让两台分别位于各自NAT 之后的机器可以建立SSH 连接。

下面演示如何使用SSH 反向隧道，让C 连接到B。

首先在A 上编辑sshd 的配置文件/etc/ssh/sshd_config，将GatewayPorts 开关打开：

```
GatewayPorts yes
```
然后重启sshd：

```
A $ service restart sshd
```
然后在B 上对之前用到的autossh 指令略加修改：

```
B $ autossh -p 22 -M 6777 -NR '*:6766:localhost:22' usera@a.site
```
之后在C 上利用A 的6766 端口SSH 连接到B：

```
C $ ssh -p 6766 userb@a.site
```
至此你已经轻而易举的穿透了两层NAT。

# 最终方案

整合一下前面提到的，最终的解决方案如下：

首先打开A 上sshd 的GatewayPorts 开关，并重启sshd（如有需要）。

然后在B 上新建一个用户autossh，根据权限最小化思想，B 上的autossh 服务将以autossh 用户的身份运行，以尽大可能避免出现安全问题：

```bash
B $ sudo useradd -m autossh
B $ sudo passwd autossh
```
紧接着在B 上为autossh 用户创建SSH 密钥，并上传到A：

```
B $ su - autossh
B $ ssh-keygen -t 'rsa' -C 'autossh@B'
B $ ssh-copy-id usera@a.site
```
注意该密钥不要设置密码，也就是运行`ssh-keygen` 指令时尽管一路回车，不要输入额外的字符。

```
然后在B 上创建以autossh 用户权限调用autossh 的service 文件。将下面文本写入到文件/lib/systemd/system/autossh.service，并设置权限为644：
```

```
[Unit]
Description=Auto SSH Tunnel
After=network-online.target
[Service]
User=autossh
Type=simple
ExecStart=/bin/autossh -p 22 -M 6777 -NR '*:6766:localhost:22' usera@a.site -i /home/autossh/.ssh/id_rsa
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=always
[Install]
WantedBy=multi-user.target
WantedBy=graphical.target
```
在B 上让network-online.target 生效：

```
B $ systemctl enable NetworkManager-wait-online
```
> 如果你使用systemd-networkd，你需要启用的服务则应当是systemd-networkd-wait-online 。

然后设置该服务自动启动：

```
B $ sudo systemctl enable autossh
```
如果你愿意，在这之后可以立刻启动它：

```
B $ sudo systemctl start autossh
```
然后你可以在A 上使用这条反向隧道穿透B 所在的NAT SSH 连接到B：

```
A $ ssh -p 6766 userb@localhost
```
或者是在C 上直接穿透两层NAT SSH 连接到B：

```
C $ ssh -p 6766 userb@a.site
```
如果你对SSH 足够熟悉，你可以利用这条隧道做更多的事情，例如你可以在反向连接时指定动态端口转发：

```
C $ ssh -p 6766 -qngfNTD 7677 userb@a.site
```
假设C 是你家中的电脑，A 是你的VPS，B 是你公司的电脑。如果你这样做了，那么为浏览器设置端口为7677 的sock4 本地（localhost）代理后，你就可以在家里的浏览器上看到公司内网的网页。


参考：
1. http://arondight.me/2016/02/17/%E4%BD%BF%E7%94%A8SSH%E5%8F%8D%E5%90%91%E9%9A%A7%E9%81%93%E8%BF%9B%E8%A1%8C%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F/
2. https://blog.windrunner.me/sa/reverse-ssh.html