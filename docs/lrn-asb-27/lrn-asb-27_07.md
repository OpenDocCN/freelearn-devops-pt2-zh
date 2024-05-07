# 往云端前进

在本章中，我们将看到如何使用 Ansible 在几分钟内配置基础架构。在我看来，这是 Ansible 最有趣和强大的功能之一，因为它允许你快速，一致地重建环境。当你有多个环境用于你的部署流程的不同阶段时，这非常重要。事实上，它允许你创建相等的环境，并在需要进行更改时保持其对齐，而不会感到任何痛苦。

让 Ansible 配置你的设备还有其他的优点，为此，我总是建议执行以下操作：

+   **审计追踪**：最近几年来，IT 行业吞食了大量其他行业，作为这个过程的一部分，审计流程现在将 IT 视为流程的关键部分。当审计师来到 IT 部门询问服务器的历史记录，从创建到目前为止，有 Ansible 播放脚本的整个过程将非常有帮助。

+   **多个分阶段的环境**：正如我们之前提到的，如果你有多个环境，使用 Ansible 配置服务器将会对你非常有帮助。

+   **迁移服务器**：当一家公司使用全球云提供商（如 AWS、DigitalOcean 或 Azure）时，他们通常会选择距离他们办公室或客户最近的区域来创建第一台服务器。这些提供商经常开设新区域，如果他们的新区域更靠近你，你可能会想将整个基础架构迁移到新区域。如果你手动配置了每个资源，这将是一场噩梦！

在本章中，从宏观层面上，我们将涵盖以下主题：

+   在 AWS 中配置机器

+   在 DigitalOcean 中配置机器

+   在 Azure 中配置机器

大多数新设备创建有两个阶段：

+   配置新机器或一组新机器

+   运行播放脚本，确保新机器被正确配置以发挥其在你的基础架构中的作用

在最初的章节中，我们已经看过了配置管理方面。在本章中，我们将更加专注于配置新机器，对配置管理的关注较少。

# 技术要求

你可以从本书的 GitHub 存储库中下载所有文件，网址为 [`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter05`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter05)。

# 在云中配置资源

有了这个，让我们跳到第一个主题。管理基础架构的团队今天有很多选择来运行他们的构建、测试和部署。提供商比如亚马逊、Azure 和 DigitalOcean 主要提供基础设施即服务（**IaaS**）。当我们谈论 IaaS 时，最好谈论资源而不是虚拟机，有几个原因：

+   这些公司允许您提供的多数产品都不是机器，而是其他关键资源，例如网络和存储。

+   最近，这些公司开始提供许多不同类型的计算实例，从裸金属机器到容器。

+   在某些非常简单的环境中，无网络（或存储）的机器设置可能就足够了，但在生产环境中可能不够。

这些公司通常提供 API、CLI、GUI 和 SDK 工具，以创建和管理云资源的整个生命周期。我们更感兴趣的是使用它们的 SDK，因为它在我们的自动化努力中将发挥重要作用。在刚开始时，建立新服务器并进行配置是有趣的，但在某个阶段，它可能变得乏味，因为它是相当重复的。每个配置步骤都会涉及几个类似的步骤，以使它们正常运行。

想象一下，某天早上您收到一封电子邮件，要求为三个新的客户设置安装，其中每个客户设置都有三到四个实例和一堆服务和依赖项。对您来说，这可能是个简单的任务，但它需要多次运行相同的重复命令，然后在服务器启动后监视它们以确认一切顺利。此外，您手动进行的任何操作都有可能引入问题。如果前两个客户设置正确启动，但由于疲劳，您遗漏了第三个客户的一个步骤，从而引入了问题怎么办？为了处理这种情况，就需要自动化。

云配置自动化使工程师能够尽快建立新的服务器，从而使他们能够集中精力处理其他优先事项。使用 Ansible，您可以轻松执行这些操作，并以最少的工作量自动化云配置。Ansible 为您提供了自动化各种不同云平台的能力，如 Amazon、Azure、DigitalOcean、Google Cloud、Rackspace 等，其中涵盖了 Ansible 核心版或扩展模块包中提供的不同服务的模块。

如前所述，启动新机器并不是结束游戏的标志。我们还需要确保我们配置它们以发挥所需的作用。

在接下来的章节中，我们将在以下环境中配置我们在之前章节中使用的环境（两个 Web 服务器和一个数据库服务器）：

