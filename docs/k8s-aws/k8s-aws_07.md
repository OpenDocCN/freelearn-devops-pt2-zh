# 第七章：一个适合生产的集群

在上一章中，我们花了一些时间思考规划 Kubernetes 集群的框架。希望对你来说应该很清楚，构建集群时需要根据正在运行的系统的要求做出许多决策。

在本章中，我们将采取更加实际的方法来解决这个问题。我们将不再试图涵盖我们可以使用的众多选项，而是首先做出一些选择，然后构建一个完全功能的集群，作为许多不同用例的基础配置。

在本章中，我们将涵盖以下主题：

+   Terraform

+   准备节点镜像和节点组

+   配置附加组件

# 构建一个集群

本章中包含的信息只是构建和管理集群的一种可能方式。构建 Kubernetes 集群时，有许多选择要做，几乎可以选择同样多的工具。出于本章的目的，我选择使用可以简单说明构建集群过程的工具。如果您或您的团队更喜欢使用不同的工具，那么本章中概述的概念和架构将很容易转移到其他工具上。

在这一章中，我们将以一种更适合生产工作负载的方式启动我们的集群。我们在这里所做的大部分工作都将与第三章“云端之手”中的内容相似，但我们将在两个关键方面进一步完善我们在那里概述的流程。首先，在构建依赖的基础设施时，能够快速部署新的基础设施实例是非常重要的，而且要能够重复进行。我们希望能够做到这一点，因为这样可以简单地测试我们想要对基础设施进行的变更，而且是无风险的。通过自动化 Kubernetes 集群的配置，我们实现了之前讨论过的不可变基础设施模式。我们可以快速部署一个替代集群，然后在将工作负载迁移到新集群之前进行测试，而不是冒险升级或更改我们的生产基础设施。

为了实现这一点，我们将使用 Terraform 基础设施配置工具与 AWS 进行交互。Terraform 允许我们使用一种类似编程语言的方式将我们的基础设施定义为代码。通过将基础设施定义为代码，我们能够使用诸如版本控制之类的工具，并遵循其他软件开发实践来管理我们基础设施的演变。

在本章中，我们将做出许多关于在 AWS 上运行的 Kubernetes 集群应该是什么样子以及如何管理的决定。对于本章和我们将要处理的示例，我心中有以下要求。

+   **说明性**：我们将看到符合平均生产用例要求的 Kubernetes 集群是什么样子。这个集群反映了我在设计用于真实生产的 Kubernetes 集群时所做的决定。为了使本章尽可能清晰易懂，我尽量保持集群及其配置尽可能简单。

+   **灵活性**：我们将创建一些您可以视为模板并添加或更改以满足您需求的东西。

+   **可扩展性**：无论您设计 Kubernetes 集群（或者任何基础设施），都应该考虑您现在所做的决定可能会阻止您以后扩展或扩展该基础设施。

显然，当您构建自己的集群时，您应该对自己的需求有一个更加具体的想法，这样您就能够根据自己的需求定制您的集群。我们将在这里构建的集群将是任何生产就绪系统的绝佳起点，然后您可以根据需要自定义和添加。

