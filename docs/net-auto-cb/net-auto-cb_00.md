# 前言

《网络自动化食谱》概述了网络自动化的各种主题以及如何使用软件开发实践来设计和操作不同的网络解决方案。我们使用 Ansible 作为我们的框架，介绍网络自动化的主题以及如何使用 Ansible 来管理不同厂商的设备。在第一部分中，我们概述了如何安装和配置 Ansible，专门用于网络自动化的目的。我们将探讨如何使用 Ansible 来管理来自 Cisco、Juniper、Arista 和 F5 等各种厂商的传统网络解决方案。接下来，我们将继续探讨如何利用 Ansible 来构建和扩展来自 AWS、Azure 和 Google Cloud Platform（GCP）等主要云服务提供商的网络解决方案。最后，我们概述了网络自动化中不同的支持开源项目，如 NetBox、Batfish 和 AWX。我们概述了如何将所有这些工具与 Ansible 集成，以构建一个完整的网络自动化框架。

通过本书，您将建立起如何将 Ansible 与不同厂商设备集成以及如何基于 Ansible 构建网络自动化解决方案的坚实基础。此外，您还将了解如何使用各种开源项目，并如何将所有这些解决方案与 Ansible 集成，以构建一个强大而可扩展的网络自动化框架。

# 本书适合对象

本书适合负责组织内部网络设备设计和操作的 IT 专业人员和网络工程师，他们希望扩展自己在使用 Ansible 自动化网络基础设施方面的知识。建议具备基本的网络和 Linux 知识。

# 本书内容

第一章《Ansible 的构建模块》着重介绍了如何安装 Ansible，并描述了 Ansible 的主要构建模块以及如何利用它们来构建高级的 Ansible playbook。

第二章《使用 Ansible 管理 Cisco IOS 设备》着重介绍了如何将 Ansible 与 Cisco IOS 设备集成，以及如何使用 Ansible 配置 Cisco IOS 设备。我们将探讨与 Cisco IOS 设备交互的核心 Ansible 模块。最后，我们还将探讨如何使用 Cisco PyATS 库以及如何将其与 Ansible 集成，以验证 Cisco IOS 和 Cisco IOS-XE 设备上的网络状态。

第三章《使用 Ansible 在服务提供商中自动化 Juniper 设备》描述了如何在服务提供商环境中将 Ansible 与 Juniper 设备集成，以及如何使用 Ansible 管理 Juniper 设备的配置。我们将探讨如何使用核心的 Ansible 模块来管理 Juniper 设备。此外，我们还将探讨 PyEZ 库，该库被 Juniper 定制的 Ansible 模块使用，以扩展 Ansible 在管理 Juniper 设备方面的功能。

第四章《使用 Arista 和 Ansible 构建数据中心网络》概述了如何将 Ansible 与 Arista 设备集成，以使用 EVPN/VXLAN 构建数据中心网络。我们将探讨如何使用核心的 Ansible 模块来管理 Arista 设备，以及如何使用这些模块来配置和验证 Arista 交换机上的网络状态。

第五章《使用 F5 LTM 和 Ansible 自动化应用交付》着重介绍了如何将 Ansible 与 F5 BIG-IP LTM 设备集成，以便为应用交付新的 BIG-IP LTM 设备，并将 BIG-IP 系统设置为应用交付的反向代理。

第六章《使用 NAPALM 和 Ansible 管理多厂商网络》介绍了 NAPALM 库，并概述了如何将该库与 Ansible 集成。我们将探讨如何利用 Ansible 和 NAPALM 来简化多厂商环境的管理。

第七章《使用 Ansible 部署和操作 AWS 网络资源》概述了如何将 Ansible 与您的 AWS 环境集成，以及如何使用 Ansible 描述您的 AWS 基础设施。我们探讨了如何利用核心 Ansible AWS 模块来管理 AWS 中的网络资源，以便使用 Ansible 构建 AWS 网络基础设施。