+   **简单的 AWS 部署**：所有机器将放置在相同的**可用区**（AZs）和相同的网络中。

+   **复杂的 AWS 部署**：在此部署中，机器将分割到多个可用区（AZs）和网络中。

+   **DigitalOcean**：由于 DigitalOcean 不允许我们进行许多网络调整，因此它与第一个相似。

+   **Azure**：在这种情况下，我们将创建一个简单的部署。

# 在 AWS 中配置机器

AWS 是被广泛使用的最大的公有云，通常会被选择，因为它有大量可用的服务以及大量的文档、回答问题和相关文章可以在这样一个热门产品周围找到。

由于 AWS 的目标是成为完整的虚拟数据中心提供商（以及更多），我们将需要创建和管理我们的网络，就像我们如果要建立一个真实的数据中心一样。显然，我们不需要为电缆等东西，因为这是一个虚拟数据中心。因此，几行 Ansible playbook 就足够了。

# AWS 全球基础设施

亚马逊一直非常谨慎地分享其云实际上由哪些数据中心组成的位置或确切数量。在我写这篇文章时，AWS 拥有 21 个区域（还宣布了四个更多的区域），共 61 个 AZ 和数百个边缘位置。亚马逊将一个区域定义为“*我们（亚马逊）拥有多个 AZ 的世界中的物理位置*”。查看亚马逊关于 AZ 的文档，它说“*一个 AZ 由一个或多个离散的数据中心组成，每个数据中心都有冗余的电力、网络和连接，设立在不同的设施中*”。对于边缘位置，没有官方定义。

如你所见，从现实生活的角度来看，这些定义并没有帮助太多。当我尝试解释这些概念时，我通常使用我自己创造的不同定义：

+   **区域**：一组物理上靠近的 AZ

+   **AZ**：一个区域中的数据中心（亚马逊表示它可能不止一个数据中心，但由于没有列出每个 AZ 的具体几何形状的文件，我假定最坏情况）

+   **边缘位置**：互联网交换或第三方数据中心，亚马逊在这里拥有 CloudFront 和 Route 53 终端点。

尽管我试图使这些定义尽可能简单和有用，但其中一些仍然很模糊。当我们开始谈论现实世界的差异时，这些定义会立即变得清晰。例如，从网络速度的角度看，当你在同一个 AZ 内移动内容时，带宽非常高。当你在同一区域内使用两个 AZ 进行相同操作时，你会获得高带宽，而如果你在两个不同区域使用两个 AZ，带宽将更低。此外，还有价格差异，因为同一区域内的所有流量是免费的，而不同区域之间的流量是免费的。

# AWS 简单存储服务

亚马逊 **简单存储服务**（**S3**）是推出的第一项 AWS 服务，也是最为人所知的 AWS 服务之一。Amazon S3 是一种对象存储服务，具有公共端点和私有端点。它使用 bucket 的概念，允许您管理不同类型的文件并以简单的方式管理它们。Amazon S3 还提供了用户更高级的功能，例如使用内置 Web 服务器来提供 bucket 内容的能力。这就是许多人决定在 Amazon S3 上托管其网站或网站上的图片的原因之一。

S3 的优点主要有以下几点：

+   **价格方案**：您将按照已使用的每 GB/月和已传输的每 GB 计费。

+   **可靠性**：亚马逊声称 AWS S3 上的对象在任何一年内有 99.999999999% 的存活率。这比任何硬盘都要高出数量级。

+   **工具**：因为 S3 是一个已经存在多年的服务，许多工具已被实现以利用这项服务。

# AWS 弹性计算云

AWS 推出的第二项服务是 **弹性计算云**（**EC2**）服务。该服务允许您在 AWS 基础设施上创建计算机。您可以将这些 EC2 实例视为 OpenStack 计算实例或 VMware 虚拟机。最初，这些机器与 VPS 非常相似，但过了一段时间亚马逊决定赋予这些机器更多的灵活性，并引入了非常先进的网络选项。旧类型的机器仍然在最古老的数据中心中提供，名为 EC2 Classic，而新类型是当前的默认选项，只被称为 EC2。

# AWS 虚拟私有云

**虚拟私有云**（**VPC**）是亚马逊在前面提到的网络实现。VPC 更多的是一组工具而不是单个工具；实际上，它所提供的功能由经典数据中心中的多个金属盒子提供。

