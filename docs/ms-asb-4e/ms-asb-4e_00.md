# 前言

欢迎阅读《精通 Ansible》，这是您全面更新的指南，介绍了 Ansible 提供的最有价值的高级功能和功能——自动化和编排工具。本书将为您提供所需的知识和技能，真正理解 Ansible 在基本水平上的功能，包括自 3.0 版本发布以来的所有最新功能和变化。这将使您能够掌握处理当今和未来复杂自动化挑战所需的高级功能。您将了解 Ansible 工作流程，探索高级功能的用例，解决意外行为，通过定制扩展 Ansible，并了解 Ansible 的许多新的重要发展，特别是基础设施和网络供应方面。

# 本书适合对象

本书适用于对 Ansible 核心元素和应用有一定了解，但现在希望通过使用 Ansible 来应用自动化来提高他们的技能的 Ansible 开发人员和运维人员。

# 本书涵盖的内容

[*第一章*]，《Ansible 的系统架构和设计》，介绍了 Ansible 在工程师代表执行任务时的内部细节，它是如何设计的，以及如何使用清单和变量。

[*第二章*]，《从早期的 Ansible 版本迁移》，解释了从 Ansible 2.x 迁移到 3.x 及更高版本时将经历的架构变化，如何使用 Ansible 集合，以及如何构建自己的集合——对于熟悉早期 Ansible 版本的任何人来说，这是必读的。

[*第三章*]，《使用 Ansible 保护您的秘密》，探讨了加密数据和防止秘密在运行时被揭示的工具。

[*第四章*]，《Ansible 和 Windows-不仅仅适用于 Linux》，探讨了将 Ansible 与 Windows 主机集成，以在跨平台环境中实现自动化的方法。

[*第五章*]，《使用 AWX 进行企业基础设施管理》，概述了强大的、开源的图形化管理框架 AWX，以及在企业环境中如何使用它。

[*第六章*]，《解锁 Jinja2 模板的强大功能》，阐述了 Jinja2 模板引擎在 Ansible 中的各种用途，并讨论了如何充分利用其功能。

[*第七章*]，《控制任务条件》，解释了如何更改 Ansible 的默认行为，定制任务错误和更改条件。

[*第八章*]，《使用角色组合可重用的 Ansible 内容》，解释了如何超越在主机上执行松散组织的任务，而是构建干净、可重用和自包含的代码结构，称为角色，以实现相同的最终结果。

[*第九章*]，《故障排除 Ansible》，带您了解可以用于检查、内省、修改和调试 Ansible 操作的各种方法。

[*第十章*]，《扩展 Ansible》，介绍了通过模块、插件和清单来源添加新功能的各种方法。

[*第十一章*]，《通过滚动部署减少停机时间》，解释了常见的部署和升级策略，以展示相关的 Ansible 功能。

[*第十二章*]，《基础设施供应》，研究了用于创建管理基础设施的云基础设施提供商和容器系统。

*第十三章*，*网络自动化*，描述了使用 Ansible 自动化网络设备配置的进展。

# 为了充分利用本书

要跟随本书提供的示例，您需要访问能够运行 Ansible 的计算机平台。目前，Ansible 可以在安装了 Python 2.7 或 Python 3（3.5 及更高版本）的任何机器上运行（Windows 支持控制机，但仅通过在较新版本上运行的 Linux 发行版中的**Windows 子系统 Linux（WSL）**层支持—有关详细信息，请参见*第四章*，*Ansible 和 Windows-不仅适用于 Linux*）。支持的操作系统包括（但不限于）Red Hat、Debian、Ubuntu、CentOS、macOS 和 FreeBSD。

本书使用 Ansible 4.x.x 系列版本。Ansible 安装说明可在[`docs.ansible.com/ansible/latest/installation_guide/intro_installation.html`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)找到。

一些示例使用了 Docker 版本 20.10.8。Docker 安装说明可在[`docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/)找到。

本书中的一些示例使用了**Amazon Web Services（AWS）**和 Microsoft Azure 上的帐户。有关这些服务的更多信息，请访问[`aws.amazon.com/`](https://aws.amazon.com/)和[`azure.microsoft.com`](https://azure.microsoft.com)。我们还深入探讨了使用 Ansible 管理 OpenStack，并且本书中的示例是根据此处的说明针对 DevStack 的单个*一体化*实例进行测试：[`docs.openstack.org/devstack/latest/`](https://docs.openstack.org/devstack/latest/)。

最后，*第十三章**，网络自动化*，在示例代码中使用了 Arista vEOS 4.26.2F 和 Cumulus VX 版本 4.4.0—请参见此处获取更多信息：[`www.arista.com/en/support/software-download`](https://www.arista.com/en/support/software-download)和[`www.nvidia.com/en-gb/networking/ethernet-switching/cumulus-vx/`](https://www.nvidia.com/en-gb/networking/ethernet-switching/cumulus-vx/)。如果您使用本书的数字版本，我们建议您自己输入代码或从书的 GitHub 存储库中访问代码（下一节中提供了链接）。这样做将帮助您避免与复制和粘贴代码相关的任何潜在错误。

# 下载示例代码文件

您可以从 GitHub 上下载本书的示例代码文件，网址为[`github.com/PacktPublishing/Mastering-Ansible-Fourth-Edition`](https://github.com/PacktPublishing/Mastering-Ansible-Fourth-Edition)。如果代码有更新，将在 GitHub 存储库中进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。请查看！

# 实际代码演示

本书的实际代码演示视频可在[`bit.ly/3vvkzbP`](https://bit.ly/3vvkzbP)观看。

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图和图表的彩色图片。您可以在这里下载：`static.packt-cdn.com/downloads/9781801818780_ColorImages.pdf`。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码字词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如：“本书将假定`ansible.cfg`文件中没有设置会影响 Ansible 默认操作的设置”

代码块设置如下：

```
---
plugin: amazon.aws.aws_ec2
boto_profile: default
```

任何命令行输入或输出都将按照以下格式编写：

```
ansible-playbook -i mastery-hosts --vault-id 
test@./password.sh showme.yaml -v
```

**粗体**：表示一个新术语，一个重要的词，或者屏幕上看到的词。例如，菜单或对话框中的词以**粗体**显示。这是一个例子：“您只需导航到您的个人资料首选项页面，然后单击**显示 API 密钥**按钮。”

提示或重要说明

以这种方式出现。