第八章《使用 Ansible 部署和操作 Azure 网络资源》概述了如何将 Ansible 与您的 Azure 环境集成，以及如何使用 Ansible 描述您的 Azure 基础设施。我们将探讨如何利用核心 Ansible Azure 模块来管理 Azure 中的网络资源，以便使用 Ansible 构建 Azure 网络解决方案。

第九章《使用 Ansible 部署和操作 GCP 网络资源》描述了如何将 Ansible 与您的 GCP 环境集成，以及如何使用 Ansible 描述您的 GCP 基础设施。我们探讨了如何利用核心 Ansible GCP 模块来管理 GCP 中的网络资源，以便使用 Ansible 构建 GCP 网络解决方案。

第十章《使用 Batfish 和 Ansible 进行网络验证》介绍了离线网络验证的 Batfish 框架以及如何将该框架与 Ansible 集成，以便使用 Ansible 和 Batfish 进行离线网络验证。

第十一章《使用 Ansible 和 NetBox 构建网络清单》介绍了 NetBox，这是一个完整的清单系统，用于记录和描述任何网络。我们概述了如何将 Ansible 与 NetBox 集成，以及如何使用 NetBox 数据构建 Ansible 动态清单。

第十二章《使用 AWX 和 Ansible 简化自动化》介绍了 AWX 项目，它扩展了 Ansible，并在 Ansible 之上提供了强大的 GUI 和 API，以简化组织内运行自动化任务。我们概述了 AWX 提供的额外功能以及如何使用它来管理组织内的网络自动化。

第十三章《Ansible 的高级技术和最佳实践》描述了可用于更高级 playbook 的各种最佳实践和高级技术。

# 为了充分利用本书

假定您具有关于不同网络概念的基本知识，例如**开放最短路径优先**（**OSPF**）和**边界网关协议**（**BGP**）。

假定您具有 Linux 的基本知识，包括如何在 Linux 机器上创建文件和文件夹以及安装软件的知识。

| 本书涵盖的软件/硬件 | 操作系统要求 |
| --- | --- |
| Ansible 2.9 | CentOS 7 |
| Python 3.6.8 |  |

**如果您使用的是本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库访问代码（链接在下一节中提供）。这样做将有助于避免与复制和粘贴代码相关的任何潜在错误。**

# 下载示例代码文件

您可以从您在[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Network-Automation-Cookbook`](https://github.com/PacktPublishing/Network-Automation-Cookbook)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789956481_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781789956481_ColorImages.pdf)。

# 代码演示

代码演示视频基于 Ansible 版本 2.8.5。该代码还经过了 2.9.2 版本的测试，可以正常运行。

访问以下链接，查看代码运行的视频：

[`bit.ly/34JooNp`](https://bit.ly/34JooNp)

# 惯例使用

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄。这是一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

一段代码设置如下：

```
$ cat ansible.cfg

[defaults]
 inventory=hosts
 retry_files_enabled=False
 gathering=explicit
 host_key_checking=False
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```
- name: Configure ACL on IOS-XR
  hosts: all
  serial: 1
  tags: deploy
  tasks:
    - name: Backup Config
      iosxr_config:
        backup:
 when: not ansible_check_mode    - name: Deploy ACLs
      iosxr_config:
        src: acl_conf.cfg
        match: line
 when: not ansible_check_mode
```

任何命令行输入或输出都写成如下形式：

```
$ python3 -m venv dev
$ source dev/bin/activate
```

**粗体**：表示一个新术语，一个重要单词，或者您在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会出现在这样的地方。提示和技巧会出现在这样的地方。

# 章节

在本书中，您会经常看到几个标题（*准备工作*，*如何做...*，*它是如何工作的...*，*还有更多...*和*另请参阅*）。

为了清晰地说明如何完成一个配方，使用以下各节：

# 准备工作

本节告诉您在配方中可以期待什么，并描述如何设置配方所需的任何软件或初步设置。

# 如何做...

本节包含了遵循配方所需的步骤。

# 它是如何工作的...

本节通常包括对上一节发生的事情的详细解释。

# 还有更多...

本节包含有关配方的其他信息，以使您对配方更加了解。

# 另请参阅

本节提供了有用的链接，指向配方的其他有用信息。