您可以通过 VPC 创建以下主要事项：

+   交换机

+   路由器

+   DHCP

+   网关

+   防火墙

+   **虚拟专用网络**（**VPN**）

当使用 VPC 时，重要的是要了解您的网络布局不是完全任意的，因为亚马逊创建了一些限制来简化其网络。基本限制如下：

+   您不能在 AZ 之间生成子网络。

+   您不能在不同的区域之间生成网络。

+   您不能直接路由不同区域的网络。

而对于前两者，唯一的解决方案是创建多个网络和子网络，而对于第三者，您实际上可以使用 VPN 服务来实现一个解决方法，该 VPN 服务可以是自我提供的，也可以是使用官方的 AWS VPN 服务提供的。

我们将主要使用 VPC 的交换和路由功能。

# AWS Route 53

与许多其他云服务一样，亚马逊提供了 **作为服务的 DNS**（**DNSaaS**） 功能，而在亚马逊的情况下，它被称为 **Route 53**。 Route 53 是一个分布式 DNS 服务，在全球各地拥有数百个端点（Route 53 存在于所有 AWS 边缘位置）。

Route 53 允许您为域创建不同的区域，从而允许分割地平线情况，根据请求 DNS 解析的客户端是否在您的 VPC 内或外部，将接收不同的响应。当您希望您的应用程序轻松地在您的 VPC 内外移动而无需更改时，这非常有用，但同时，您希望您的流量尽可能地保留在一个私有（虚拟）网络中。

# AWS 弹性块存储

AWS **弹性块存储**（**EBS**）是一个块存储提供者，允许您的 EC2 实例保留数据，这些数据将在重新启动后保留，并且非常灵活。从用户的角度来看，EBS 看起来很像任何其他 SAN 产品，只是具有更简单的界面，因为您只需要创建卷并告诉 EBS 需要连接到哪台机器，然后 EBS 会完成其余工作。您可以将多个卷附加到单个服务器，但每个卷一次只能连接到一个服务器。

# AWS 身份和访问管理

为了允许您管理用户和访问方法，亚马逊提供了 **身份和访问管理**（**IAM**） 服务。IAM 服务的主要特点如下：

+   创建、编辑和删除用户

+   更改用户密码

+   创建、编辑和删除组

+   管理用户和组关联

+   管理令牌

+   管理双因素身份验证

+   管理 SSH 密钥

我们将使用此服务来设置用户及其权限。

# 亚马逊关系型数据库服务

设置和维护关系数据库是复杂且非常耗时的。为了简化这一过程，亚马逊提供了一些广泛使用的 **作为服务的数据库**（**DBaaS**），具体如下：

+   Aurora

+   MariaDB

+   MySQL

+   Oracle

+   PostgreSQL

+   SQL Server

对于这些引擎中的每一个，亚马逊提供不同的功能和价格模型，但每个引擎的具体细节超出了本书的目标。

# 在 AWS 上设置账户

在开始使用 AWS 之前，我们需要的第一件事是账户。在 AWS 上创建账户非常简单，并且由亚马逊官方文档以及多个独立站点进行了很好的记录，因此在这些页面中不会涉及此操作。

创建好 AWS 账户后，需要进入 AWS 并完成以下操作：

+   在 EC2 | Keypairs 中上传您的 SSH 密钥。

+   在 Identity & Access Management | Users | 创建新用户 中创建新用户，并在 `~/.aws/credentials` 中创建一个文件，其中包含以下行：

```
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

创建好 AWS 密钥并上传 SSH 密钥后，您需要设置 Route 53。在 Route 53 中，您需要为您的域创建两个区域（如果您没有未使用的域，也可以使用子域）：一个公共区域和一个私有区域。

如果你只创建公共区域，Route 53 将在全局范围内传播该区域，但如果你创建了一个公共区域和一个私有区域，Route 53 将在创建私有区域时指定的 VPC 以外的所有地方提供你的公共区域服务。如果你在该 VPC 内查询这些 DNS 条目，将使用私有区域。这种方法有多个优点：

+   只公开公共机器的 IP 地址。

+   即使对于内部流量，也要始终使用 DNS 名称而不是 IP 地址。

+   确保你的内部机器直接通信，而无需通过公共网络传递数据。

+   由于 AWS 中的外部 IP 是由 Amazon 管理的虚拟 IP 地址，并使用 NAT 与你的实例关联，因此这种方法可以提供最少的跳数，从而减少时延。

如果你在公共区域中声明了一个条目，但在私有区域中没有声明该条目，那么 VPC 中的机器将无法解析该条目。

在创建公共区域后，AWS 将给出一些域名服务器 IP 地址，你需要将这些 IP 地址放入你的注册/根区域 DNS 中，以便你实际上可以解析这些 DNS。

# 简单的 AWS 部署

正如我们之前所说，我们首先需要的是网络连接。对于这个示例，我们只需要一个单独的可用区中的一个网络来容纳所有的机器。

在本节中，我们将在`playbooks/aws_simple_provision.yaml`文件中工作。

前两行只用于声明将执行命令的主机（localhost）和任务部分的开始：

```
- hosts: localhost
  tasks:  
