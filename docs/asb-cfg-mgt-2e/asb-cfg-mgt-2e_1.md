# 第一章：开始使用 Ansible

**Ansible**与今天可用的其他配置管理工具有很大不同。它旨在使配置几乎在各个方面都变得容易，从其简单的英语配置语法到其易于设置。您会发现，Ansible 允许您停止编写自定义配置和部署脚本，而只需简单地继续您的工作。

Ansible 只需要安装在您用来管理基础架构的机器上。它不需要在被管理机上安装客户端，也不需要在使用之前设置任何服务器基础设施。甚至在安装后几分钟内就应该能够使用，正如我们将在本章中向您展示的那样。

本章涵盖的主题如下：

+   安装 Ansible

+   配置 Ansible

+   使用 Ansible 命令行

+   使用 Ansible 管理 Windows 机器

+   如何获取帮助

# 所需的硬件和软件

你将在一台机器上使用 Ansible 命令行，我们将其称为**控制机**，并用它来配置另一台机器，我们将其称为**被管理机**。目前，Ansible 仅支持 Linux 或 OS X 控制机；然而，被管理机可以是 Linux、OS X、其他类 Unix 的机器或 Windows。Ansible 对控制机的要求不多，对被管理机的要求更少。

控制机的要求如下：

+   Python 2.6 或更高版本

+   paramiko

+   PyYAML

+   Jinja2

+   httplib2

+   基于 Unix 的操作系统

被管理机需要 Python 2.4 或更高版本和 simplejson；然而，如果您的 Python 是 2.5 或更高版本，您只需要 Python。被管理的 Windows 机器将需要打开 Windows 远程，并且需要大于 3.0 的 Windows PowerShell 版本。虽然 Windows 机器有更多的要求，但所有工具都是免费提供的，Ansible 项目甚至包括帮助您轻松设置依赖项的脚本。

# 安装方法

如果您想使用 Ansible 来管理一组现有的机器或基础架构，您可能希望使用这些系统上包含的任何软件包管理器。这意味着您将获得 Ansible 的更新，因为您的发行版更新它，这可能会滞后于其他方法几个版本。但是，这意味着您将运行经过测试的版本，可以在您使用的系统上正常工作。

如果您运行现有基础架构，但需要更新版本的 Ansible，可以通过 pip 安装 Ansible。**Pip**是一个用于管理 Python 软件和库包的工具。Ansible 发布的版本一经发布就会推送到 pip，因此如果您与 pip 保持最新，您应该始终运行最新版本。

如果你想象自己开发了很多模块，可能会贡献给 Ansible，你应该运行从源代码安装的版本。由于你将运行最新且测试最少的 Ansible 版本，可能会遇到一两个问题。

## 从您的发行版安装

大多数现代发行版都包含一个自动管理软件包依赖关系和更新的软件包管理器。这使得通过软件包管理器安装 Ansible 成为开始使用 Ansible 最简单的方法；通常只需要一个命令。它也会随着您更新您的机器而更新，尽管可能会滞后一两个版本。以下是在最常见的发行版上安装 Ansible 的命令。如果您使用其他软件包，请参考您的软件包的用户指南或您的发行版的软件包列表：

+   Fedora、RHEL、CentOS 和兼容系统：

```
**$ yum install ansible**

```

+   Ubuntu、Debian 和兼容系统：

```
**$ apt-get install ansible**

```

### 注意

