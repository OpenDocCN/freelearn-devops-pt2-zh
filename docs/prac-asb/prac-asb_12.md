# 第九章：使用 Ansible 进行网络自动化

多年前，标准做法是手动配置每个网络设备。这主要是因为路由器和交换机主要是路由物理服务器的流量，因此每个网络设备上不需要太多的配置，而且变化缓慢。此外，只有人类才有足够的信息来设置网络。无论是规划还是执行，一切都非常手动。

虚拟化改变了这种范式，因为它导致了数千台机器连接到同一交换机或路由器，每台机器可能具有不同的网络需求。变化快速且频繁预期，并且随着虚拟基础架构的代码定义，人类管理员只需跟上基础设施的变化就成了全职工作。虚拟化编排平台对机器位置有更好的了解，甚至可以为我们生成清单，正如我们在前几章中看到的。实际上，没有办法让一个人类记住或管理现代大规模虚拟化基础设施。因此，很明显，自动化在配置网络基础设施时是必需的。

我们将在本章中学习更多关于这一点，以及我们可以做些什么来自动化我们的网络，内容包括以下主题：

+   为什么要自动化网络管理？

+   Ansible 如何管理网络设备

+   如何启用网络自动化

+   可用的 Ansible 网络模块

+   连接到网络设备

+   网络设备的环境变量

+   网络设备的自定义条件语句

让我们开始吧！

# 技术要求

本章假定您已经按照第一章 *开始使用 Ansible*中详细介绍的方式设置了控制主机，并且正在使用最新版本——本章的示例是使用 Ansible 2.9 进行测试的。本章还假定您至少有一个额外的主机进行测试，并且最好是基于 Linux 的。由于本章以网络设备为中心，我们理解并不是每个人都能够访问特定的网络设备进行测试（例如 Cisco 交换机）。在给出示例并且您可以访问这些设备的情况下，请随时探索这些示例。但是，如果您无法访问任何网络硬件，我们将使用免费提供的 Cumulus VX 进行示例演示，该软件提供了 Cumulus Networks 的交换环境的完整演示。尽管本章将给出特定主机名的示例，但您可以自由地用自己的主机名和/或 IP 地址替换它们。如何进行替换将在适当的位置提供详细信息。