```

首先，我们将确保存在公钥/私钥对：

```
    - name: Ensure key pair is present
      ec2_key:
        name: fale
        key_material: "{{ lookup('file', '~/.ssh/fale.pub') }}"
```

在 AWS 中，我们需要有一个 VPC 网络和子网。默认情况下，它们已经存在，但如果需要，可以执行以下步骤创建 VPC 网络：

```
    - name: Ensure VPC network is present
      ec2_vpc_net:
        name: NET_NAME
        state: present
        cidr_block: 10.0.0.0/16
        region: AWS_REGION
      register: aws_net
    - name: Ensure the VPC subnetwork is present
      ec2_vpc_subnet:
        state: present
        az: AWS_AZ
        vpc_id: '{{ aws_simple_net.vpc_id }}'
        cidr: 10.0.1.0/24
      register: aws_subnet
```

由于我们使用的是默认的 VPC，我们需要查询 AWS 以了解 VPC 网络和子网的值：

```
   - name: Ensure key pair is present
      ec2_key:
        name: fale
        key_material: "{{ lookup('file', '~/.ssh/fale.pub') }}"
    - name: Gather information of the EC2 VPC net in eu-west-1
 ec2_vpc_net_facts:
 region: eu-west-1
 register: aws_simple_net
 - name: Gather information of the EC2 VPC subnet in eu-west-1
 ec2_vpc_subnet_facts:
 region: eu-west-1
 filters:
 vpc-id: '{{ aws_simple_net.vpcs.0.id }}'
 register: aws_simple_subnet
```

现在我们已经获得了关于网络和子网的所有信息，接下来我们可以转向安全组。我们可以使用`ec2_group`模块来完成。在 AWS 世界中，安全组用于防火墙。安全组与共享相同目标的防火墙规则组非常相似（对于入口规则）或相同目标（对于出口规则）。与标准防火墙规则相比，实际上有三个不同之处值得一提：

+   多个安全组可以应用于同一个 EC2 实例。

+   作为源（对于入口规则）或目标（对于出口规则），你可以指定以下之一：

    +   一个实例 ID

    +   另一个安全组

    +   一个 IP 范围

+   你不需要在链的末尾指定默认拒绝规则，因为 AWS 会默认添加它。

所以，对于我的情况，以下代码将被添加到`playbooks/aws_simple_provision.yaml`中：

```
    - name: Ensure wssg Security Group is present
      ec2_group:
        name: wssg
        description: Web Security Group
        region: eu-west-1
        vpc_id: '{{ aws_simple_net.vpcs.0.id }}'
        rules:
          - proto: tcp 
            from_port: 22
            to_port: 22
            cidr_ip: 0.0.0.0/0
          - proto: tcp 
            from_port: 80
            to_port: 80
            cidr_ip: 0.0.0.0/0
          - proto: tcp 
            from_port: 443 
            to_port: 443 
            cidr_ip: 0.0.0.0/0
        rules_egress:
          - proto: all 
            cidr_ip: 0.0.0.0/0
      register: aws_simple_wssg
```

现在我们即将为我们的数据库创建另一个安全组。在这种情况下，我们只需要向 Web 安全组中的服务器打开`3036`端口即可：

```
    - name: Ensure dbsg Security Group is present
      ec2_group:
        name: dbsg
        description: DB Security Group
        region: eu-west-1
        vpc_id: '{{ aws_simple_net.vpcs.0.id }}'
        rules:
          - proto: tcp
            from_port: 3036
            to_port: 3036
            group_id: '{{ aws_simple_wssg.group_id }}'
        rules_egress:
          - proto: all
            cidr_ip: 0.0.0.0/0
