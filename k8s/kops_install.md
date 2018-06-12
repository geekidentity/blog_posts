# 使用 kops 在 AWS 部署 Kubernetes 集群

​    [kops](https://github.com/kubernetes/kops) 是官方推荐的工具，用来在 AWS 生产环境中，快速地部署 Kubernetes 集群。	





使用 kops 在 AWS 部署 Kubernetes 集群

## 环境准备

在 1.6.2 版本之前，通过 kops 部署 K8s 集群，需要使用 AWS 的 Route53 来提供 DNS 服务的功能。但从 1.6.2 版本开始，kops 支持部署基于 gossip 的集群，不再依赖 Route53，这让部署操作变得更加简单，在中国区使用更加方便。

在部署集群之前，需要安装 kubectl、kops 和 awscli 这些工具，下面是安装步骤：

### 安装kops

```bash
wget -O kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

### 安装kubectl

```bash
wget -O kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

由于众所周知的原因，kubectl的安装需要科学上网，可以先下载下来，再传到服务器上去。

### 安装awscli

```bash
pip install awscli
```



### 配置 AWS 账号

为了在AWS中构建集群，我们将为kops创建一个专用的IAM用户。 该用户需要API凭证才能使用kops。 使用AWS控制台创建用户和凭证。

```bash
$ aws configure
AWS Access Key ID [None]: <your-accesskeyID>
AWS Secret Access Key [None]: <your-secretAccessKey>
Default region name [None]: cn-north-1
Default output format [None]: json
```

添加环境变量

```bash
export AWS_REGION=$(aws configure get region)
```

kops用户将需要以下IAM权限才能正常工作（中国区Route53无法使用，在下面授权的时候会报错，请忽略）：

```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

创建kops IAM用户：

```bash
aws iam create-group --group-name kops

aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name kops
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name kops

aws iam create-user --user-name kops

aws iam add-user-to-group --user-name kops --group-name kops
```

为`kops`用户创建密钥：

```bash
aws iam create-access-key --user-name kops
```

上面的命令会返回`kops`用户的`AccessKeyID`和`SecretAccessKey`，请务必记录下来。

接着我们就可以更新`awscli`的配置，让它使用新创建的`kops`用户的密钥，同时将密钥导入环境变量：

```bash
# configure the aws client to use your new IAM user
aws configure           # Use your new access and secret key here
aws iam list-users      # you should see a list of all your IAM users here

# Because "aws configure" doesn't export these vars for kops to use, we export them now
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)
```

最后是生成 SSH 密钥：

```bash
$ ssh-keygen
```

## 配置DNS

通过让群集名以 .k8s.local 结尾，可以很容易地创建一个基于 gossip 的群集。这里我们将集群命名为`cluster.k8s.local`：

```bash
$ export NAME=cluster.k8s.local
```

## 集群状态存储

创建一个 S3 bucket，用户存储集群的数据，例如，这里我们将这个 bucket 命名为`cluster-k8s-local-state-store`

```bash
$ export KOPS_STATE_STORE=s3://cluster-k8s-local-state-store
$ aws s3api create-bucket --bucket ${NAME}-state-store --create-bucket-configuration LocationConstraint=$AWS_REGION
```

## 创建集群

下面的命令会创建集群的配置文件，并不会真正地创建集群：

```bash
kops create cluster \
    --zones ${AWS_REGION}a \
    --vpc ${VPC_ID} \
    --network-cidr ${VPC_NETWORK_CIDR} \
    --image ${AMI} \
    --associate-public-ip=false \
    --api-loadbalancer-type internal \
    --topology private \
    --networking weave \
    ${NAME}
```

在创建集群之前，可以检查集群的配置文件是否正确：

```bash
$ kops edit cluster ${NAME}
```

如果确认没问题，就可以使用下面的命令创建集群：

```bash
$ kops update cluster ${NAME} --yes
```

参考：

http://senlinzhan.github.io/2018/01/11/k8s-on-aws/

[Installing Kubernetes on AWS with kops](https://kubernetes.io/docs/getting-started-guides/kops/)

http://blog.csdn.net/ljxfblog/article/details/38396421