# 前言

Ansible 已经迅速从一个小型的开源编排工具发展成为一款完整的编排和配置管理工具，由红帽公司拥有。在本书中，您将学习如何使用核心 Ansible 模块编写 playbook，部署从基本的 LAMP 堆栈到完整的高可用公共云基础架构。

通过本书，您将学会以下内容：

+   编写自己的 playbook 来配置运行 CentOS 7、Ubuntu 17.04 和 Windows Server 的服务器

+   定义了一个高可用的云基础架构代码，使得很容易将您的基础架构配置与您自己的代码一起分发

+   部署和配置 Ansible Tower 和 Ansible AWX

+   使用社区贡献的角色，并学习如何贡献自己的角色

+   通过几个用例，了解如何在日常角色和项目中使用 Ansible

通过本书，您应该对如何将 Ansible 集成到日常角色中有一个很好的想法，比如系统管理员、开发人员和 DevOps 从业者。

# 这本书适合谁

这本书非常适合想要将他们当前的工作流程转变为可重复使用的 playbook 的系统管理员、开发人员和 DevOps 从业者。不需要先前对 Ansible 的了解。

# 本书涵盖的内容

第一章，*Ansible 简介*，讨论了 Ansible 开发的问题，作者是谁，并谈到了红帽公司在收购 Ansible 后的参与情况。

第二章，*安装和运行 Ansible*，讨论了我们将如何在 macOS 和 Linux 上安装 Ansible，然后介绍其背景。我们还将讨论为什么没有本地的 Windows 安装程序，并介绍在 Windows 10 专业版的 Ubuntu shell 上安装 Ansible。

第三章，*Ansible 命令*，解释了在开始编写和执行更高级的 playbook 之前，我们将先了解 Ansible 命令。在这里，我们将介绍组成 Ansible 的一组命令的用法。

第四章，*部署 LAMP 堆栈*，讨论了使用随 Ansible 提供的各种核心模块部署完整的 LAMP 堆栈。我们将针对本地运行的 CentOS 7 虚拟机进行操作。

第五章，*部署 WordPress*，解释了我们在上一章部署的 LAMP 堆栈作为基础。我们将使用 Ansible 来下载、安装和配置 WordPress。

第六章，*面向多个发行版*，解释了我们将如何调整 playbook，使其可以针对 Ubuntu 17.04 和 CentOS 7 服务器运行。前两章的最终 playbook 已经编写成针对 CentOS 7 虚拟机。

第七章，*核心网络模块*，解释了我们将如何查看随 Ansible 一起提供的核心网络模块。由于这些模块的要求，我们只会涉及这些模块提供的功能。

第八章，*迁移到云*，讨论了我们将如何从使用本地虚拟机转移到使用 Ansible 在 DigitalOcean 中启动 Droplet，然后我们将使用前几章的 playbook 来安装和配置 LAMP 堆栈和 WordPress。

第九章，*构建云网络*，讨论了在 DigitalOcean 中启动服务器后。我们将转移到 Amazon Web Services，然后启动实例。我们需要为它们创建一个网络来进行托管。

第十章，*高可用云部署*，继续我们的 Amazon Web Services 部署。我们将开始将服务部署到上一章中创建的网络中，到本章结束时，我们将留下一个高可用的 WordPress 安装。

第十一章，*构建 VMware 部署*，讨论了允许您与构成典型 VMware 安装的各种组件进行交互的核心模块。

第十二章，*Ansible Windows 模块*，介绍了不断增长的核心 Ansible 模块集，支持并与基于 Windows 的服务器交互。

第十三章，*使用 Ansible 和 OpenSCAP 加固您的服务器*，解释了如何使用 Ansible 安装和执行 OpenSCAP。我们还将研究如何使用 Ansible 解决在 OpenSCAP 扫描期间发现的任何问题。

第十四章，*部署 WPScan 和 OWASP ZAP*，解释了创建一个部署和运行两个安全工具 OWASP ZAP 和 WPScan 的 playbook。然后，使用前几章的 playbook 启动 WordPress 安装以运行它们。

第十五章，*介绍 Ansible Tower 和 Ansible AWX*，介绍了两个与 Ansible 相关的图形界面，商业版的 Ansible Tower 和开源版的 Ansible AWX。

第十六章，*Ansible Galaxy*，讨论了 Ansible Galaxy，这是一个社区贡献角色的在线存储库。在本章中，我们将发现一些最好的可用角色，如何使用它们，以及如何创建自己的角色并将其托管在 Ansible Galaxy 上。

第十七章，*使用 Ansible 的下一步*，教我们如何将 Ansible 集成到我们的日常工作流程中，从与协作服务的交互到使用内置调试器解决 playbook 的故障。我们还将看一些我如何使用 Ansible 的真实例子。

# 为了充分利用本书

为了充分利用本书，我假设你：

+   有一些在 Linux 和 macOS 上使用命令行的经验

+   对如何在 Linux 服务器上安装和配置服务有基本的了解

+   对服务和语言（如 Git、YAML 和虚拟化）有工作知识

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，并按照屏幕上的说明操作。

一旦文件下载完成，请确保您使用最新版本的解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learn-Ansible`](https://github.com/PacktPublishing/Learn-Ansible)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到！看看它们！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/LearnAnsible_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearnAnsible_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“这将创建一个密钥并将其存储在您的`user`目录中的`.ssh`文件夹中。”

代码块设置如下：

```
  config.vm.provider "virtualbox" do |v|
    v.memory = "2024"
    v.cpus = "2"
  end
```

任何命令行输入或输出都将按如下方式编写：

```
$ sudo -H apt-get install python-pip
$ sudo -H pip install ansible
```

**Bold**：表示一个新术语、一个重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中以这种方式出现。这是一个例子：“要做到这一点，打开控制面板，然后点击程序和功能，然后点击打开或关闭 Windows 功能。”

警告或重要说明会出现在这样的形式中。提示和技巧会出现在这样的形式中。