```

如你所见，我们允许所有出站流量流动。这并不是安全最佳实践建议的做法，因此您可能需要调节出站流量。经常迫使您调节出站流量的情况是，如果您希望目标机器符合 PCI-DSS 标准。

现在我们有了 VPC、VPC 中的子网和所需的安全组，我们现在可以继续实际创建 EC2 实例了：

```
    - name: Setup instances
      ec2:
        assign_public_ip: '{{ item.assign_public_ip }}'
        image: ami-3548444c
        region: eu-west-1
        exact_count: 1
        key_name: fale
        count_tag:
          Name: '{{ item.name }}'
        instance_tags:
          Name: '{{ item.name }}'
        instance_type: t2.micro
        group_id: '{{ item.group_id }}'
        vpc_subnet_id: '{{ aws_simple_subnet.subnets.0.id }}'
        volumes:
          - device_name: /dev/sda1
            volume_type: gp2
            volume_size: 10
            delete_on_termination: True
      register: aws_simple_instances
      with_items:
        - name: ws01.simple.aws.fale.io
          group_id: '{{ aws_simple_wssg.group_id }}'
          assign_public_ip: True
        - name: ws02.simple.aws.fale.io
          group_id: '{{ aws_simple_wssg.group_id }}'
          assign_public_ip: True
        - name: db01.simple.aws.fale.io
          group_id: '{{ aws_simple_dbsg.group_id }}'
          assign_public_ip: False 
```

当我们创建 DB 机器时，我们没有指定 `assign_public_ip: True` 行。在这种情况下，该机器将不会收到公共 IP，因此它将无法从 VPC 外部访问。由于我们为此服务器使用了非常严格的安全组，因此它不会从 `wssg` 外的任何机器访问。

正如你所猜到的那样，我们刚刚看到的代码片段将创建我们的三个实例（两个 Web 服务器和一个 DB 服务器）。

现在，我们可以将这些新创建的实例添加到我们的 Route 53 帐户中，以便解析这些机器的完全限定域名。为了与 AWS Route 53 交互，我们将使用 `route53` 模块，该模块允许我们创建条目、查询条目和删除条目。要创建新条目，我们将使用以下代码：

```
    - name: Add route53 entry for server SERVER_NAME
      route53:
        command: create
        zone: ZONE_NAME
        record: RECORD_TO_ADD
        type: RECORD_TYPE
        ttl: TIME_TO_LIVE
        value: IP_VALUES
        wait: True
```

因此，为我们的服务器创建条目，我们将添加以下代码：

```
    - name: Add route53 rules for instances
      route53:
        command: create
        zone: aws.fale.io
        record: '{{ item.tagged_instances.0.tags.Name }}'
        type: A
        ttl: 1
        value: '{{ item.tagged_instances.0.public_ip }}'
        wait: True
      with_items: '{{ aws_simple_instances.results }}'
      when: item.tagged_instances.0.public_ip
    - name: Add internal route53 rules for instances
      route53:
        command: create
        zone: aws.fale.io
        private_zone: True
        record: '{{ item.tagged_instances.0.tags.Name }}'
        type: A
        ttl: 1
        value: '{{ item.tagged_instances.0.private_ip }}'
        wait: True
      with_items: '{{ aws_simple_instances.results }}'  
```

由于数据库服务器没有公共地址，将此机器发布到公共区域是没有意义的，因此我们只在内部区域中创建了此机器条目。

将所有内容整合在一起，`playbooks/aws_simple_provision.yaml` 将如下所示。完整的代码可在 GitHub 上找到：

```
---
- hosts: localhost
  tasks:
    - name: Ensure key pair is present
      ec2_key:
        name: fale
        key_material: "{{ lookup('file', '~/.ssh/fale.pub') }}"
    - name: Gather information of the EC2 VPC net in eu-west-1
      ec2_vpc_net_facts:
        region: eu-west-1
      register: aws_simple_net
    - name: Gather information of the EC2 VPC subnet in eu-west-1
      ec2_vpc_subnet_facts:
        region: eu-west-1
        filters:
          vpc-id: '{{ aws_simple_net.vpcs.0.id }}'
      register: aws_simple_subnet
   ...