本章的代码包在此处可用：[`github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%209`](https://github.com/PacktPublishing/Practical-Ansible-2/tree/master/Chapter%209)。

# 为什么要自动化网络管理？

在过去的 30 年里，我们设计数据中心的方式发生了根本性的变化。在 90 年代，典型的数据中心充满了具有非常特定目的的物理机器。在许多公司，服务器是根据机器的用途由不同的供应商购买的。这意味着需要机器、网络设备和存储设备，并且这些设备被购买、配置和交付。

这里的一个重大缺点是在确定需要机器和交付之间存在显著的滞后。在那段时间里，这是可以接受的，因为大多数公司的系统非常少，它们很少改变。此外，这种方法非常昂贵，因为许多设备被低效利用。

随着社会和科技公司的进步，我们知道今天，对于公司来说削减基础设施部署时间和成本变得非常重要。这为一个新的想法打开了道路：虚拟化。通过创建一个虚拟化集群，您不需要拥有正确尺寸的物理主机，因此您可以预先配置一些主机，将它们添加到资源池中，然后在虚拟化平台中创建合适尺寸的机器。这意味着当需要新的机器时，您可以通过几次点击创建它，并且几秒钟内就可以准备好。

这种转变还使企业能够从每个项目都部署具有独特数据中心要求的基础设施转变为一个可以由软件和配置定义行为的大型中央基础设施。这意味着一个单一的网络基础设施可以支持所有项目，而不管它们的规模如何。我们称之为虚拟数据中心基础设施，在这种基础设施中，我们尽可能地利用通用设计模式。这使得企业能够以大规模部署、切换和提供基础设施，以便通过简单地对其进行细分（例如创建虚拟服务器）来成功实施多种项目。

虚拟化带来的另一个重大优势是工作负载和物理主机的解耦。从历史上看，由于工作负载与物理主机绑定，如果主机死机，工作负载本身也会死机，除非在不同硬件上进行适当复制。虚拟化解决了这个问题，因为工作负载现在与一个或多个虚拟主机绑定，但这些虚拟主机可以自由地从一个物理主机移动到另一个物理主机。

快速配置机器的能力以及这些机器能够从一个主机移动到另一个主机的能力，导致了网络配置管理的问题。以前，人为地在安装新机器时调整配置细节是可以接受的，但现在，机器在主机之间移动（因此从一个物理交换机端口移动到另一个端口）而不需要任何人为干预。这意味着系统也需要更新网络配置。

与此同时，VLAN 在网络中占据了一席之地，这使得网络设备的利用得到了显著改善，从而优化了它们的成本。

今天，我们在更大的规模上工作，虚拟对象（机器、容器、函数等）在我们的数据中心中移动，完全由软件系统管理，人类参与的越来越少。

在这种环境中，自动化网络是他们成功的关键部分。

今天，有一些公司（著名的“云服务提供商”）在一个规模上工作，手动网络管理不仅不切实际，而且即使雇佣了大量网络工程师团队，也是不可能的。另一方面，有许多环境在技术上可能（至少部分地）手动管理网络配置，但仍然不切实际。

除了配置网络设备所需的时间之外，网络自动化最大的优势（从我的角度来看）是大大减少人为错误的机会。如果一个人必须在 100 台设备上配置 VLAN，很可能在这个过程中会出现一些错误。这是绝对正常的，但仍然是有问题的，因为这些配置将需要进行全面测试和修改。通常情况下，问题并不会止步于此，因为当设备损坏并因此需要更换时，人们必须以与旧设备相同的方式配置新设备。通常情况下，随着时间的推移，配置会发生变化，而且很多时候没有明确的方法来追踪这一点，因此在更换有故障的网络设备时，可能会出现一些在旧设备中存在但在新设备中不存在的规则问题。

既然我们已经讨论了自动化网络管理的必要性，让我们看看如何使用 Ansible 管理网络设备。

# 学习 Ansible 如何管理网络设备

Ansible 允许您管理许多不同的网络设备，包括 Arista EOS、Cisco ASA、Cisco IOS、Cisco IOS XR、Cisco NX-OS、Dell OS 6、Dell OS 9、Dell OS 10、Extreme EXOS、Extreme IronWare、Extreme NOS、Extreme SLX-OS、Extreme VOSS、F5 BIG-IP、F5 BIG-IQ、Junos OS、Lenovo CNOS、Lenovo ENOS、MikroTik RouterOS、Nokia SR OS、Pluribus Netvisor、VyOS 和支持 NETCONF 的 OS。正如您可以想象的那样，我们可以通过各种方式让 Ansible 与它们进行通信。

此外，我们必须记住，Ansible 网络模块在控制器主机上运行（即您发出`ansible`命令的主机），而通常，Ansible 模块在目标主机上运行。这种差异很重要，因为它允许 Ansible 根据目标设备类型使用不同的连接机制。请记住，即使您的主机具有 SSH 管理功能（许多交换机都有），由于 Ansible 在目标主机上运行其模块，因此需要目标主机安装 Python。大多数交换机（和嵌入式硬件）缺乏 Python 环境，因此我们必须使用其他连接协议。Ansible 支持的用于网络设备管理的关键协议在此处给出。

Ansible 使用以下五种主要连接类型连接这些网络设备：

+   `network_cli`

+   `netconf`

+   `httpapi`

+   `local`

+   `ssh`

当您与网络设备建立连接时，您需要根据设备支持的连接机制和您的需求选择连接机制：

+   `network_cli` 得到大多数模块的支持，它与 Ansible 通常使用非网络模块的方式最相似。这种模式通过 SSH 使用 CLI。该协议在配置开始时创建持久连接，并在整个任务的持续时间内保持连接，因此您不必为每个后续任务提供凭据。

+   `netconf` 受到很少模块的支持（在撰写本文时，这些模块只是支持 NETCONF 和 Junos OS 的操作系统）。这种模式通过 SSH 使用 XML，因此基本上它将基于 XML 的配置应用到设备上。该协议在配置开始时创建持久连接，并在整个任务的持续时间内保持连接，因此您不必为每个后续任务提供凭据。

+   `httpapi` 受到少量模块的支持（在撰写本文时，这些模块是 Arista EOS、Cisco NX-OS 和 Extreme EXOS）。这种模式使用设备发布的 HTTP API。该协议在配置开始时创建持久连接，并在整个任务的持续时间内保持连接，因此您不必为每个后续任务提供凭据。

+   `Local`被大多数设备支持，但是它是一个已弃用的模式。这基本上是一个依赖于供应商的连接模式，可能需要使用一些供应商包。这种模式不会创建持久连接，因此在每个任务开始时，你都需要传递凭据。在可能的情况下，避免使用这种模式。

+   在本节中，`ssh` 不能被忘记。虽然许多设备依赖于此处列出的连接模式，但正在创建一种新型设备，它们在白盒交换机硬件上原生运行 Linux。Cumulus Networks 就是一个这样的例子，由于软件是基于 Linux 的，所有配置都可以通过 SSH 进行，就好像交换机实际上只是另一台 Linux 服务器一样。

了解 Ansible 如何连接和与你的网络硬件通信是很重要的，因为它能让你理解你构建 Ansible playbooks 和调试问题时所需的知识。在本节中，我们介绍了在处理网络硬件时会遇到的通信协议。在下一节中，我们将在此基础上继续，看看如何使用 Ansible 开始我们的网络自动化之旅的基础知识。

# 启用网络自动化

在你使用 Ansible 进行网络自动化之前，你需要确保你拥有一切所需的东西。

根据我们将要使用的连接方法的不同，我们需要不同的依赖。举例来说，我们将使用具有`network_cli`连接性的 Cisco IOS 设备。

Ansible 网络自动化的唯一要求如下：

+   Ansible 2.5+

+   与网络设备的正确连接

首先，我们需要检查 Ansible 的版本：

1.  确保你有一个最新的 Ansible 版本，你可以运行以下命令：

```
$ ansible --version
```

这将告诉你你的 Ansible 安装的版本。

1.  如果是 2.5 或更高版本，你可以发出以下命令（带有适当的选项）来检查网络设备的连接：

```
$ ansible all -i n1.example.com, -c network_cli -u my_user -k -m ios_facts -e ansible_network_os=ios all
```

这应该返回你设备的事实，证明我们能够连接。对于任何其他目标，Ansible 都能够检索事实，这通常是 Ansible 与目标交互时的第一步。

这是一个关键步骤，因为这使得 Ansible 能够了解设备的当前状态，从而采取适当的行动。

通过在目标设备上运行`ios_facts`模块，我们只是执行了这个第一个标准步骤（因此不会对设备本身或其配置进行任何更改），但这将确认 Ansible 能够连接到设备并对其执行命令。

显然，只有当你有权限访问运行 Cisco IOS 的网络设备时，你才能运行前面的命令并探索其行为。我们知道并非每个人都有相同的网络设备可用于测试（或者根本没有！）。幸运的是，一种新型交换机正在出现——“白盒”交换机。这些交换机由各种制造商制造，基于标准化硬件，你可以在上面安装自己的网络操作系统。Cumulus Linux 就是这样一种操作系统，它的一个免费测试版本叫做 Cumulus VX，可以供你下载。

在撰写本文时，Cumulus VX 的下载链接是[`cumulusnetworks.com/products/cumulus-vx/`](https://cumulusnetworks.com/products/cumulus-vx/)。你需要注册才能下载，但这样做可以让你免费访问开放网络的世界。

只需下载适合你的 hypervisor（例如 VirtualBox）的镜像，然后像运行任何其他 Linux 虚拟机一样运行它。完成后，你可以连接到 Cumulus VX 交换机，就像连接到任何其他 SSH 设备一样。例如，要运行一个关于所有交换机端口接口的事实的临时命令（在 Cumulus VX 上被枚举为`swp1`、`swp2`和`swpX`），你可以运行以下命令：

```
$ ansible -i vx01.example.com, -u cumulus -m setup -a 'filter=ansible_swp*' all --ask-pass
```

如果成功，这应该会导致关于 Cumulus VX 虚拟交换机的交换机端口接口的大量信息。在我的测试系统上，此输出的第一部分如下所示：

```
vx01.example.com | SUCCESS => {
 "ansible_facts": {
 "ansible_swp1": {
 "active": false,
 "device": "swp1",
 "features": {
 "esp_hw_offload": "off [fixed]",
 "esp_tx_csum_hw_offload": "off [fixed]",
 "fcoe_mtu": "off [fixed]",
 "generic_receive_offload": "on",
 "generic_segmentation_offload": "on",
 "highdma": "off [fixed]",
...
```

正如您所看到的，使用诸如 Cumulus Linux 之类的操作系统来使用白盒交换机的优势在于，您可以使用标准的 SSH 协议进行连接，甚至可以使用内置的`setup`模块来收集有关它的信息。使用其他专有硬件并不更加困难，但只是需要指定更多的参数，就像我们在本章前面展示的那样。

现在您已经了解了启用网络自动化的基本知识，让我们学习如何在 Ansible 中发现适合我们所需自动化任务的适当网络模块。

# 审查可用的 Ansible 网络模块

目前，有超过 20 种不同的网络平台上的数千个模块。让我们学习如何找到对您更相关的模块：

1.  首先，您需要知道您有哪种设备类型以及 Ansible 如何调用它。在[`docs.ansible.com/ansible/latest/network/user_guide/platform_index.html`](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html)页面上，您可以找到 Ansible 支持的不同设备类型以及它们的指定方式。在我们的示例中，我们将以 Cisco IOS 为例。

1.  在[`docs.ansible.com/ansible/latest/modules/list_of_network_modules.html`](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html)页面上，您可以搜索专门针对您所需交换机家族的类别，并且您将能够看到所有可以使用的模块。

模块列表对于我们来说太大且特定于家族，无法对其进行深入讨论。每个版本都会有数百个新的模块添加，因此此列表每次发布都会变得更大。

如果您熟悉如何以手动方式配置设备，您将很快发现模块的名称相当自然，因此您将很容易理解它们的功能。但是，让我们从 Cisco IOS 模块集合中挑选出一些例子，具体参考[`docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#ios`](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#ios)：

+   `ios_banner`：顾名思义，此模块将允许您微调和修改登录横幅（在许多系统中称为`motd`）。

+   `ios_bgp`：此模块允许您配置 BGP 路由。

+   `ios_command`：这是 Ansible `command`模块的 IOS 等效模块，它允许您执行许多不同的命令。就像`command`模块一样，这是一个非常强大的模块，但最好使用特定的模块来执行我们要执行的操作，如果它们可用的话。

+   `ios_config`：此模块允许我们对设备的配置文件进行几乎任何更改。就像`ios_command`模块一样，这是一个非常强大的模块，但最好使用特定的模块来执行我们要执行的操作，如果它们可用的话。如果使用了缩写命令，则此模块的幂等性将无法保证。

+   `ios_vlan`：此模块允许配置 VLAN。

这只是一些例子，但对于 Cisco IOS 还有许多其他模块（在撰写本文时有 27 个），如果您找不到执行所需操作的特定模块，您总是可以退而使用`ios_command`和`ios_config`，由于它们的灵活性，将允许您执行任何您能想到的操作。

相比之下，如果你正在使用 Cumulus Linux 交换机，你会发现只有一个模块 - `nclu`（参见[`docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#cumulus`](https://docs.ansible.com/ansible/latest/modules/list_of_network_modules.html#cumulus)）。这反映了在 Cumulus Linux 中所有配置工作都是用这个命令处理的事实。如果你需要自定义每日消息或 Linux 操作系统的其他方面，你可以以正常方式进行（例如，使用我们在本书中之前演示过的`template`或`copy`模块）。

与往常一样，Ansible 文档是你的朋友，当你学习如何在新类设备上自动化命令时，它应该是你的首要选择。在本节中，我们演示了一个简单的过程，用于查找适用于你的网络设备类别的 Ansible 模块，以 Cisco 作为具体示例（尽管你可以将这些原则应用于任何其他设备）。现在，让我们看看 Ansible 如何连接到网络设备。

# 连接到网络设备

正如我们所看到的，Ansible 网络中有一些特殊之处，因此需要特定的配置。

为了使用 Ansible 管理网络设备，你至少需要一个设备进行测试。假设我们有一个 Cisco IOS 系统可供使用。可以肯定的是，不是每个人都有这样的设备进行测试，因此以下内容仅作为假设示例提供。

根据[`docs.ansible.com/ansible/latest/network/user_guide/platform_index.html`](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html)页面，我们可以看到这个设备的正确`ansible_network_os`是`ios`，我们可以使用`network_cli`和`local`连接到它。由于`local`已经被弃用，我们将使用`network_cli`。按照以下步骤配置 Ansible，以便你可以管理 IOS 设备：

1.  首先，让我们创建包含我们设备的清单文件在`routers`组中：

```
[routers]
n1.example.com
n2.example.com

[cumulusvx]
vx01.example.com
```

1.  要知道要使用哪些连接参数，我们将设置 Ansible 的特殊连接变量，以便它们定义连接参数。我们将在 playbook 的组变量子目录中进行这些操作，因此我们需要创建包含以下内容的`group_vars/routers.yml`文件：

```
---
ansible_connection: network_cli
ansible_network_os: ios
ansible_become: True
ansible_become_method: enable
```

凭借这些特殊的 Ansible 变量，它将知道如何连接到你的设备。我们在本书的前面已经涵盖了一些这些示例，但作为一个回顾，Ansible 使用这些变量的值来确定其行为的方式如下：

+   `ansible_connection`：这个变量被 Ansible 用来决定如何连接到设备。通过选择`network_cli`，我们指示 Ansible 以 SSH 模式连接到 CLI，就像我们在前一段讨论的那样。

+   `ansible_network_os`：这个变量被 Ansible 用来理解我们将要使用的设备的设备系列。通过选择`ios`，我们指示 Ansible 期望一个 Cisco IOS 设备。

+   `ansible_become`：这个变量被 Ansible 用来决定是否在设备上执行特权升级。通过指定`True`，我们告诉 Ansible 执行特权升级。

+   `ansible_become_method`：在各种设备上执行特权升级有许多不同的方法（通常在 Linux 服务器上是`sudo` - 这是默认设置），对于 Cisco IOS，我们必须将其设置为`enable`。

通过这些步骤，你已经学会了连接到网络设备的必要步骤。

为了验证连接是否按预期工作（假设你可以访问运行 Cisco IOS 的路由器），你可以运行这个简单的 playbook，名为`ios_facts.yaml`：

```
---
- name: Play to return facts from a Cisco IOS device
  hosts: routers
  gather_facts: False
  tasks:
    - name: Gather IOS facts
      ios_facts:
        gather_subset: all
```

你可以使用以下命令来运行这个过程：

```
$ ansible-playbook -i hosts ios_facts.yml --ask-pass
```

如果它成功返回，这意味着你的配置是正确的，并且你已经能够给予 Ansible 管理你的 IOS 设备所需的授权。

同样，如果您想连接到 Cumulus VX 设备，您可以添加另一个名为`group_vars/cumulusvx.yml`的组变量文件，其中包含以下代码：

```
---
ansible_user: cumulus
become: false
```

一个类似的 playbook，返回有关我们的 Cumulus VX 交换机的所有信息，可能如下所示：

```
---
- name: Simply play to gather Cumulus VX switch facts
  hosts: cumulusvx
  gather_facts: no

  tasks:
    - name: Gather facts
      setup:
        gather_subset: all
```

您可以通过使用以下命令以正常方式运行：

```
$ ansible-playbook -i hosts cumulusvx_facts.yml --ask-pass
```

如果成功，您应该会从您的 playbook 运行中看到以下输出：

```
SSH password:

PLAY [Simply play to gather Cumulus VX switch facts] ************************************************************************************************

TASK [Gather facts] ************************************************************************************************
ok: [vx01.example.com]

PLAY RECAP ************************************************************************************************
vx01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

这演示了连接到 Ansible 中两种不同类型的网络设备的技术，其中一种您可以自行测试，而无需访问任何特殊硬件。现在，让我们通过查看如何为 Ansible 中的网络设备设置环境变量来进一步学习。

# 网络设备的环境变量

网络的复杂性很高，网络系统也非常多样化。因此，Ansible 拥有大量的变量，可以帮助您调整它，使 Ansible 适应您的环境。

假设您有两个不同的网络（即一个用于计算，一个用于网络设备），它们不能直接通信，但必须通过堡垒主机才能从一个网络到达另一个网络。由于我们在计算网络中使用了 Ansible，我们需要通过堡垒主机跳转网络，以配置管理网络中的 IOS 路由器。此外，我们的目标交换机需要代理才能访问互联网。

要连接到数据库网络中的 IOS 路由器，我们需要为我们的网络设备创建一个新的组，这些设备位于一个单独的网络上。例如，对于这个例子，可能会指定如下：

```
[bastion_routers]
n1.example.com
n2.example.com

[bastion_cumulusvx]
vx01.example.com
```

创建了更新后的清单后，我们可以创建一个新的组变量文件，例如`group_vars/bastion_routers.yaml`，其中包含以下内容：

```
---
ansible_connection: network_cli
ansible_network_os: ios
ansible_become: True
ansible_become_method: enable
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q bastion.example.com"'
proxy_env:
    http_proxy: http://proxy.example.com:8080
```

如果我们的 Cumulus VX 交换机位于堡垒服务器后面，我们也可以创建一个`group_vars/bastion_cumulusvx.yml`文件来实现相同的效果：

```
---
ansible_user: cumulus
ansible_become: false
ansible_ssh_common_args: '-o ProxyCommand="ssh -W %h:%p -q bastion.example.com"'
proxy_env:
    http_proxy: http://proxy.example.com:8080
```

除了我们在上一节中讨论的选项之外，我们现在还有两个额外的选项：

+   `ansible_ssh_common_args`：这是一个非常强大的选项，允许我们向 SSH 连接添加额外的选项，以便我们可以调整它们的行为。这些选项应该相当容易识别，因为您已经在您的 SSH 配置中使用它们，只需简单地 SSH 到目标机器。在这种特定情况下，我们正在添加一个`ProxyCommand`，这是执行跳转到主机（通常是堡垒主机）的 SSH 指令，以便我们可以安全地进入目标主机。

+   `http_proxy`：这个选项位于`proxy_env`选项下面，在网络隔离很强的环境中非常关键，因此您的机器除非使用代理，否则无法与互联网进行交互。

假设您已经设置了无密码（例如基于 SSH 密钥的）访问到您的堡垒主机，您应该能够对您的 Cumulus VX 主机运行一个临时的 Ansible `ping`命令，如下所示：

```
$ ansible -i hosts -m ping -u cumulus --ask-pass bastion_cumulusvx
SSH password:

vx01.example.com | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
```

请注意，堡垒服务器的使用变得透明 - 您可以继续使用 Ansible 进行自动化，就好像您在同一个平面网络上一样。如果您可以访问基于 Cisco IOS 的设备，您也应该能够对`bastion_routers`组运行类似的命令，并取得类似的积极结果。现在您已经学会了为网络设备设置环境变量的必要步骤，以及如何使用 Ansible 访问它们，即使它们在隔离的网络中，让我们学习如何为网络设备设置条件语句。

# 网络设备的条件语句

尽管没有特定于网络的 Ansible 条件语句，但在与网络相关的 Ansible 使用中，条件语句是相当常见的。

在网络中，启用和禁用端口是很常见的。要使数据通过电缆，电缆两端的端口都应该启用，并且结果为“连接”状态（一些供应商可能会使用不同的名称，但概念是相同的）。

假设我们有两个 Arista Networks EOS 设备，并且我们在端口上发出了 ON 状态，并且需要等待连接建立后再继续。

要等待`Ethernet4`接口启用，我们需要在我们的 playbook 中添加以下任务：

```
- name: Wait for interface to be enabled
  eos_command:
      commands:
          - show interface Ethernet4 | json
      wait_for:
          - "result[0].interfaces.Ethernet4.interfaceStatus  eq  connected"
```

`eos_command`是允许我们向 Arista Networks EOS 设备发出自由形式命令的模块。命令本身需要在`commands`选项中指定为数组。通过`wait_for`选项，我们可以指定一个条件，Ansible 将在指定任务上重复，直到条件满足。由于命令的输出被重定向到`json`实用程序，输出将是一个 JSON，因此我们可以使用 Ansible 操作 JSON 数据的能力来遍历其结构。

我们可以在 Cumulus VX 上实现类似的结果——例如，我们可以查询从交换机收集的事实，看看端口`swp2`是否已启用。如果没有启用，那么我们将启用它；但是，如果已启用，我们将跳过该命令。我们可以通过一个简单的 playbook 来实现这一点，如下所示：

```
---
- name: Simple play to demonstrate conditional on Cumulus Linux
  hosts: cumulusvx

  tasks:
    - name: Enable swp2 if it is disabled
      nclu:
        commands:
          - add int swp2
        commit: yes
      when: ansible_swp2.active == false
```

注意我们任务中`when`子句的使用，意味着我们只应在`swp2`不活动时发出配置指令。如果我们在未配置的 Cumulus Linux 交换机上第一次运行此 playbook，我们应该看到类似以下的输出：

```
PLAY [Simple play to demonstrate conditional on Cumulus Linux] ***************************************************************

TASK [Gathering Facts] 
***************************************************************
ok: [vx01.example.com]

TASK [Enable swp2 if it is disabled] ***************************************************************
changed: [vx01.example.com]

PLAY RECAP 
***************************************************************
vx01.example.com : ok=2 changed=1 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

正如我们所看到的，`nclu`模块将我们的更改提交到了交换机配置中。然而，如果我们再次运行 playbook，输出应该更像这样：

```
PLAY [Simple play to demonstrate conditional on Cumulus Linux] ***************************************************************

TASK [Gathering Facts] 
***************************************************************
ok: [vx01.example.com]

TASK [Enable swp2 if it is disabled] ***************************************************************
skipping: [vx01.example.com]

PLAY RECAP
***************************************************************
vx01.example.com : ok=1 changed=0 unreachable=0 failed=0 skipped=1 rescued=0 ignored=0
```

这一次，任务被跳过了，因为 Ansible 事实显示端口`swp2`已经启用。这显然是一个非常简单的例子，但它展示了你可以如何在网络设备上使用条件语句，这与你之前在本书中已经看到的在 Linux 服务器上使用条件语句的方式非常相似。

这就结束了我们对使用 Ansible 进行网络设备自动化的简要介绍——更深入的工作需要查看网络配置，并需要更多的硬件，因此这超出了本书的范围。然而，我希望这些信息向你展示了 Ansible 可以有效地用于自动化和配置各种网络设备。

# 摘要

快速变化的现代大规模基础设施需要自动化网络任务。幸运的是，Ansible 支持各种网络设备，从专有硬件，如基于 Cisco IOS 的设备，到运行操作系统如 Cumulus Linux 的白盒交换机等开放标准。当涉及管理网络配置时，Ansible 是一个强大而有支持性的工具，它允许你快速而安全地实施变更。你甚至可以在网络中替换整个设备，并且有信心能够通过你的 Ansible playbook 在新设备上放置正确的配置。

在本章中，你了解了自动化网络管理的原因。然后，你看了 Ansible 如何管理网络设备，如何在 Ansible 中启用网络自动化，以及如何找到执行你希望完成的自动化任务所需的 Ansible 模块。然后，通过实际示例，你学会了如何连接到网络设备，如何设置环境变量（并通过堡垒主机连接到隔离网络），以及如何对 Ansible 任务应用条件语句以配置网络设备。

在下一章中，我们将学习如何使用 Ansible 管理 Linux 容器和云基础设施。

# 问题

1.  以下哪个不是 Ansible 用于连接这些网络设备的四种主要连接类型之一？

A) `netconf`

B) `network_cli`

C) `local`

D) `netstat`

E) `httpapi`

1.  真或假：`ansible_network_os`变量被 Ansible 用来理解我们将要使用的设备的设备系列。

A) True

B) False

1.  真或假：为了连接到一个独立网络中的 IOS 路由器，您需要指定主机的特殊连接变量，可能作为清单组变量。

A) 正确

B) 错误

# 进一步阅读

+   有关 Ansible 网络的官方文档：[`docs.ansible.com/ansible/latest/network/index.html`](https://docs.ansible.com/ansible/latest/network/index.html)