请注意，RHEL 和 CentOS 需要安装 EPEL 存储库。有关 EPEL 的详细信息，包括如何安装它，可以在[`fedoraproject.org/wiki/EPEL`](https://fedoraproject.org/wiki/EPEL)找到。

如果您使用的是 Ubuntu，并希望使用最新版本而不是操作系统提供的版本，可以使用 Ansible 提供的 Ubuntu PPA。有关设置的详细信息，请访问[`launchpad.net/~ansible/+archive/ubuntu/ansible`](https://launchpad.net/~ansible/+archive/ubuntu/ansible)。

## 从 pip 安装

Pip，就像发行版的软件包管理器一样，将处理查找、安装和更新您要求的软件包及其依赖关系。这使得通过 pip 安装 Ansible 与通过软件包管理器安装一样简单。但是需要注意的是，它不会随操作系统更新。此外，更新操作系统可能会破坏您的 Ansible 安装；但是，这不太可能发生。如果您是 Python 用户，可能希望在隔离环境（虚拟环境）中安装 Ansible：这是不受支持的，因为 Ansible 尝试将其模块安装到系统中。您应该使用 pip 在系统范围内安装 Ansible。

以下是通过 pip 安装 Ansible 的命令：

```
**$ pip install ansible**

```

## 从源代码安装

从源代码安装是获取最新版本的好方法，但可能没有经过正确测试，与发布的版本不同。您还需要自行更新到新版本，并确保 Ansible 将继续与操作系统更新一起工作。要克隆`git`存储库并安装它，请运行以下命令。您可能需要 root 访问权限才能执行此操作：

```
**$ git clone git://github.com/ansible/ansible.git**
**$ cd ansible**
**$ sudo make install**

```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

# 设置 Ansible

Ansible 需要能够获取您想要配置的机器的清单，以便对其进行管理。由于清单插件的原因，可以通过多种方式完成这一点。基本安装包含了几种不同的清单插件。我们将在本书的后面介绍这些。现在，我们将介绍简单的主机文件清单。

默认的 Ansible 清单文件名为 hosts，位于`/etc/ansible`。它的格式类似于`INI`文件。组名用方括号括起来，下面的所有内容，直到下一个组标题，都分配给该组。机器可以同时属于多个组。组用于允许您一次配置多台机器。您可以在后续示例中使用组而不是主机名作为主机模式，Ansible 将一次在整个组上运行模块。

在以下示例中，我们有一个名为`webservers`的组中的三台机器，分别是`site01`、`site02`和`site01-dr`。我们还有一个`production`组，其中包括`site01`、`site02`、`db01`和`bastion`。

```
**[webservers]**
**site01**
**site02**
**site01-dr**

**[production]**
**site01**
**site02**
**db01**
**bastion**

```

一旦您将主机放入 Ansible 清单中，就可以开始针对它们运行命令。Ansible 包括一个名为`ping`的简单模块，可让您测试自己与主机之间的连接。让我们从命令行使用 Ansible 针对我们的一台机器，以确认我们可以配置它们。

Ansible 旨在简单易用，开发人员采用的一种方式是使用 SSH 连接到受管机器。然后通过 SSH 连接发送代码并执行它。这意味着您不需要在受管机器上安装 Ansible。这也意味着 Ansible 使用与您已经用于管理机器的相同通道。这使得设置更加容易，因为在大多数情况下不需要任何设置，也不需要在防火墙中打开端口。

首先，我们使用 Ansible 的`ping`模块检查要配置的服务器的连接。该模块简单地连接到以下服务器：

```
**$ ansible site01 -u root -k -m ping**

```

这应该要求输入 SSH 密码，然后产生类似以下的结果：

```
**site01 | success >> {**
 **"changed": false,**
 **"ping": "pong"**
**}**

```

如果您为远程系统设置了 SSH 密钥，您可以省略`-k`参数以跳过提示并使用密钥。您还可以通过在清单中按主机或在全局 Ansible 配置中配置来始终使用特定用户名。

全局设置用户名，编辑`/etc/ansible/ansible.cfg`并更改在`[defaults]`部分设置`remote_user`的行。您还可以更改`remote_port`以更改 Ansible 将 SSH 到的默认端口。这将更改所有机器的默认设置，但可以在清单文件中按服务器或组的基础上进行覆盖。

要在清单文件中设置用户名，只需将`ansible_ssh_user`附加到清单中的行。例如，以下代码部分显示了一个清单，其中`site01`主机使用用户名`root`，`site02`主机使用用户名`daniel`。您还可以使用其他变量。`ansible_ssh_host`变量允许您设置不同的主机名，`ansible_ssh_port`变量允许您设置不同的端口，这在`site01-dr`主机上进行了演示。最后，`db01`主机使用用户名`fred`，并使用`ansible_ssh_private_key_file`设置了私钥。

```
**[webservers]      #1**
**site01 ansible_ssh_user=root     #2**
**site02 ansible_ssh_user=daniel      #3**
**site01-dr ansible_ssh_host=site01.dr ansible_ssh_port=65422      #4**
**[production]      #5**
**site01      #6**
**site02      #7**
**db01 ansible_ssh_user=fred ansible_ssh_private_key_file=/home/fred/.ssh.id_rsa     #8**
**bastion      #9**

```

如果您不愿意让 Ansible 直接访问受管机器上的 root 帐户，或者您的机器不允许 SSH 访问 root 帐户（例如 Ubuntu 的默认配置），您可以配置 Ansible 使用`sudo`来获取 root 访问权限。使用`sudo`的 Ansible 意味着您可以强制执行与以前相同的审计。配置 Ansible 使用`sudo`与配置端口一样简单，只是它需要在受管机器上配置`sudo`。

第一步是向`/etc/sudoers`文件添加一行；在受管节点上，如果选择使用自己的帐户，可能已经设置了这个。您可以使用`sudo`密码，也可以使用无密码`sudo`。如果决定使用密码，您将需要使用`-k`参数到 Ansible，或者在`/etc/ansible/ansible.cfg`中将`ask_sudo_pass`值设置为`true`。要使 Ansible 使用 sudo，请在命令行中添加`--sudo`。

```
**ansible site01 -s -m command -a 'id -a'**

```

如果这样做，它应该返回类似于以下内容：

```
**site01 | success | rc=0 >>**
**uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023**

```

## 在 Windows 上设置它

最近，Ansible 添加了管理 Windows 机器的功能。现在，您可以使用 Ansible 轻松管理 Windows 机器，就像管理 Linux 机器一样。

这使用 Windows PowerShell 远程工具，就像在 Linux 机器上使用 SSH 一样，远程执行模块。已添加了几个新模块，明确支持 Windows，但还为一些现有模块提供了与 Windows 管理的机器一起工作的能力。

要开始管理 Windows 机器，您必须执行一些复杂的设置。您需要按照以下步骤操作：

1.  在清单中创建一些 Windows 机器

1.  安装 Python-winrm 以允许 Ansible 连接到 Windows 机器

1.  升级到 PowerShell 3.0+以支持 Windows 模块

1.  启用 Windows 远程，以便 Ansible 可以连接

Windows 机器与清单中的所有其他机器以相同的方式创建。它们通过`ansible_connection`变量的值进行区分。当`ansible_connection`设置为`winrm`时，它将尝试通过 winrm 连接到远程计算机上的 Windows PowerShell。Ansible 还使用`ansible_ssh_user`，`ansible_ssh_pass`和`ansible_ssh_port`值，就像在其他机器上一样。尽管它们的名称中有 ssh，但它们用于提供将用于连接到 Windows PowerShell 远程服务的端口和凭据。以下是示例 Windows 机器的样子：

```
**[windows]**
**dc.ad.example.com**
**web01.ad.example.com**
**web02.ad.example.com**

**[windows:vars]**
**ansible_connection=winrm**
**ansible_ssh_user=daniel**
**ansible_ssh_pass=s3cr3t**
**ansible_ssh_port=5986**

```

出于安全原因，您可能不希望将密码存储在清单文件中。您可以通过简单地省略`ansible_ssh_user`和`ansible_ssh_pass`变量，并使用 Ansible 的`-k`和`-u`参数来让 Ansible 提示输入密码，就像我们之前为 Unix 系统展示的那样。您也可以选择将它们存储在 Ansible 保险库中，这将在本书的后面介绍。

创建清单后，您需要在控制器机器上安装 winrm Python 库。这个库将使 Ansible 能够连接到 Windows 远程管理服务并配置远程 Windows 系统。

目前，这个库还相当实验性，并且它与 Ansible 的连接并不完美，因此您必须安装与您使用的 Ansible 版本相匹配的特定版本。随着 Ansible 1.8 的发布，这应该会稍微解决一些问题。大多数发行版尚未打包该库，因此您可能希望通过 pip 安装它。作为 root 用户，您需要运行：

```
**$ pip install https://github.com/diyan/pywinrm/archive/df049454a9309280866e0156805ccda12d71c93a.zip**

```

然而，对于更新版本，您应该能够直接运行：

```
**pip install http://github.com/diyan/pywinrm/archive/master.zip**

```

这将安装与 Ansible 1.7 兼容的特定版本的 winrm。对于其他更新版本的 Ansible，您可能需要不同的版本，最终 winrm Python 库应该由不同的发行版打包。您的机器现在将能够连接到并管理 Windows 机器与 Ansible。

接下来，您需要在要管理的机器上执行一些设置步骤。其中第一步是确保您已安装了 PowerShell 3.0 或更高版本。您可以使用以下命令检查已安装的版本：

```
**$PSVersionTable.PSVersion.Major**

```

如果您收到的值不是 3 或高于 3，则需要升级您的 PowerShell 版本。您可以选择通过手动下载和安装最新的 Windows 管理框架来完成此操作，或者您可以使用 Ansible 项目提供的脚本。为了节省空间，我们将在此处解释脚本化安装；手动安装留给读者作为练习。

```
**Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/release1.7.0/examples/scripts/upgrade_to_ps3.ps1 -OutFile upgrade_to_ps3.ps1**
**.\upgrade_to_ps3.ps1**

```

第一个命令从 GitHub 上的 Ansible 项目存储库下载升级脚本并将其保存到磁盘上。第二个命令将检测您的操作系统以下载正确版本的 Windows 管理框架并安装它。

接下来，您需要配置 Windows 远程管理服务。Ansible 项目提供了一个脚本，将自动配置 Windows 远程管理，以符合 Ansible 的预期配置方式。虽然您可以手动设置它，但强烈建议您使用此脚本，以防止错误配置。要下载并运行此脚本，请打开 PowerShell 终端并运行以下命令：

```
**Invoke-WebRequest https://raw.githubusercontent.com/ansible/ansible/release1.7.0/examples/scripts/ConfigureRemotingForAnsible.ps1 -OutFile ConfigureRemotingForAnsible.ps1**
**.\ConfigureRemotingForAnsible.ps1**

```

第一个命令从 GitHub 上的 Ansible 项目下载配置脚本，第二个命令运行它。如果一切正常，您应该从第二个脚本中收到`Ok`的输出。

现在，您应该能够连接到您的机器并使用 Ansible 对其进行配置。与之前一样，让我们运行一个 ping 命令来确认 Ansible 能够远程执行其模块。虽然 Unix 机器可以使用`ping`模块，但 Windows 机器使用`win_ping`模块。使用方式几乎完全相同；但是，由于我们已将密码添加到清单文件中，您不需要`-k`选项。

```
**$ ansible web01.ad.example.com -u daniel -m win_ping**

```

如果一切正常，您应该看到以下输出：

```
**web01.ad.example.com | success >> {**
 **"changed": false,**
 **"ping": "pong"**
**}**

```

输出表明 Ansible 能够连接到 Windows 远程管理服务，成功登录并在远程主机上执行模块。如果这个工作正常，那么您应该能够使用所有其他 Windows 模块来管理您的机器。

# Ansible 的第一步

Ansible 模块以类似于`key=value`的键值对形式接受参数，在远程服务器上执行任务，并将有关任务的信息返回为`JSON`。键值对允许模块在请求时知道该做什么。它们可以是硬编码的值，或者在 playbooks 中可以使用变量，这将在第二章中进行介绍，*简单的 Playbooks*。模块返回的数据让 Ansible 知道托管主机是否有任何更改，或者之后是否应该更改 Ansible 保存的任何信息。

模块通常在 playbooks 中运行，因为这样可以将许多模块链接在一起，但也可以在命令行上使用。之前，我们使用`ping`命令来检查 Ansible 是否已正确设置并能够访问配置的节点。`ping`模块只检查 Ansible 的核心是否能够在远程机器上运行，但实际上什么也不做。

一个稍微更有用的模块名为`setup`。该模块连接到配置的节点，收集有关系统的数据，然后返回这些值。在命令行中运行时，这对我们来说并不特别方便。但是，在 playbook 中，您可以稍后在其他模块中使用收集到的值。

要从命令行运行 Ansible，您需要传递两个参数，通常是三个。首先是要匹配要应用模块的机器的主机模式。其次，您需要提供要运行的模块的名称，以及可选的要传递给模块的参数。对于主机模式，您可以使用组名、机器名、通配符和波浪号(~)，后跟与主机名匹配的正则表达式。或者，为了表示所有这些，您可以使用单词`all`或简单地使用`*`。以这种方式在命令行上运行 Ansible 模块被称为临时的 Ansible 命令。

要在您的节点之一上运行`setup`模块，您需要以下命令行：

```
**$ ansible machinename -u root -k -m setup**

```

`setup`模块将连接到机器并返回一些有用的事实。`setup`模块本身提供的所有事实都以`ansible_`开头，以区别于变量。

该模块将在 Windows 和 Unix 机器上运行。目前，Unix 机器将提供比 Windows 机器更多的信息。但是，随着 Ansible 的新版本发布，您可以期望看到更多的 Windows 功能被包含在 Ansible 中。

```
**machinename | success >> {**
 **"ansible_facts": {**
 **"ansible_distribution": "Microsoft Windows NT 6.3.9600.0",**
 **"ansible_distribution_version": "6.3.9600.0",**
 **"ansible_fqdn": "ansibletest",**
 **"ansible_hostname": "ANSIBLETEST",**
 **"ansible_ip_addresses": [**
 **"100.72.124.51",**
 **"fe80::1fd:fc3b:1eff:350d"**
 **],**
 **"ansible_os_family": "Windows",**
 **"ansible_system": "Win32NT",**
 **"ansible_totalmem": "System.Object[]"**
 **},**
 **"changed": false**
**}**

```

以下是您将使用的最常见值的表格；并非所有这些值都适用于所有机器。特别是 Windows 机器从 setup 模块返回的数据要少得多。

| 字段 | 示例 | 描述 |
| --- | --- | --- |
| `ansible_architecture` | x86_64 | 这是托管机器的架构 |
| `ansible_distribution` | CentOS | 这是托管机器上的 Linux 或 Unix 发行版 |
| `ansible_distribution_version` | 6.3 | 这是先前发行版的版本 |
| `ansible_domain` | example.com | 这是服务器主机名的域名部分 |
| `ansible_fqdn` | machinename.example.com | 这是托管机器的完全限定域名 |
| `ansible_interfaces` | ["lo", "eth0"] | 这是机器拥有的所有接口的列表，包括环回接口 |
| `ansible_kernel` | 2.6.32-279.el6.x86_64 | 这是托管机器上安装的内核版本 |
| `ansible_memtotal_mb` | 996 | 这是托管机器上可用的总内存（以兆字节为单位） |
| `ansible_processor_count` | 1 | 这是托管机器上可用的 CPU 总数 |
| `ansible_virtualization_role` | guest | 这确定了机器是客户机还是主机机器 |
| `ansible_virtualization_type` | kvm | 这是托管机器上的虚拟化设置的类型 |

在 Unix 机器上，这些变量是使用 Python 从受控机器中收集的；如果在远程节点上安装了 facter 或 ohai，`setup`模块将执行它们并返回它们的数据。与其他事实一样，ohai 事实以`ohai_`开头，facter 事实以`facter_`开头。虽然`setup`模块在命令行上似乎不太有用，但一旦开始编写 playbooks，它就会变得有用。请注意，facter 和 ohai 在 Windows 主机上不可用。

如果 Ansible 中的所有模块都像`setup`和`ping`模块那样少，我们将无法在远程机器上进行任何更改。几乎所有 Ansible 提供的其他模块，如`file`模块，都允许我们实际配置远程机器。

`file`模块可以使用单个路径参数调用；这将导致它返回有关所讨论文件的信息。如果给它更多的参数，它将尝试更改文件的属性，并告诉您是否已更改了任何内容。Ansible 模块将告诉您是否已更改任何内容，这在编写 playbooks 时变得更加重要。

您可以调用`file`模块，如以下命令所示，以查看有关`/etc/fstab`的详细信息：

```
**$ ansible machinename -u root -k -m file -a 'path=/etc/fstab'**

```

上述命令应该引发以下响应：

```
**machinename | success >> {**
 **"changed": false,**
 **"group": "root",**
 **"mode": "0644",**
 **"owner": "root",**
 **"path": "/etc/fstab",**
 **"size": 779,**
 **"state":**
 **"file"**
**}**

```

或者，响应可能是类似以下命令以在`/tmp`中创建一个新的测试目录：

```
**$ ansible machinename -u root -k -m file -a 'path=/tmp/teststate=directory mode=0700 owner=root'**

```

上述命令应该返回类似以下内容：

```
**machinename | success >> {**
 **"changed": true,**
 **"group": "root",**
 **"mode": "0700",**
 **"owner": "root",**
 **"path": "/tmp/test",**
 **"size": 4096,**
 **"state": "directory"**
**}**

```

我们可以看到在响应中将`changed`变量设置为`true`，因为目录不存在或具有不同的属性，并且需要进行更改以使其与提供的参数给出的状态匹配。如果使用相同的参数再次运行它，`changed`的值将设置为`false`，这意味着该模块没有对系统进行任何更改。

有几个模块接受与`file`模块类似的参数，其中一个例子是`copy`模块。`copy`模块在控制器机器上获取一个文件，将其复制到受控机器，并根据需要设置属性。例如，要将`/etc/fstab`文件复制到受控机器上的`/tmp`，您将使用以下命令：

```
**$ ansible machinename -m copy -a 'src=/etc/fstab dest=/tmp/fstab'**

```

第一次运行上述命令时，应该返回类似以下内容：

```
**machinename | success >> {**
 **"changed": true,**
 **"dest": "/tmp/fstab",**
 **"group": "root",**
 **"md5sum": "fe9304aa7b683f58609ec7d3ee9eea2f",**
 **"mode": "0700",**
 **"owner": "root",**
 **"size": 637,**
 **"src": "/root/.ansible/tmp/ansible-1374060150.96- 77605185106940/source",**
 **"state": "file"**
**}**

```

还有一个名为`command`的模块，它将在受控机器上运行任意命令。这使您可以配置它以运行任意命令，例如`preprovided`安装程序或自编写脚本；它还可用于重新启动机器。请注意，此模块不在 shell 中运行命令，因此无法执行重定向、使用管道、扩展 shell 变量或后台命令。

Ansible 模块努力防止在不需要时进行更改。这被称为幂等性，并且可以使针对多个服务器运行命令变得更快。不幸的是，Ansible 无法知道您的命令是否已更改任何内容，因此为了帮助它更具幂等性，您必须给它一些帮助。它可以通过`creates`或`removes`参数来实现。如果给出`creates`参数，如果文件名参数存在，则不会运行命令。`removes`参数则相反；如果文件名存在，则会运行命令。

您可以按以下方式运行该命令：

```
**$ ansible machinename -m command -a 'rm -rf /tmp/testing removes=/tmp/testing'**

```

如果没有名为`/tmp/testing`的文件或目录，则命令输出将指示它已被跳过，如下所示：

```
**machinename | skipped**

```

否则，如果文件存在，它将如下所示的代码：

```
**ansibletest | success | rc=0 >>**

```

通常最好使用`command`模块的替代模块。其他模块提供更多选项，并且可以更好地捕捉它们所在的问题领域。例如，在这种情况下，使用`file`模块比使用`command`模块要少得多，因为如果状态设置为`absent`，`file`模块将递归删除某些内容。因此，前面的命令等同于以下命令：

```
**$ ansible machinename -m file -a 'path=/tmp/testing state=absent'**

```

如果您需要在运行命令时使用通常在 shell 中可用的功能，您将需要`shell`模块。这样您就可以使用重定向、管道或作业后台。您可以使用可执行参数选择要使用的 shell。您可以按以下方式使用`shell`模块：

```
**$ ansible machinename -m shell -a '/opt/fancyapp/bin/installer.sh >/var/log/fancyappinstall.log creates=/var/log/fancyappinstall.log'**

```

# 模块帮助

不幸的是，我们没有足够的空间来涵盖 Ansible 中可用的每个模块；幸运的是，Ansible 包含一个名为`ansible-doc`的命令，可以检索帮助信息。所有包含在 Ansible 中的模块都有这些数据；然而，对于从其他地方收集的模块，您可能会找到更少的帮助。`ansible-doc`命令还允许您查看可用的所有模块列表。

要获取可用模块的列表以及每种类型的简短描述，请使用以下命令：

```
**$ ansible-doc -l**

```

要查看特定模块的帮助文件，将其作为`ansible-doc`的单个参数提供。例如，要查看`file`模块的帮助信息，请使用以下命令：

```
**$ ansible-doc file**

```

# 总结

在本章中，我们已经介绍了选择安装类型以安装 Ansible 以及如何构建清单文件以反映您的环境。之后，我们看到了如何以临时方式使用 Ansible 模块来执行简单任务。最后，我们讨论了如何了解系统上可用的模块以及如何使用命令行获取使用模块的说明。

在下一章中，您将学习如何在 playbook 中一起使用多个模块。这使您能够执行比单个模块更复杂的任务。