```

运行 `ansible-playbook playbooks/aws_simple_provision.yaml`，Ansible 将负责创建我们的环境。

# 复杂的 AWS 部署

在本节中，我们将稍微修改之前的示例，将其中一个 Web 服务器移至同一地区的另一个可用区。为此，我们将在 `playbooks/aws_complex_provision.yaml` 中创建一个新文件，该文件与之前的文件非常相似，唯一的区别在于帮助我们配置机器的部分。事实上，我们将使用以下代码片段代替我们上次运行时使用的代码片段。完整的代码可在 GitHub 上找到：

```
    - name: Setup instances
      ec2:
        assign_public_ip: '{{ item.assign_public_ip }}'
        image: ami-3548444c
        region: eu-west-1
        exact_count: 1
        key_name: fale
        count_tag:
          Name: '{{ item.name }}'
        instance_tags:
          Name: '{{ item.name }}'
        instance_type: t2.micro
        group_id: '{{ item.group_id }}'
        vpc_subnet_id: '{{ item.vpc_subnet_id }}'
        volumes:
          - device_name: /dev/sda1
            volume_type: gp2
            volume_size: 10
            delete_on_termination: True
    ...
```

如你所见，我们将 `vpc_subnet_id` 放入一个变量中，这样我们就可以为 `ws02` 机器使用不同的子网。由于 AWS 已经默认提供了两个子网（每个子网都绑定到不同的可用区），因此只需使用以下的可用区即可。安全组和 Route 53 代码不需要更改，因为它们不在子网/可用区级别工作，而在 VPC 级别（对于安全组和内部 Route 53 区域）或全局级别（对于公共 Route 53）工作。

# 在 DigitalOcean 中进行机器配置

与 AWS 相比，DigitalOcean 似乎非常不完整。直到几个月前，DigitalOcean 只提供了小水滴、SSH 密钥管理和 DNS 管理。在撰写本文时，DigitalOcean 最近推出了额外的块存储服务。与许多竞争对手相比，DigitalOcean 的优势如下：

+   价格比 AWS 低。

+   非常简单的 API。

+   文档非常完善的 API。

+   小水滴与标准虚拟机非常相似（它们不进行奇怪的自定义）。

+   小水滴上下移动速度很快。

+   由于 DigitalOcean 具有非常简单的网络堆栈，因此比 AWS 更高效。

# 小水滴

小水滴是 DigitalOcean 提供的主要服务，是非常类似于 Amazon EC2 Classic 的计算实例。DigitalOcean 依赖于**Kernel Virtual Machine**（**KVM**）来虚拟化机器，确保非常高的性能和安全性。

由于他们没有以任何明智的方式更改 KVM，而且由于 KVM 是开源的，并且可在任何 Linux 机器上使用，这使得系统管理员可以在私有和公共云上创建相同的环境。DigitalOcean 小水滴将具有一个外部 IP，它们最终可以添加到一个虚拟网络中，该虚拟网络将允许您的机器使用内部 IP。

与许多其他可比较的服务不同，DigitalOcean 允许您的小水滴除了 IPv4 地址外，还具有 IPv6 地址。该服务是免费的。

# SSH 密钥管理

每次想要创建一个小水滴时，都必须指定是否要为 root 用户分配特定的 SSH 密钥，或者是否要设置一个密码（第一次登录时必须更改）。要能够选择一个 SSH 密钥，您需要一个用于上传的接口。DigitalOcean 允许您使用非常简单的界面进行此操作，该界面允许您列出当前的密钥，以及创建和删除密钥。

# 私有网络

正如在小水滴部分中所提到的，DigitalOcean 允许我们拥有一个私有网络，我们的机器可以与另一个机器通信。这允许对服务进行隔离（例如数据库服务）仅在内部网络上以提供更高级别的安全性。由于默认情况下，MySQL 绑定到所有可用接口，因此我们将需要稍微调整数据库角色以仅绑定到内部网络。

为了识别内部网络和外部网络，由于一些 DigitalOcean 的特殊性，有许多方法：

+   私有网络始终位于`10.0.0.0/8`网络中，而公共 IP 从不在该网络中。

+   公网始终是`eth0`，而私网始终是`eth1`。

根据您的可移植性需求，您可以使用这两种策略之一来了解在哪里绑定您的服务。

# 在 DigitalOcean 中添加 SSH 密钥

我们首先需要一个 DigitalOcean 帐户。一旦我们有了 DigitalOcean 用户、设置了信用卡和 API 密钥，我们就可以开始使用 Ansible 将我们的 SSH 密钥添加到我们的 DigitalOcean 云中。为此，我们需要创建一个名为`playbooks/do_provision.yaml`的文件，其结构如下：

```
- hosts: localhost
  tasks:
    - name: Add the SSH Key to Digital Ocean
      digital_ocean:
        state: present
        command: ssh
        name: SSH_KEY_NAME
        ssh_pub_key: 'ssh-rsa AAAA...'
        api_token: XXX
      register: ssh_key
