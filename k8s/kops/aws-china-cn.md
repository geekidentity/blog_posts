---
categories: k8s

tags: 
  - k8s
  - kubernetes
  - kops

title: 如何在AWS中国区使用kops

date: 2018-03-15

---
# 如何在AWS中国区使用kops

## 入门

​    Kops过去只支持Google Cloud DNS和Amazon Route53配置kubernetes集群。 但是从1.6.2添加了gossip，因此可以在没有这些DNS提供商的情况下配置群集。 感谢gossip，从[1.7](https://github.com/kubernetes/kops/blob/master/docs/releases/1.7-NOTES.md)开始正式支持在AWS中国地区[没有Route53](http://docs.amazonaws.cn/en_us/aws/latest/userguide/unsupported.html)的情况下提供功能齐全的kubernetes群集。 目前只有cn-north-1（北京区）可用，但新的地区[即将推出](https://aws.amazon.com/about-aws/global-infrastructure/)。

以下大多数配置集群的过程与[在AWS中使用kops的指南相同](https://github.com/kubernetes/kops/blob/master/docs/aws.md)。 所以下面主要描述在中国区使用的不同点，相似的部分将被省略。

### [Install kops](https://github.com/kubernetes/kops/blob/master/docs/aws.md#install-kops)

### [Install kubectl](https://github.com/kubernetes/kops/blob/master/docs/aws.md#install-kubectl)

### [Setup your environment](https://github.com/kubernetes/kops/blob/master/docs/aws.md#setup-your-environment)

#### AWS

当执行 aws configure 时，请记住将默认区域名称设置为正确的名称，例如，cn-north-1。

```
AWS Access Key ID [None]:
AWS Secret Access Key [None]:
Default region name [None]:
Default output format [None]:
```

并正确导出它。

```bash
export AWS_REGION=$(aws configure get region)
```

## [配置 DNS](https://github.com/kubernetes/kops/blob/master/docs/aws.md#configure-dns)

正如一开始提到的那样，通过让群集名以.k8s.loca	l结尾，可以很容易地创建一个基于gossip的群集。 我们将在下面采用这个技巧。 其余部分可以安全地跳过。

## [Testing your DNS setup](https://github.com/kubernetes/kops/blob/master/docs/aws.md#testing-your-dns-setup)

由于gossip，这部分也可以安全地跳过。

## [群集状态存储](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#cluster-state-storage)

由于我们正在AWS中国地区部署集群，因此我们需要在AWS中国地区创建一个专用S3存储桶。

```bash
aws s3api create-bucket --bucket prefix-example-com-state-store --create-bucket-configuration LocationConstraint=$AWS_REGION
```

## [创建你的第一个集群](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#creating-your-first-cluster)

### 确保你有一个可以正常访问互联网的VPC

首先，我们必须解决与中国以外的互联网的缓慢和不稳定的连接，否则以下流程将无法工作。 一种方法是建立一个NAT实例，通过一些可靠的连接路由流量。 细节将不在这里讨论。

### 准备 kops ami

我们必须建立自己的AMI，因为AWS中国地区没有官方的kops ami。 有两种方法可以实现。

#### ImageBuilder

首先，在一个私有子网中启动一个快速稳定访问互联网的实例。

因为实例在私有子网中启动，所以我们需要确保它可以通过VPN或bastion使用私有IP进行连接。

```
SUBNET_ID=<subnet id> # 私有子网ID
SECURITY_GROUP_ID=<security group id>
KEY_NAME=<key pair name on aws>

AMI_ID=$(aws ec2 describe-images --filters Name=name,Values=debian-jessie-amd64-hvm-2016-02-20-ebs --query 'Images[*].ImageId' --output text)
INSTANCE_ID=$(aws ec2 run-instances --image-id $AMI_ID --instance-type m3.medium --key-name $KEY_NAME --security-group-ids $SECURITY_GROUP_ID --subnet-id $SUBNET_ID --no-associate-public-ip-address --query 'Instances[*].InstanceId' --output text)
aws ec2 create-tags --resources ${INSTANCE_ID} --tags Key=k8s.io/role/imagebuilder,Value=1
```

现在按照kube-deploy中的[ImageBuilder](https://github.com/kubernetes/kube-deploy/tree/master/imagebuilder)文档构建镜像。

```bash
go get k8s.io/kube-deploy/imagebuilder
cd ${GOPATH}/src/k8s.io/kube-deploy/imagebuilder

sed -i '' "s|publicIP := aws.StringValue(instance.PublicIpAddress)|publicIP := aws.StringValue(instance.PrivateIpAddress)|" pkg/imagebuilder/aws.go
make

# If the keypair specified is not `$HOME/.ssh/id_rsa`, `aws.yaml` need to be modified to add the full path to the private key.
echo 'SSHPrivateKey: "/absolute/path/to/the/private/key"' >> aws.yaml

${GOPATH}/bin/imagebuilder --config aws.yaml --v=8 --publish=false --replicate=false --up=false --down=false
```

*注意*

imagebuilder可能会抱怨镜像在构建和执行失败后找不到。 但是从异常日志中，我们可以发现AMI已经被实际注册了。 尽管bootstrap-vz声称有这个问题，但这似乎新创建的AMI暂时处理不可用状态导致的。 [kubernetes/kube-deploy#293](https://github.com/kubernetes/kube-deploy/issues/293)。

等一分钟左右，AMI应该可以使用了。

#### 从另一个地区复制AMI

按[这个评论](https://github.com/kubernetes-incubator/kube-aws/pull/390#issue-212435055)在从其他区域复制kops镜像，例如：`ap-southeast-1`。

#### 获取AMI ID

无论使用何种方式，我们最终得到AMI，例如，k8s-1.7-debian-jessie-amd64-hvm-ebs-2017-09-09。

### [准备本地环境](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#prepare-local-environment)

设置几个环境变量。

```bash
export NAME=example.k8s.local
export KOPS_STATE_STORE=s3://prefix-example-com-state-store
```

### [创建集群配置](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#create-cluster-configuration)

我们需要注意哪些可用区域可供我们使用。 AWS中国（北京）地区只有两个可用区域。 它会有[同样的问题](https://github.com/kubernetes/kops/issues/3088)，像其他地区少于三个AZ，在两个AZ上没有真正的HA支持。 但可以[添加更多主节点](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws-china.md#add-more-master-nodes)来提高一个AZ的可靠性。

```bash
aws ec2 describe-availability-zones
```

下面是 create cluster  命令，它将在现有的VPC中创建一个完整的内部群集。 以下命令将生成群集配置，但不会开始构建它（不会启动服务器）。 确保在创建群集之前生成了SSH密钥对。

```bash
VPC_ID=<vpc id>
VPC_NETWORK_CIDR=<vpc network cidr> # e.g. 172.30.0.0/16
AMI=<owner id/ami name> # e.g. 123456890/k8s-1.7-debian-jessie-amd64-hvm-ebs-2017-09-09

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

### [自定义群集配置](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#prepare-local-environment)

现在我们有一个集群配置文件，我们通过编辑描述来调整子网配置以重用[共享子网](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/run_in_existing_vpc.md#shared-subnets)。

```bash
kops edit cluster $NAME
```

然后更改相应的子网以指定该子网id并除去cidr，例如

```yaml
spec:
  subnets:
  - id: subnet-12345678
    name: cn-north-1a
    type: Private
    zone: cn-north-1a
  - id: subnet-87654321
    name: utility-cn-north-1a
    type: Utility
    zone: cn-north-1a
```

我们在这里可以采用的另一个调整是添加一个docker配置，将镜像更改为中国的官方注册表镜像。 这将增加从docker hub中拉取镜像的稳定性和下载速度。

```yaml
spec:
  docker:
    logDriver: ""
    registryMirrors:
    - https://registry.docker-cn.com
```

请注意，这面镜子请注意，这面可能**不适合某些情况。 只要它与docker API兼容，它就可以被任何其他注册表镜像替换。不适合某些情况。 只要它与docker API兼容，它就可以被任何其他注册表镜像替换。

### [Build the Cluster](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#build-the-cluster)

### [Use the Cluster](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#use-the-cluster)

### [Delete the Cluster](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#delete-the-cluster)

## [What's next?](https://github.com/kubernetes/kops/blob/de9e728d5e2cc9229ecd2b145da46ca6c7eeb104/docs/aws.md#whats-next)

### 添加更多主结点

#### 一个 AZ

为了达到这个目的，我们可以添加更多参数给`kops create cluster`。

```yaml
  --master-zones ${AWS_REGION}a --master-count 3 \
  --zones ${AWS_REGION}a --node-count 2 \
```

#### 两个 AZ

```yaml
  --master-zones ${AWS_REGION}a,${AWS_REGION}b --master-count 3 \
  --zones ${AWS_REGION}a,${AWS_REGION}b --node-count 2 \
```

请注意，当其中一个AZ宕机时，这仍有50％的可能性导致集群不可用。

### 离线模式

这是一个天真的，未完成的尝试，以最小化互联网需求的方式配置群集，因为即使使用某种代理或VPN，它仍然不是那么快，而且总是比从S3下载要贵得多。

```bash
## Setup vars

KUBERNETES_VERSION=$(curl -fsSL --retry 5 "https://dl.k8s.io/release/stable.txt")
KOPS_VERSION=$(curl -fsSL --retry 5 "https://api.github.com/repos/kubernetes/kops/releases/latest" | grep 'tag_name' | cut -d\" -f4)
ASSET_BUCKET="some-asset-bucket"
ASSET_PREFIX=""

# Please note that this filename of cni asset may change with kubernetes version
CNI_FILENAME=cni-0799f5732f2a11b329d9e3d51b9c8f2e3759f2ff.tar.gz


export KOPS_BASE_URL=https://s3.cn-north-1.amazonaws.com.cn/$ASSET_BUCKET/kops/$KOPS_VERSION/
export CNI_VERSION_URL=https://s3.cn-north-1.amazonaws.com.cn/$ASSET_BUCKET/kubernetes/network-plugins/$CNI_FILENAME

## Download assets

KUBERNETES_ASSETS=(
  network-plugins/$CNI_FILENAME
  release/$KUBERNETES_VERSION/bin/linux/amd64/kube-apiserver.tar
  release/$KUBERNETES_VERSION/bin/linux/amd64/kube-controller-manager.tar
  release/$KUBERNETES_VERSION/bin/linux/amd64/kube-proxy.tar
  release/$KUBERNETES_VERSION/bin/linux/amd64/kube-scheduler.tar
  release/$KUBERNETES_VERSION/bin/linux/amd64/kubectl
  release/$KUBERNETES_VERSION/bin/linux/amd64/kubelet
)
for asset in "${KUBERNETES_ASSETS[@]}"; do
  dir="kubernetes/$(dirname "$asset")"
  mkdir -p "$dir"
  url="https://storage.googleapis.com/kubernetes-release/$asset"
  wget -P "$dir" "$url"
  [ "${asset##*.}" != "gz" ] && wget -P "$dir" "$url.sha1"
  [ "${asset##*.}" == "tar" ] && wget -P "$dir" "${url%.tar}.docker_tag"
done

KOPS_ASSETS=(
  "images/protokube.tar.gz"
  "linux/amd64/nodeup"
  "linux/amd64/utils.tar.gz"
)
for asset in "${KOPS_ASSETS[@]}"; do
  kops_path="kops/$KOPS_VERSION/$asset"
  dir="$(dirname "$kops_path")"
  mkdir -p "$dir"
  url="https://kubeupv2.s3.amazonaws.com/kops/$KOPS_VERSION/$asset"
  wget -P "$dir" "$url"
  wget -P "$dir" "$url.sha1"
done

## Upload assets

## Get default S3 multipart_threshold

AWS_S3_DEFAULT_MULTIPART_THRESHOLD=$(aws configure get default.s3.multipart_threshold)

if [ ! -n "$AWS_S3_DEFAULT_MULTIPART_THRESHOLD" ]; then
  AWS_S3_DEFAULT_MULTIPART_THRESHOLD=8MB
fi

## Set multipart_threshold to 1024MB to prevent Etag not returns MD5 when upload multipart

aws configure set default.s3.multipart_threshold 1024MB

aws s3api create-bucket --bucket $ASSET_BUCKET --create-bucket-configuration LocationConstraint=$AWS_REGION
for dir in "kubernetes" "kops"; do
  aws s3 sync --acl public-read "$dir" "s3://$ASSET_BUCKET/$ASSET_PREFIX$dir"
done

aws configure set default.s3.multipart_threshold $AWS_S3_DEFAULT_MULTIPART_THRESHOLD
```

创建群集时，将这些参数添加到命令行。

```bash
  --kubernetes-version https://s3.cn-north-1.amazonaws.com.cn/$ASSET_BUCKET/kubernetes/release/$KUBERNETES_VERSION
```

现在，通过kops和kubernetes设置集群所需的大部分资源将从指定的S3存储桶下载，但pause-amd64，dns相关等images除外。由于托管在gcr.io上，因此Docker Hub镜像里没有这些images。如果不能科学上网，会有一些问题。

### Assets API

它没有经过测试，因为当作者在AWS中国区域尝试提供集群时，这种方法只是PR。这是实现离线模式的官方方式，应该优于以前的天真尝试。