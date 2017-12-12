---
categories: linux
tags:
  - Linux
  - Hadoop
  - SSH

title: SSH免密码登录
date: 2016-12-18
---

# SSH免密码登录
Hadoop要运行在多个服务器上，因些SSH免密码登录必不可少，所有整理一下，共享。

SSH无密码登录使用公钥与私钥。linux下可以用用ssh-keygen生成公钥/私钥对，以CentOS为例。
假设有机器A(192.168.1.1)，B(192.168.1.2)。现A通过SSH免密码登录到B。


# 在A机下生成公钥/私钥对。

```
$ ssh-keygen -t rsa -P ''
```

-P表示密码，-P '' 就表示空密码，也可以不用-P参数，这样就要三车回车，用-P就一次回车。
在当前用户下生成.ssh目录，.ssh下有id_rsa和id_rsa.pub。
# 公钥复制

把A机下的id_rsa.pub复制到B机下，在B机的~/.ssh/authorized_keys文件里，我用scp复制。

```
$ scp ~/.ssh/id_rsa.pub user@192.168.1.2:/home/user/id_rsa.pub
```

由于还没有免密码登录的，所以要输入密码。
# 添加授权
B机把从A机复制的id_rsa.pub添加到.ssh/authorzied_keys文件里。

```
$ cat id_rsa.pub >> .ssh/authorized_keys
$ chmod 600 .ssh/authorized_keys
```

authorized_keys的权限要是600
# 测试登录
A机登录B机。

```
$ ssh 192.168.1.2
```

第一次登录是时要你输入yes。
现在A机可以无密码登录B机了。

# 小结
登录的机子持有私钥，被登录的机子要有登录机子的公钥。这个公钥/私钥对一般在私钥宿主机产生。上面是用RSA算法的公钥/私钥对，当然也可以用DSA(对应的文件是id_dsa，id_dsa.pub)

想让A，B机无密码互登录，那B机以上面同样的方式配置即可。