```

在我这个案例中，这是我的文件内容：

```
    - name: Add the SSH Key to Digital Ocean
      digital_ocean:
        state: present
        command: ssh
        name: faleKey
        ssh_pub_key: "{{ lookup('file', '~/.ssh/fale.pub') }}"
        api_token: ee02b...2f11d
      register: ssh_key
```

然后我们可以执行它，你将会得到类似以下的结果：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [localhost]

TASK [Add the SSH Key to Digital Ocean] ******************************
changed: [localhost]

PLAY RECAP ***********************************************************
localhost : ok=2 changed=1 unreachable=0 failed=0
```

此任务是幂等的，因此我们可以多次执行它。如果密钥已经上传，那么每次运行都会返回 SSH 密钥 ID。

# 在 DigitalOcean 中部署

在撰写本文时，使用 Ansible 创建 droplet 的唯一方法是使用`digital_ocean`模块，该模块可能很快就会被弃用，因为它的许多功能现在已经由其他模块以更好、更干净的方式完成，而且 Ansible 错误跟踪器上已经有一个 bug 来跟踪它的完全重写和可能的弃用。我猜新模块将被称为`digital_ocean_droplet`，并且将具有类似的语法，但目前没有代码，所以这只是我的猜测。

要创建 droplets，我们将使用类似以下的`digital_ocean`模块语法：

```
 - name: Ensure the ws and db servers are present
   digital_ocean:
     state: present
     ssh_key_ids: KEY_ID
     name: '{{ item }}'
     api_token: DIGITAL_OCEAN_KEY
     size_id: 512mb
     region_id: lon1
     image_id: centos-7-0-x64
     unique_name: True
   with_items:
     - WEBSERVER 1
     - WEBSERVER 2
     - DBSERVER 1
```

为了确保我们所有的 provisioning 都是完全且健康的，我始终建议为整个基础架构创建一个单独的 provision 文件。因此，在我的情况下，我将在`playbooks/do_provision.yaml`文件中添加以下任务：

```
    - name: Ensure the ws and db servers are present
      digital_ocean:
        state: present
        ssh_key_ids: '{{ ssh_key.ssh_key.id }}'
        name: '{{ item }}'
        api_token: ee02b...2f11d
        size_id: 512mb
        region_id: lon1
        image_id: centos-7-x64
        unique_name: True
      with_items:
        - ws01.do.fale.io
        - ws02.do.fale.io
        - db01.do.fale.io
      register: droplets
```

这之后，我们可以使用`digital_ocean_domain`模块添加域名：

```
    - name: Ensure domain resolve properly
      digital_ocean_domain:
        api_token: ee02b...2f11d
        state: present
        name: '{{ item.droplet.name }}'
        ip: '{{ item.droplet.ip_address }}'
      with_items: '{{ droplets.results }}'
```

因此，将所有这些放在一起，我们的`playbooks/do_provision.yaml`将如下所示，完整的代码块可在 GitHub 上找到：

```
---
- hosts: localhost
  tasks:
    - name: Ensure domain is present
      digital_ocean_domain:
        api_token: ee02b...2f11d
        state: present
        name: do.fale.io
        ip: 127.0.0.1
    - name: Add the SSH Key to Digital Ocean
      digital_ocean:
        state: present
        command: ssh
        name: faleKey
        ssh_pub_key: "{{ lookup('file', '~/.ssh/fale.pub') }}"
        api_token: ee02b...2f11d
      register: ssh_key
   ...
```

因此，我们现在可以用以下命令运行它：

```
ansible-playbook playbooks/do_provision.yaml 
```

我们将看到类似以下的结果。完整的代码输出文件可在 GitHub 上找到：

