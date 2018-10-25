---
categories: k8s

tags: 
  - k8s
  - kubernetes
  - kops

title: 在AWS中国区使用kops安装k8s完全指南

date: 2018-04-02
---

因为一些众所周知的原因，AWS中国区并没有k8s集群，因此我们需要自己安装k8s。而k8s官方提供了一个工具[kops](https://github.com/geekidentity/kops)来帮助我们快速地在AWS上创建k8s集群。下面是使用kops在AWS创建集群的详细过程。

## 安装kops (Binaries)

我们建议使用一台低配服务器作为k8s的管理机，在上面安装kops等管理工具。

从github上下载已经编译好的二进制文件

```bash
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

## 安装其它依赖

### kubectl

kubectl是管理和操作Kubernetes集群的CLI工具。

从kubernetes官方kubectl获取发布版本：

```bash
wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 安装AWS CLI 工具

awscli是用Python写的，安装Python和pip后直接运行下面命令就可以了。

```bash
pip install awscli
```

## 创建账号

在 1.6.2 版本之前，通过 kops 部署 K8s 集群，需要使用 AWS 的 Route53 来提供 DNS 服务的功能。但从 1.6.2 版本开始，kops 支持部署基于 gossip 的集群，不再依赖 Route53，这让部署操作变得更加简单。

配置AWS 账号，使用该账号为kops创建专用账号：

```bash·
$ aws configure
AWS Access Key ID [None]: <your-accesskeyID>
AWS Secret Access Key [None]: <your-secretAccessKey>
Default region name [None]: cn-north-1
Default output format [None]: json
```

为了使用 kops 部署集群，还需要为 kops 创建一个 IAM 用户`kops`，并分配相应的权限：

```bash
$ aws iam create-group --group-name kops
$ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
$ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
$ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
$ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
$ aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops
$ aws iam create-user --user-name kops
$ aws iam add-user-to-group --user-name kops --group-name kops
```

为`kops`用户创建密钥：

```bash
$ aws iam create-access-key --user-name kops
```

上面的命令会返回`kops`用户的`AccessKeyID`和`SecretAccessKey`。接着我们就可以更新`awscli`的配置，让它使用新创建的`kops`用户的密钥：

```bash
$ aws configure
AWS Access Key ID [None]: <accesskeyID-of-kops-user>
AWS Secret Access Key [None]: <secretAccessKey-of-kops-user>
Default region name [None]: cn-north-1
Default output format [None]: json
```

同时还需要将`kops`用户的密钥导出到命令行的环境变量：

```bash
$ export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
$ export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
$ export AWS_REGION=$(aws configure get region)
```

最后是生成 SSH 密钥：

```bash
$ ssh-keygen
```

## 配置 S3

需要注意，为了让 kops 创建基于 gossip 的集群，集群的命名需要使用`.k8s.local`作为后缀，例如，这里我们将集群命名为`cluster.k8s.local`：

```bash
$ export NAME=cluster.k8s.local
```

接着创建一个 S3 bucket，用户存储集群的数据，例如，这里我们将这个 bucket 命名为`cluster.k8s.local-state.ym`：

```bash
$ aws s3api create-bucket --bucket ${NAME}-state-store --create-bucket-configuration LocationConstraint=$AWS_REGION
$ export KOPS_STATE_STORE=s3://cluster.k8s.local-state-store
```

## 准备kops ami

我们必须建立自己的AMI，因为AWS中国地区没有官方的kops ami。

## 创建集群

下面的命令会创建集群的配置文件，并不会真正地创建集群：

注意：kops-1.8.1不支持中国宁夏区，只支持北京区，	

```bash
$ kops create cluster \

     --name=${NAME} \
     --image=ami-089b06f993df09d53 \
     --zones=cn-north-1a \
     --master-count=1 \
     --master-size="t2.micro" \
     --node-count=1 \
     --node-size="t2.micro"  \
     --vpc=<your vpc id> \
     --subnets=<stringSlice> \
     --networking=calico \
     --ssh-public-key="~/.ssh/id_rsa.pub"
```

对于网络模型，使用calico，因为在线上都会自己进行网络规划，当使用k8s默认的kubenet时，k8s会修改AWS路由表，这意味着k8s需要有自己的路由表所有需要有自己的子网，如果在生产环境已经做好了网络规划，使用指定subnet，k8s网络将无法正常运行。

在创建集群之前，可以检查集群的配置文件是否正确：

```bash
$ kops edit cluster ${NAME}
```

在AWS上我们通常使用自己的密钥连接服务器　　

```yaml
...
spec:
  sshKeyName: <your ssh key name>
...
```

因为一些网站被墙，因此建议使用[代理](https://github.com/kubernetes/kops/blob/master/docs/http_proxy.md)搭建集群。

```yaml
...
spec:
  egressProxy:
    httpProxy:
      host: http-proxy
      port: port
    excludes: amazonaws.com.cn,amazonaws.cn,aliyun.cn,aliyuncs.com

...
```

还可以指定docker版本

```yaml
...
spec:
    docker:
        logDriver: json-file
        version: 17.03.2-ce
...
```

如果确认没问题，就可以使用下面的命令创建集群：

```bash
$ kops update cluster ${NAME} --yes
```

​        创建集群之后，需要一段时间等待集群的初始化，等待集群起来之后，可以验证集群的状况：

```bash
$ kops validate cluster
```

　　前面已经安装好了`kubectl`工具，这里也可以使用`kubectl`检查集群状况：

```bash
$ kubectl get nodes
```

## 销毁集群

　　在销毁集群之前，需要先确认一下 kops 会删除哪些资源：

```bash
$ kops delete cluster --name ${NAME}
```

　　如果确认没问题，就可以真正删除集群：

```bash
$ kops delete cluster --name ${NAME} --yes
```

## 参考资料

1. [kops install](https://github.com/kubernetes/kops/blob/master/docs/install.md)
2. [How to use kops in AWS China Region](https://github.com/kubernetes/kops/blob/master/docs/aws-china.md)
3. [使用 kops 在 AWS 部署 Kubernetes 集群](http://senlinzhan.github.io/2018/01/11/k8s-on-aws/)