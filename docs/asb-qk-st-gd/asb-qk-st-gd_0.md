# 前言

这是一本面向初学系统管理员的 Ansible 指南。它旨在适当地介绍 Ansible 作为自动化和配置管理工具。本书的读者在结束时应该能够通过从真实样本代码中学习每个模块的功能来掌握 Ansible playbook 和模块的基本用法，以帮助实现基础设施和任务的自动化和编排。本书还包含一些额外的高级技巧，供那些希望走得更远并与 Ansible 社区合作的人学习。

# 这本书是为谁准备的

这本书适用于三个主要受众。首先是与 Linux、Windows 或 Mac OS X 打交道的系统管理员。这包括那些在裸机、虚拟基础设施或基于云的环境上工作的人。然后是网络管理员，那些在分布式专有网络设备上工作的人。最后是 DevOps。本书可以帮助他们充分了解他们将部署应用程序的系统的行为，使他们能够相应地编码或建议可以使他们的应用程序受益的修改。

# 本书涵盖的内容

第一章，*什么是 Ansible?*，是对 Ansible 的介绍，并将其与其他配置管理工具进行了比较。

第二章，*Ansible 设置和配置*，解释了如何在多个系统上设置和配置 Ansible。

第三章，*Ansible 清单和 playbook*，是对 Ansible 清单和 playbook 的介绍和概述。

第四章，*Ansible 模块*，涵盖了 Ansible 最常用的模块，并提供了真实样本使用代码。

第五章，*Ansible 自动化基础设施*，列举了 Ansible 在多个基础设施中的用例。

第六章，*用于配置管理的 Ansible 编码*，包含了编写 Ansible playbook 的最佳实践。

第七章，*Ansible Galaxy 和社区角色*，是对 Ansible 社区角色、用法和贡献的介绍。

第八章，*Ansible 高级功能*，是对 Ansible 的一些高级功能的概述，例如 Vault、插件和容器。

# 充分利用本书

在阅读本书之前，您应该对 Linux shell 有基本的了解，并具备一些系统管理技能，以便能够跟随实际示例。此外，一些基本的编码技能在处理 YAML playbooks 时将非常有用。作为可选要求，具备一些基本的配置管理知识将有助于简化本书中的许多要点。

为了能够运行大部分代码，我们建议在至少两台 Linux 机器、一台 Windows 机器和一台 Mac OS X 上运行虚拟环境。对于网络设备测试，您可能需要一个测试网络设备或一些虚拟网络设备。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择 SUPPORT 选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本的软件解压或提取文件夹：

+   Windows 系统使用 WinRAR/7-Zip

+   Mac 系统使用 Zipeg/iZip/UnRarX

+   Linux 系统使用 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Ansible-Quick-Start-Guide`](https://github.com/PacktPublishing/Ansible-Quick-Start-Guide)。如果代码有更新，将在现有的 GitHub 存储库中进行更新。

我们还有来自我们丰富的书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789532937_ColorImages.pdf.`](https://www.packtpub.com/sites/default/files/downloads/9781789532937_ColorImages.pdf)

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。以下是一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```
$link = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$script = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($link, $script)
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```
$link = "https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$script = "$env:temp\ConfigureRemotingForAnsible.ps1"

(New-Object -TypeName System.Net.WebClient).DownloadFile($link, $script)
```

任何命令行输入或输出都以以下方式编写：

```
sudo apt install -y expect
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单中的单词或对话框中的单词会以这种形式出现在文本中。以下是一个例子：“从管理面板中选择系统信息。”

警告或重要提示会出现在这样的形式中。提示和技巧会以这种形式出现。