```
PLAY [localhost] *****************************************************

TASK [Gathering Facts] ***********************************************
ok: [localhost]

TASK [Ensure domain is present] **************************************
changed: [localhost]

TASK [Add the SSH Key to Digital Ocean] ******************************
changed: [localhost]

TASK [Ensure the ws and db servers are present] **********************
changed: [localhost] => (item=ws01.do.fale.io)
changed: [localhost] => (item=ws02.do.fale.io)
changed: [localhost] => (item=db01.do.fale.io)

...
```

我们已经看到了如何使用几行 Ansible 在 DigitalOcean 上提供三台机器。我们现在可以使用我们在前几章中讨论过的 playbook 来配置它们。

# 在 Azure 中提供机器

最近，Azure 正在成为一些公司中最大的云之一。

正如你可能想象的那样，Ansible 有 Azure 特定的模块，可以轻松创建 Azure 环境。

在创建了帐户之后，我们在 Azure 上首先需要做的事情是设置授权。

有几种方法可以做到这一点，但最简单的方法可能是创建以 INI 格式包含`[default]`部分的`~/.azure/credentials`文件，其中包含`subscription_id`和，可选的，`client_id`和`secret`或`ad_user`和`password`。

一个示例如下文件：

```
[default]
subscription_id: __AZURE_SUBSCRIPTION_ID__
client_id: __AZURE_CLIENT_ID__ secret: __AZURE_SECRET__
```

之后，我们需要一个资源组，然后我们将在其中创建所有资源。

为此，我们可以使用`azure_rm_resourcegroup`，语法如下：

```
    - name: Create resource group
      azure_rm_resourcegroup:
        name: myResourceGroup
        location: eastus
```

现在我们有了资源组，我们可以在其中创建虚拟网络和虚拟子网络：

```
     - name: Create Azure VM
            hosts: localhost
            tasks:
      - name: Create resource group
            azure_rm_resourcegroup:
            name: myResourceGroup
            location: eastus
      - name: Create virtual network
            azure_rm_virtualnetwork:
            resource_group: myResourceGroup
            name: myVnet
           address_prefixes: "10.0.0.0/16"
    - name: Add subnet
           azure_rm_subnet:
           resource_group: myResourceGroup
          name: mySubnet
          address_prefix: "10.0.1.0/24"
          virtual_network: myVnet
```

在我们继续创建虚拟机之前，我们仍然需要一些网络项目，更具体地说，需要一个公共 IP、一个网络安全组和一个虚拟网络卡：

```
    - name: Create public IP address
      azure_rm_publicipaddress:
        resource_group: myResourceGroup
        allocation_method: Static
        name: myPublicIP
      register: output_ip_address
    - name: Dump public IP for VM which will be created
      debug:
        msg: "The public IP is {{ output_ip_address.state.ip_address }}."
    - name: Create Network Security Group that allows SSH 
      azure_rm_securitygroup:
        resource_group: myResourceGroup
        name: myNetworkSecurityGroup
        rules:
          - name: SSH 
            protocol: Tcp 
            destination_port_range: 22
            access: Allow
            priority: 1001
            direction: Inbound
    - name: Create virtual network inteface card
      azure_rm_networkinterface:
        resource_group: myResourceGroup
        name: myNIC
        virtual_network: myVnet
        subnet: mySubnet
        public_ip_name: myPublicIP
        security_group: myNetworkSecurityGroup
```

现在我们准备创建我们的第一台 Azure 机器，使用以下代码：

```
    - name: Create VM
      azure_rm_virtualmachine:
        resource_group: myResourceGroup
        name: myVM
        vm_size: Standard_DS1_v2
        admin_username: azureuser
        ssh_password_enabled: false
        ssh_public_keys:
          - path: /home/azureuser/.ssh/authorized_keys
            key_data: "{{ lookup('file', '~/.ssh/fale.pub') }}"
        network_interfaces: myNIC
        image:
          offer: CentOS
          publisher: OpenLogic
          sku: '7.5'
        version: latest
```

运行 playbook 后，您将在 Azure 上获得一个运行 CentOS 的机器！

# 摘要

在本章中，我们看到了如何在 AWS 云、DigitalOcean 和 Azure 中配置我们的机器。在 AWS 云的情况下，我们看到了两个不同的示例，一个非常简单，一个稍微复杂一些。

在下一章中，我们将讨论当 Ansible 发现问题时如何通知我们。