本章中的许多配置都已经被缩短了。您可以在[`github.com/PacktPublishing/Kubernetes-on-AWS/tree/master/chapter07`](https://github.com/PacktPublishing/Kubernetes-on-AWS/tree/master/chapter07)查看本章中使用的完整配置。

# 开始使用 Terraform

Terraform 是一个命令行工具，您可以在工作站上运行它来对基础设施进行更改。Terraform 是一个单一的二进制文件，只需安装到您的路径上即可。

您可以从[`www.terraform.io/downloads.html`](https://www.terraform.io/downloads.html)下载 Terraform，支持六种不同的操作系统，包括 macOS、Windows 和 Linux。下载适用于您操作系统的 ZIP 文件，解压缩，然后将 Terraform 二进制文件复制到您的路径上。

Terraform 使用扩展名为`.tf`的文件来描述您的基础架构。因为 Terraform 支持在许多不同的云平台上管理资源，它可以包含相关提供者的概念，这些提供者根据需要加载，以支持不同云提供商提供的不同 API。

首先，让我们配置 AWS Terraform 提供者，以便准备构建一个 Kubernetes 集群。创建一个新目录来保存 Kubernetes 集群的 Terraform 配置，然后创建一个文件，在其中我们将配置 AWS 提供者，如下所示的代码：

```
aws.tf
provider "aws" { 
  version = "~> 1.0" 

  region = "us-west-2" 
} 
```

保存文件，然后运行以下命令：

```
terraform.init
Initializing provider plugins... 
- Checking for available provider plugins on https://releases.hashicorp.com... 
- Downloading plugin for provider "aws" (1.33.0)... 
Terraform has been successfully initialized! 
```

当您使用支持的提供者时，Terraform 可以发现并下载所需的插件。请注意，我们已经配置了提供者的 AWS 区域为`us-west-2`，因为这是我们在本例中将要启动集群的区域。

为了让 Terraform 与 AWS API 通信，您需要为 AWS 提供者提供一些凭据。我们在第三章中学习了如何获取凭据，*云的探索*。如果您遵循了第三章中的建议，并使用`aws configure`命令设置了您的凭据，那么 Terraform 将从您的本地配置文件中读取默认凭据。

另外，Terraform 可以从`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`环境变量中读取 AWS 凭据，或者如果您在 EC2 实例上运行 Terraform，它可以使用 EC2 实例角色提供的凭据。

也可以通过在 AWS 提供者块中内联添加`access_key`和`secret_key`参数来静态配置凭据，但我并不真的推荐这种做法，因为这样会使得将配置检入版本控制系统变得更加困难。

默认情况下，Terraform 使用名为`terraform.tfstate`的本地文件来跟踪您基础架构的状态。这样可以跟踪自上次运行 Terraform 以来对配置所做的更改。

如果你将是唯一管理基础架构的人，那么这可能是可以接受的，但你需要安全地备份状态文件。它应被视为敏感信息，如果丢失，Terraform 将无法正常运行。

如果您正在使用 AWS，我建议使用 S3 作为后端。您可以在 Terraform 文档中阅读如何设置这一点，网址为[`www.terraform.io/docs/backends/types/s3.html`](https://www.terraform.io/docs/backends/types/s3.html)。如果配置正确，S3 存储是非常安全的，如果您正在团队中工作，那么您可以利用 DynamoDB 表作为锁，以确保多个 Terraform 实例不会同时运行。如果您想使用这个功能，请在`backend.tf`文件中设置配置，否则删除该文件。

# 变量

Terraform 允许我们定义变量，以使我们的配置更具重用性。如果以后您想要将您的配置用作模块来定义多个集群，这将特别有用。我们不会在本章中涵盖这一点，但我们可以遵循最佳实践并定义一些关键变量，以便您可以简单地塑造集群以满足您的需求。

创建一个`variables.tf`文件来包含项目中的所有变量是标准的。这很有帮助，因为它作为关于如何控制您的配置的高级文档。

正如你所看到的，选择变量的描述性名称并添加可选的描述字段之间，整个文件都相当自解释。因为我为每个变量提供了默认值，所以我们可以在不传递这些变量的任何值的情况下运行 Terraform，如下面的代码所示：

```
variables.tf
variable "cluster_name" { 
  default = "lovelace" 
} 

variable "vpc_cidr" { 
  default     = "10.1.0.0/16" 
  description = "The CIDR of the VPC created for this cluster" 
} 

variable "availability_zones" { 
  default     = ["us-west-2a","us-west-2b"] 
  description = "The availability zones to run the cluster in" 
} 

variable "k8s_version" { 
  default     = "1.10" 
  description = "The version of Kubernetes to use" 
} 
```

# 网络

我们将首先创建一个配置文件来描述我们的 Kubernetes 集群的网络设置。您可能会注意到这个网络的设计，因为它与我们在第三章中手动创建的网络非常相似，但是增加了一些内容，使其更适合生产环境。

Terraform 配置文件可以用注释进行文档化，为了更好地说明这个配置，我提供了一些注释形式的评论。你会注意到它们被`/*`和`*/`包围着。

为了支持高可用性，我们将为多个可用区域创建子网，如下面的代码所示。在这里，我们使用了两个，但如果您想要更高的弹性，您可以轻松地将另一个可用区域添加到`availability_zones`变量中：

```
networking.tf
/*  Set up a VPC for our cluster. 
*/resource "aws_vpc" "k8s" { 
  cidr_block           = "${var.vpc_cidr}" 
  enable_dns_hostnames = true 

  tags = "${ 
    map( 
     "Name", "${var.cluster_name}", 
     "kubernetes.io/cluster/${var.cluster_name}", "shared", 
    ) 
  }" 
} 

/*  In order for our instances to connect to the internet 
  we provision an internet gateway.*/ 
resource "aws_internet_gateway" "gateway" { 
  vpc_id = "${aws_vpc.k8s.id}" 

  tags { 
    Name = "${var.cluster_name}" 
  } 
} 

/*  For instances without a Public IP address we will route traffic  
  through a NAT Gateway. Setup an Elastic IP and attach it. 

  We are only setting up a single NAT gateway, for simplicity. 
  If the availability is important you might add another in a  
  second availability zone. 
*/ 
resource "aws_eip" "nat" { 
  vpc        = true 
  depends_on = ["aws_internet_gateway.gateway"] 
} 

resource "aws_nat_gateway" "nat_gateway" { 
  allocation_id = "${aws_eip.nat.id}" 
  subnet_id     = "${aws_subnet.public.*.id[0]}" 
} 
```

我们将为我们集群使用的每个可用区域提供两个子网。一个公共子网，它可以直接连接到互联网，Kubernetes 将在其中提供可供互联网访问的负载均衡器。还有一个私有子网，Kubernetes 将用它来分配给 pod 的 IP 地址。

因为私有子网中可用的地址空间将是 Kubernetes 能够启动的 pod 数量的限制因素，所以我们为其提供了一个大的地址范围，其中有 16382 个可用的 IP 地址。这应该为我们的集群提供一些扩展空间。

如果您只打算运行对外部不可访问的内部服务，那么您可能可以跳过公共子网。您可以在本章的示例文件中找到完整的`networking.tf`文件。

# 计划和应用

Terraform 允许我们通过添加和更改定义基础设施的代码来逐步构建我们的基础设施。然后，如果您希望，在阅读本章时，您可以逐步构建您的配置，或者您可以使用 Terraform 一次性构建整个集群。

每当您使用 Terraform 对基础设施进行更改时，它首先会生成一个将要进行的更改的计划，然后应用这个计划。这种两阶段的操作在修改生产基础设施时是理想的，因为它可以让您在实际应用到集群之前审查将要应用的更改。

一旦您将网络配置保存到文件中，我们可以按照一些步骤安全地为我们的基础设施提供。

我们可以通过运行以下命令来检查配置中的语法错误：

```
terraform validate  
```

如果您的配置正确，那么不会有输出，但如果您的文件存在语法错误，您应该会看到一个解释问题的错误消息。例如，缺少闭合括号可能会导致错误，如`Error parsing networking.tf: object expected closing RBRACE got: EOF`。

一旦您确保您的文件已正确格式化为 Terraform，您可以使用以下命令为您的基础设施创建变更计划：

```
terraform plan -out k8s.plan  
```

该命令将输出一个摘要，显示如果运行此计划将对基础架构进行的更改。`-out`标志是可选的，但这是一个好主意，因为它允许我们稍后应用这些更改。如果您在运行 Terraform 计划时注意输出，那么您应该已经看到了这样的消息：

```
To perform exactly these actions, run the following command to apply:
terraform apply "k8s.plan"
```

当您使用预先计算的计划运行`terraform apply`时，它将进行在生成计划时概述的更改。您也可以运行`terraform plan`命令而不预先生成计划，但在这种情况下，它仍将计划更改，然后在应用更改之前提示您。

Terraform 计算基础架构中不同资源之间的依赖关系，例如，它确保在创建路由表和其他资源之前先创建 VPC。一些资源可能需要几秒钟才能创建，但 Terraform 会等待它们可用后再继续创建依赖资源。

如果您想删除 Terraform 在您的 AWS 帐户中创建的资源，只需从相关的`.tf`文件中删除定义，然后计划并应用您的更改。当您测试 Terraform 配置时，删除特定配置创建的所有资源可能很有用，以便测试从头开始配置基础架构。如果需要这样做，`terraform destroy`命令非常有用；它将从基础架构中删除在 Terraform 文件中定义的所有资源。但是，请注意，这可能导致关键资源被终止和删除，因此您不应该在运行中的生产系统上使用此方法。在删除任何资源之前，Terraform 将列出它们，然后询问您是否要删除它们。

# 控制平面

为了为我们的集群提供一个弹性和可靠的 Kubernetes 控制平面，我们将首次大幅偏离我们在第三章中构建的简单集群，*云端的追求*。

正如我们在第一章中所学到的，“谷歌的基础设施服务于我们其他人”，Kubernetes 控制平面的关键组件是支持 etcd 存储、API 服务器、调度程序和控制器管理器。如果我们想要构建和管理一个弹性的控制平面，我们需要跨多个实例管理这些组件，最好分布在多个可用区。

由于 API 服务器是无状态的，并且调度程序和控制器管理器具有内置的领导者选举功能，因此在 AWS 上运行多个实例相对简单，例如，通过使用自动扩展组。

运行生产级别的 etcd 略微棘手，因为在添加或删除节点时，应小心管理 etcd 以避免数据丢失和停机。在 AWS 上成功运行 etcd 集群是一项相当困难的任务，需要手动操作或复杂的自动化。

幸运的是，AWS 开发了一项服务，几乎消除了在配置 Kubernetes 控制平面时涉及的所有操作复杂性——Amazon EKS，或者使用全名，亚马逊弹性容器服务用于 Kubernetes。

通过 EKS，AWS 将代表您在多个可用区管理和运行组成 Kubernetes 控制平面的组件，从而避免任何单点故障。有了 EKS，您不再需要担心执行或自动化运行稳定 etcd 集群所需的操作任务。

我们应该牢记，使用 EKS 时，我们集群基础设施的关键部分现在由第三方管理。您应该对 AWS 能够比您自己的团队更好地提供弹性控制平面感到满意。这并不排除您设计集群以在控制平面故障时具有一定的抗性的可能性——例如，如果 kubelet 无法连接到控制平面，那么正在运行的容器将保持运行，直到控制平面再次可用。您应该确保您添加到集群中的任何其他组件都能以类似的方式应对临时停机。

EKS 减少了管理 Kubernetes 最复杂部分（控制平面）所需的工作量，从而减少了设计集群和维护集群所需的时间（和金钱）。此外，即使是规模适中的集群，EKS 服务的成本也明显低于在多个 EC2 实例上运行自己的控制平面的成本。

为了让 Kubernetes 控制平面管理 AWS 账户中的资源，你需要为 EKS 提供一个 IAM 角色，EKS 本身将扮演这个角色。

EKS 在你的 VPC 中创建网络接口，以允许 Kubernetes 控制平面与 kubelet 通信，从而提供**日志流**和**执行**等服务。为了控制这种通信，我们需要在启动时为 EKS 提供一个安全组。你可以在本章的示例文件中的`control_plane.tf`中找到用于配置控制平面的完整 Terraform 配置。

我们可以使用 Terraform 资源来查询 EKS 集群，以获取用于访问 Kubernetes API 的端点和证书颁发机构。

这些信息，结合 Terraform 的模板功能，允许我们生成一个`kubeconfig`文件，其中包含连接到 EKS 提供的 Kubernetes API 所需的信息。我们以后可以使用这个文件来配置附加组件。

如果你愿意，你也可以使用这个文件手动连接到集群，使用 kubectl 命令，可以通过将文件复制到默认位置`~/.kube/config`，或者通过`--kubeconfig`标志或`KUBECONFIG`环境变量传递其位置给 kubectl，如下面的代码所示：

`KUBECONFIG`环境变量在管理多个集群时非常有用，因为你可以通过分隔它们的路径轻松加载多个配置；例如：

将 KUBECONFIG 环境变量设置为`$HOME/.kube/config:/path/to/other/conf`。

```
kubeconfig.tpl 
apiVersion: v1 
kind: Config 
clusters: 
- name: ${cluster_name} 
  cluster: 
    certificate-authority-data: ${ca_data} 
    server: ${endpoint} 
users: 
- name: ${cluster_name} 
  user: 
    exec: 
      apiVersion: client.authentication.k8s.io/v1alpha1 
      command: aws-iam-authenticator 
      args: 
      - "token" 
      - "-i" 
      - "${cluster_name}" 
contexts: 
- name: ${cluster_name} 
  context: 
    cluster: ${cluster_name} 
    user: ${cluster_name} 
current-context: ${cluster_name} 
```

# 准备节点镜像

就像我们在第三章中所做的那样，*云端之手*，我们现在将为集群中的工作节点准备一个 AMI。但是，我们将通过**Packer**自动化这个过程。Packer 是一个在 AWS（和其他平台）上构建机器镜像的简单工具。

# 安装 Packer

就像 Terraform 一样，Packer 被分发为一个单一的二进制文件，只需将其复制到您的路径上。您可以在 Packer 网站上找到详细的安装说明[`www.packer.io/intro/getting-started/install.html`](https://www.packer.io/intro/getting-started/install.html)。

安装了 Packer 之后，您可以运行`packer version`来检查您是否已经正确地将其复制到您的路径上。

# Packer 配置

Packer 配置为一个 JSON 格式的配置文件，您可以在`ami/node.json`中看到。

这里的示例配置有三个部分。第一个是变量列表。在这里，我们使用变量来存储我们将在镜像中安装的重要软件的版本号。这将使得在将来可用时，构建和测试具有更新版本的 Kubernetes 软件的镜像变得简单。

配置的第二部分配置了构建器。Packer 允许我们选择使用一个或多个构建器来构建支持不同云提供商的镜像。由于我们想要构建一个用于 AWS 的镜像，我们使用了`amazon-ebs`构建器，它通过启动临时 EC2 实例然后从其根 EBS 卷的内容创建 AMI 来创建镜像（就像我们在第三章中遵循的手动过程，*云的到来*）。这个构建器配置允许我们选择我们的机器将基于的基础镜像；在这里，我们使用了官方的 Ubuntu 服务器镜像，一个可信的来源。构建器配置中的`ami-name`字段定义了输出镜像将被赋予的名称。我们已经包含了所使用的 Kubernetes 软件的版本和时间戳，以确保这个镜像名称是唯一的。拥有唯一的镜像名称让我们能够精确地定义在部署服务器时使用哪个镜像。

最后，我们配置了一个 provisioner 来安装我们的镜像所需的软件。Packer 支持许多不同的 provisioners，可以安装软件，包括诸如 Chef 或 Ansible 之类的完整配置管理系统。为了保持这个示例简单，我们将使用一个 shell 脚本来自动安装我们需要的软件。Packer 将上传配置的脚本到构建实例，然后通过 SSH 执行它。

我们只是使用一个简单的 shell 脚本，但如果您的组织已经在使用配置管理工具，那么您可能更喜欢使用它来安装您的镜像所需的软件，特别是因为它可以简单地包含您组织的基本配置。

在这个脚本中，我们正在安装我们的工作节点将需要加入 EKS 集群并正确运行的软件和配置，如下面的列表所示。在实际部署中，可能还有其他工具和配置，您希望除了这些之外添加。

+   **Docker**：Docker 目前是与 Kubernetes 一起使用的经过最全面测试和最常见的容器运行时

+   **kubelet**：Kubernetes 节点代理

+   **ekstrap**：配置 kubelet 连接到 EKS 集群端点

+   **aws-iam-authenticator**：允许节点使用节点的 IAM 凭据与 EKS 集群进行身份验证

我们使用以下代码安装这些元素：

```
install.sh
          #!/bin/bash
          set -euxo pipefail  
...
          # Install aws-iam-authenticator
curl -Lo /usr/local/bin/heptio-authenticator-aws https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.3.0/heptio-authenticator-aws_0.3.0_linux_amd64
chmod +x /usr/local/bin/heptio-authenticator-aws

apt-get install -y \
  docker-ce=$DOCKER_VERSION* \
  kubelet=$K8S_VERSION* \
  ekstrap=$EKSTRAP_VERSION*
# Cleanup
apt-get clean
rm -rf /tmp/*
   # Cleanup
   apt-get clean
   rm -rf /tmp/*
```

一旦您为 Packer 准备好配置，您可以使用`packer build`命令在您的 AWS 账户中构建 AMI，如下面的代码所示。这将启动一个临时的 EC2 实例。将新的 AMI 保存到您的账户中，并清理临时实例：

```
packer build node.json  
```

如果您的组织使用持续集成服务，您可能希望配置它以便定期构建您的节点镜像，以便获取基本操作系统的安全更新。

# 节点组

现在我们已经为集群中的工作节点准备好了一个镜像，我们可以设置一个自动扩展组来管理启动 EC2 实例，这些实例将组成我们的集群。

EKS 不会限制我们以任何特定的方式管理我们的节点，因此自动扩展组并不是管理集群中节点的唯一选项，但使用它们是管理集群中多个工作实例的最简单方式之一。

如果您想在集群中使用多种实例类型，您可以为您想要使用的每种实例类型重复启动配置和自动扩展组配置。在这个配置中，我们正在按需启动`c5.large`实例，但您应该参考第六章 *生产规划*，了解有关为您的集群选择适当实例大小的更多信息。

配置的第一部分设置了我们的实例要使用的 IAM 角色。这很简单，因为 AWS 提供了托管策略，这些策略具有 Kubernetes 所需的权限。`AmazonEKSWorkerNodePolicy`代码短语允许 kubelet 查询有关 EC2 实例、附加卷和网络设置的信息，并查询有关 EKS 集群的信息。`AmazonEKS_CNI_Policy`提供了`vpc-cni-k8s`网络插件所需的权限，以将网络接口附加到实例并为这些接口分配新的 IP 地址。`AmazonEC2ContainerRegistryReadOnly`策略允许实例从 AWS Elastic Container Registry 中拉取 Docker 镜像（您可以在第十章中了解更多关于使用此功能的信息，*管理容器镜像*）。我们还将手动指定一个策略，允许`kube2iam`工具假定角色，以便为在集群上运行的应用程序提供凭据，如下面的代码所示：

```
nodes.tf 
/* 
  IAM policy for nodes 
*/ 
data "aws_iam_policy_document" "node" { 
  statement { 
    actions = ["sts:AssumeRole"] 

    principals { 
      type        = "Service" 
      identifiers = ["ec2.amazonaws.com"] 
    } 
  } 
} 
... 

resource "aws_iam_instance_profile" "node" { 
  name = "${aws_iam_role.node.name}" 
  role = "${aws_iam_role.node.name}" 
} 
```

我们的工作节点在能够向 Kubernetes API 服务器注册之前，需要具有正确的权限。在 EKS 中，IAM 角色和用户之间的映射是通过向集群提交配置映射来配置的。

您可以在 EKS 文档中阅读有关如何将 IAM 用户和角色映射到 Kubernetes 权限的更多信息，网址为[`docs.aws.amazon.com/eks/latest/userguide/add-user-role.html`](https://docs.aws.amazon.com/eks/latest/userguide/add-user-role.html)。

Terraform 将使用我们在设置控制平面时生成的`kubeconfig`文件，通过 local-exec provisioner 使用`kubectl`将此配置提交到集群，如下面的`nodes.tf`继续的代码所示：

```
/* 
  This config map configures which IAM roles should be trusted by Kubernetes 
*/ 

resource "local_file" "aws_auth" { 
  content = <<YAML 
apiVersion: v1 
kind: ConfigMap 
metadata: 
  name: aws-auth 
  namespace: kube-system 
data: 
  mapRoles: | 
    - rolearn: ${aws_iam_role.node.arn} 
      username: system:node:{{EC2PrivateDNSName}} 
      groups: 
        - system:bootstrappers 
        - system:nodes 
YAML 
  filename = "${path.module}/aws-auth-cm.yaml" 
  depends_on = ["local_file.kubeconfig"] 

  provisioner "local-exec" { 
    command = "kubectl --kubeconfig=${local_file.kubeconfig.filename} apply -f ${path.module}/aws-auth-cm.yaml" 
  } 
} 
```

接下来，我们需要准备安全组来控制与我们的节点之间的网络流量。

我们将设置一些规则，以允许以下通信流，这些流对于我们的集群能够正常运行是必需的：

+   节点需要相互通信，用于集群内的 pod 和服务通信。

+   运行在节点上的 Kubelet 需要连接到 Kubernetes API 服务器，以便读取和更新有关集群状态的信息。

+   控制平面需要连接到端口`10250`上的 Kubelet API；这用于功能，如`kubectl exec`和`kubectl logs`。

+   为了使用 API 的代理功能将流量代理到 pod 和服务，控制平面需要连接到在集群中运行的 pod。在这个例子中，我们打开了所有端口，但是，例如，如果您只在您的 pod 上打开了非特权端口，那么您只需要允许流量到 1024 以上的端口。

我们使用以下代码设置这些规则。`nodes.tf`的代码如下：

```
 resource "aws_security_group" "nodes" { 
  name        = "${var.cluster_name}-nodes" 
  description = "Security group for all nodes in the cluster" 
  vpc_id      = "${aws_vpc.k8s.id}" 

  egress { 
    from_port   = 0 
    to_port     = 0 
    protocol    = "-1" 
    cidr_blocks = ["0.0.0.0/0"] 
  } 

... 
resource "aws_security_group_rule" "nodes-control_plane-proxy" { 
  description              = "API (proxy) communication to pods" 
  from_port                = 0 
  to_port                  = 65535 
  protocol                 = "tcp" 
  security_group_id        = "${aws_security_group.nodes.id}" 
  source_security_group_id = \ 
                          "${aws_security_group.control_plane.id}" 
  type                     = "ingress" 
} 
```

现在我们已经准备好运行我们的节点的基础设施，我们可以准备一个启动配置并将其分配给一个自动扩展组，以实际启动我们的节点，如下面的代码所示。

显然，我在这里选择的实例类型和磁盘大小可能不适合您的集群，因此在选择集群实例大小时，您需要参考第六章中的信息，即*生产规划*。所需的磁盘大小将在很大程度上取决于您的应用程序的平均镜像大小。`nodes.tf`的代码如下：

```
data "aws_ami" "eks-worker" { 
  filter { 
    name   = "name" 
    values = ["eks-worker-${var.k8s_version}*"] 
  } 

  most_recent = true 
  owners      = ["self"] 
} 

...                                                                    
  resource "aws_autoscaling_group" "node" { 
  launch_configuration = "${aws_launch_configuration.node.id}" 
  max_size             = 2 
  min_size             = 10 
  name                 = "eks-node-${var.cluster_name}" 
  vpc_zone_identifier  = ["${aws_subnet.private.*.id}"] 

  tag { 
    key                 = "Name" 
    value               = "eks-node-${var.cluster_name}" 
    propagate_at_launch = true 
  } 

  tag { 
    key              = "kubernetes.io/cluster/${var.cluster_name}" 
    value               = "owned" 
    propagate_at_launch = true 
  } 
} 

```

`kubernetes.io/cluster/<node name>`标签被`ekstrap`工具用来发现 EKS 端点以注册节点到集群，并被`kubelet`用来验证它是否已连接到正确的集群。

# 配置附加组件

Kubernetes 的许多功能来自于它易于通过添加额外的服务来扩展以提供额外的功能。

我们将通过部署`kube2iam`来看一个例子。这是一个守护程序，在我们的集群中的每个节点上运行，并拦截由我们的 pod 中运行的进程发出的对 AWS 元数据服务的调用。

通过使用 DaemonSet 在集群中的每个节点上运行一个 pod 来为这样的服务提供服务是一个简单的方法，如下面的代码所示。这种方法已经在我们的集群中用于将`aws-vpc-cni`网络插件部署到每个节点，并运行`kube-proxy`，这是运行在每个节点上的 Kubernetes 组件，负责将流向服务 IP 的流量路由到底层 pod：

```
kube2iam.yaml
--- 
apiVersion: v1 
kind: ServiceAccount 
metadata: 
  name: kube2iam 
  namespace: kube-system 
--- 
apiVersion: v1 
kind: List 
items:         
...                                                           kube2iam.tf 
resource "null_resource" "kube2iam" { 
  triggers = { 
    manifest_sha1 = "${sha1(file("${path.module}/kube2iam.yaml"))}" 
  } 

  provisioner "local-exec" { 
    command = " kubectl --kubeconfig=${local_file.kubeconfig.filename} apply -f 
${path.module}/kube2iam.yaml" 
  } 
} 
```

# 变革管理

使用像 Terraform 这样的工具来管理您的 Kubernetes 集群比我们在第三章中探索的手动方法具有许多优势，*云的选择*。当您想要测试对配置的更改，甚至当您要升级集群正在运行的 Kubernetes 版本时，能够快速轻松地重复配置集群的过程非常有用。

将基础设施定义为代码的另一个关键优势是，您可以使用版本控制工具随着时间的推移跟踪对基础设施所做的更改。其中一个关键优势是，每次进行更改时，您都可以留下提交消息。您现在做出的决定可能看起来很明显，但记录为什么以某种方式选择做某事将肯定有助于您和其他人在将来与您的配置一起工作，特别是因为那些其他人可能没有在您进行更改时拥有相同的上下文。

许多软件工程师已经写了很多关于写好提交消息的东西。最好的建议是确保您包含尽可能多的信息来解释为什么需要进行更改。如果您需要返回到配置几个月后，您未来的自己会感谢您。

考虑这个提交消息：

```
Update K8s Node Security Groups 

Open port 80 on the Node Security Group  
```

还要考虑这个提交消息：

```
Allow deveopers to access the guestbook app 

The guestbook is served from port 80\. We are allowing the control plane access to this port on the Node security groups, so developers can test the application using kubectl proxy. 

Once the application is in production and we provision a LoadBalancer, we can remove these rules. 
```

第一个提交消息很糟糕，因为它只是解释了你做了什么，而这应该很明显，只需看看配置如何改变就可以了。第二个消息提供了更多的信息。重要的是，第二个消息解释了为什么需要进行更改，并提供了一些对将来对集群进行更改的人有用的信息。没有这个重要的上下文，您可能会想知道为什么打开了端口`80`，并担心如果更改了该信息会发生什么。

在生产环境中操作 Kubernetes 集群不仅仅是关于如何在第一天启动集群；而是确保您可以随着时间的推移更新和扩展集群，以继续满足组织的要求。

# 总结

我们在本章中构建的集群仍然非常简单，实际上反映了我们可以在接下来的章节中建立的起点。然而，它确实满足了生产就绪的以下基本要求：

+   **可靠性**：通过使用 EKS，我们已经配置了一个可靠的控制平面，我们可以依赖它来管理我们的集群。

+   **可扩展性**：通过通过自动扩展组操作我们的节点，我们可以简单地在几秒钟内增加集群的额外容量。

+   **可维护性**：通过使用 Terraform 将我们的基础设施定义为代码，我们已经简化了将来管理我们的集群。通过为我们的节点机器使用的 AMI 设置构建过程，我们能够快速重建镜像以引入安全更新和更新版本的节点软件。
