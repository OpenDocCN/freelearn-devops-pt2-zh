# 第一章：开始使用 Ansible

Ansible 使您能够使用诸如 SSH 和 WinRM 等本机通信协议轻松一致和可重复地部署应用程序和系统。也许最重要的是，Ansible 是无代理的，因此在受管系统上不需要安装任何东西（除了 Python，这些天大多数系统都有）。因此，它使您能够为您的环境构建一个简单而强大的自动化平台。

安装 Ansible 简单直接，并且适用于大多数现代系统。它的架构是无服务器和无代理的，因此占用空间很小。您可以选择从中央服务器或您自己的笔记本电脑上运行它——完全取决于您。您可以从一个 Ansible 控制机器管理单个主机到数十万个远程主机。所有远程机器都可以（通过编写足够的 playbooks）由 Ansible 管理，并且一切创建正确的话，您可能再也不需要单独登录这些机器了。

在本章中，我们将开始教授您实际技能，涵盖 Ansible 的基本原理，从如何在各种操作系统上安装 Ansible 开始。然后，我们将看看如何配置 Windows 主机以使其能够通过 Ansible 自动化进行管理，然后深入探讨 Ansible 如何连接到其目标主机的主题。然后我们将看看节点要求以及如何验证您的 Ansible 安装，最后看看如何获取和运行最新的 Ansible 源代码，如果您希望为其开发做出贡献或获得最新的功能。

在本章中，我们将涵盖以下主题：

+   安装和配置 Ansible

+   了解您的 Ansible 安装

+   从源代码运行与预构建的 RPM 包

# 技术要求

Ansible 有一组相当简单的系统要求——因此，您应该会发现，如果您有一台能够运行 Python 的机器（无论是笔记本电脑、服务器还是虚拟机），那么您就可以在上面运行 Ansible。在本章的后面，我们将演示在各种操作系统上安装 Ansible 的方法，因此您可以决定哪些操作系统适合您。

前述声明的唯一例外是 Microsoft Windows——尽管 Windows 有 Python 环境可用，但目前还没有 Windows 的原生构建。运行更高版本 Windows 的读者可以使用 Windows 子系统来安装 Ansible（以下简称 WSL），并按照后面为所选的 WSL 环境（例如，如果您在 WSL 上安装了 Ubuntu，则应该简单地按照本章中为 Ubuntu 安装 Ansible 的说明进行操作）的程序进行操作。

# 安装和配置 Ansible

Ansible 是用 Python 编写的，因此可以在各种系统上运行。这包括大多数流行的 Linux、FreeBSD 和 macOS 版本。唯一的例外是 Windows，尽管存在原生的 Python 发行版，但目前还没有原生的 Ansible 构建。因此，在撰写本文时，您最好的选择是在 WSL 下安装 Ansible，就像在本机 Linux 主机上运行一样。

一旦您确定了要运行 Ansible 的系统，安装过程通常是简单直接的。在接下来的章节中，我们将讨论如何在各种不同的系统上安装 Ansible，因此大多数读者应该能够在几分钟内开始使用 Ansible。

# 在 Linux 和 FreeBSD 上安装 Ansible

Ansible 的发布周期通常约为四个月，在这个短暂的发布周期内，通常会有许多变化，从较小的错误修复到较大的错误修复，到新功能，甚至有时对语言进行根本性的更改。不仅可以使用本地包来快速上手并保持最新状态，而且可以使用本地包来保持最新状态。

例如，如果你希望在诸如 CentOS、Fedora、Red Hat Enterprise Linux（RHEL）、Debian 和 Ubuntu 等 Linux 发行版上运行最新版本的 Ansible，我强烈建议你使用操作系统包管理器，如基于 Red Hat 的发行版上的`yum`或基于 Debian 的发行版上的`apt`。这样，每当你更新操作系统时，你也会同时更新 Ansible。

当然，可能是因为你需要保留特定版本的 Ansible 以用于特定目的——也许是因为你的 playbooks 已经经过了测试。在这种情况下，你几乎肯定会选择另一种安装方法，但这超出了本书的范围。此外，建议在可能的情况下，按照记录的最佳实践创建和维护你的 playbooks，这应该意味着它们能够在大多数 Ansible 升级中生存下来。

以下是一些示例，展示了如何在几种 Linux 发行版上安装 Ansible：

+   **在 Ubuntu 上安装 Ansible：**要在 Ubuntu 上安装最新版本的 Ansible 控制机，`apt`包装工具使用以下命令很容易：

```
$ sudo apt-get update 
$ sudo apt-get install software-properties-common 
$ sudo apt-add-repository --yes --update ppa:ansible/ansible 
$ sudo apt-get install ansible
```

如果你正在运行较旧版本的 Ubuntu，你可能需要用`python-software-properties`替换`software-properties-common`。

+   **在 Debian 上安装 Ansible：**你应该将以下行添加到你的`/etc/apt/sources.list`文件中：

```
deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main
```

你会注意到在上述配置行中出现了`ubuntu`一词，以及`trusty`，这是一个 Ubuntu 版本。在撰写本文时，Debian 版本的 Ansible 是从 Ubuntu 的 Ansible 仓库中获取的，并且可以正常工作。你可能需要根据你的 Debian 版本更改上述配置中的版本字符串，但对于大多数常见用例，这里引用的行就足够了。

完成后，你可以按以下方式在 Debian 上安装 Ansible：

```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367 
$ sudo apt-get update 
$ sudo apt-get install ansible
```

+   **在 Gentoo 上安装 Ansible：**要在 Gentoo 上安装最新版本的 Ansible 控制机，`portage`包管理器使用以下命令很容易：

```
$ echo 'app-admin/ansible' >> /etc/portage/package.accept_keywords
$ emerge -av app-admin/ansible
```

+   **在 FreeBSD 上安装 Ansible：**要在 FreeBSD 上安装最新版本的 Ansible 控制机，PKG 管理器使用以下命令很容易：

```
$ sudo pkg install py36-ansible
$ sudo make -C /usr/ports/sysutils/ansible install
```

+   **在 Fedora 上安装 Ansible：**要在 Fedora 上安装最新版本的 Ansible 控制机，`dnf`包管理器使用以下命令很容易：

```
$ sudo dnf -y install ansible
```

+   **在 CentOS 上安装 Ansible：**要在 CentOS 或 RHEL 上安装最新版本的 Ansible 控制机，`yum`包管理器使用以下命令很容易：

```
$ sudo yum install epel-release
$ sudo yum -y install ansible
```

如果你在 RHEL 上执行上述命令，你必须确保 Ansible 仓库已启用。如果没有，你需要使用以下命令启用相关仓库：

```
$ sudo subscription-manager repos --enable rhel-7-server-ansible-2.9-rpms
```

+   **在 Arch Linux 上安装 Ansible：**要在 Arch Linux 上安装最新版本的 Ansible 控制机，`pacman`包管理器使用以下命令很容易：

```
$ pacman -S ansible
```

一旦你在你使用的特定 Linux 发行版上安装了 Ansible，你就可以开始探索。让我们从一个简单的例子开始——当你运行`ansible`命令时，你会看到类似以下的输出：

```
$ ansible --version
ansible 2.9.6
 config file = /etc/ansible/ansible.cfg
 configured module search path = [u'/home/jamesf_local/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/dist-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.17 (default, Nov 7 2019, 10:07:09) [GCC 9.2.1 20191008]
```

那些希望测试来自 GitHub 最新版本的 Ansible 的人可能对构建 RPM 软件包以安装到控制机器感兴趣。当然，这种方法只适用于基于 Red Hat 的发行版，如 Fedora、CentOS 和 RHEL。为此，您需要从 GitHub 存储库克隆源代码，并按以下方式构建 RPM 软件包：

```
$ git clone https://github.com/ansible/ansible.git
$ cd ./ansible
$ make rpm
$ sudo rpm -Uvh ./rpm-build/ansible-*.noarch.rpm
```

现在您已经了解了如何在 Linux 上安装 Ansible，我们将简要介绍如何在 macOS 上安装 Ansible。

# 在 macOS 上安装 Ansible

在本节中，您将学习如何在 macOS 上安装 Ansible。最简单的安装方法是使用 Homebrew，但您也可以使用 Python 软件包管理器。让我们从安装 Homebrew 开始，这是 macOS 的快速便捷的软件包管理解决方案。

如果您在 macOS 上尚未安装 Homebrew，可以按照此处的详细说明轻松安装它：

+   安装 Homebrew：通常，这里显示的两个命令就足以在 macOS 上安装 Homebrew：

```
$ xcode-select --install
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

如果您已经为其他目的安装了 Xcode 命令行工具，您可能会看到以下错误消息：

```
xcode-select: error: command line tools are already installed, use "Software Update" to update
```

您可能希望在 macOS 上打开 App Store 并检查是否需要更新 Xcode，但只要安装了命令行工具，您的 Homebrew 安装应该顺利进行。

如果您希望确认您的 Homebrew 安装成功，可以运行以下命令，它会警告您有关安装的任何潜在问题，例如，以下输出警告我们，尽管 Homebrew 已成功安装，但它不在我们的`PATH`中，因此我们可能无法运行任何可执行文件而不指定它们的绝对路径：

```
$ brew doctor
Please note that these warnings are just used to help the Homebrew maintainers
with debugging if you file an issue. If everything you use Homebrew for is
working fine: please don't worry or file an issue; just ignore this. Thanks!

Warning: Homebrew's sbin was not found in your PATH but you have installed
formulae that put executables in /usr/local/sbin.
Consider setting the PATH for example like so
 echo 'export PATH="/usr/local/sbin:$PATH"' >> ~/.bash_profile
```

+   安装 Python 软件包管理器（pip）：如果您不希望使用 Homebrew 安装 Ansible，您可以使用以下简单命令安装`pip`：

```
$ sudo easy_install pip
```

还要检查您的 Python 版本是否至少为 2.7，因为旧版本的 Ansible 无法运行（几乎所有现代 macOS 安装都应该是这种情况）：

```
$ python --version
Python 2.7.16
```

您可以使用 Homebrew 或 Python 软件包管理器在 macOS 上安装最新版本的 Ansible，方法如下：

+   通过 Homebrew 安装 Ansible：要通过 Homebrew 安装 Ansible，请运行以下命令：

```
$ brew install ansible
```

+   通过 Python 软件包管理器（pip）安装 Ansible：要通过`pip`安装 Ansible，请使用以下命令：

```
$ sudo pip install ansible
```

如果您有兴趣直接从 GitHub 运行最新的 Ansible 开发版本，那么您可以通过运行以下命令来实现：

```
$ pip install git+https://github.com/ansible/ansible.git@devel 
```

现在您已经使用您喜欢的方法安装了 Ansible，您可以像以前一样运行`ansible`命令，如果一切按计划进行，您将看到类似以下的输出：

```
$ ansible --version
ansible 2.9.6
  config file = None
  configured module search path = ['/Users/james/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/2.9.4_1/libexec/lib/python3.8/site-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.8.1 (default, Dec 27 2019, 18:05:45) [Clang 11.0.0 (clang-1100.0.33.16)]
```

如果您正在运行 macOS 10.9，使用`pip`安装 Ansible 可能会遇到问题。以下是一个解决此问题的解决方法：

```
$ sudo CFLAGS=-Qunused-arguments CPPFLAGS=-Qunused-arguments pip install ansible
```

如果您想更新您的 Ansible 版本，`pip`可以通过以下命令轻松实现：

```
$ sudo pip install ansible --upgrade
```

同样，如果您使用的是`brew`命令进行安装，也可以使用该命令进行升级：

```
$ brew upgrade ansible
```

现在您已经学会了在 macOS 上安装 Ansible 的步骤，让我们看看如何为 Ansible 配置 Windows 主机以进行自动化。

# 为 Ansible 配置 Windows 主机

正如前面讨论的，Windows 上没有直接的 Ansible 安装方法，建议在可用的情况下安装 WSL，并像在本章前面概述的过程中那样安装 Ansible，就像在 Linux 上本地运行一样。

尽管存在这种限制，但是 Ansible 并不仅限于管理 Linux 和基于 BSD 的系统，它能够使用本机 WinRM 协议对 Windows 主机进行无代理管理，使用 PowerShell 模块和原始命令，这在每个现代 Windows 安装中都可用。在本节中，您将学习如何配置 Windows 以启用 Ansible 的任务自动化。

让我们看看在自动化 Windows 主机时，Ansible 能做些什么：

+   收集远程主机的信息。

+   安装和卸载 Windows 功能。

+   管理和查询 Windows 服务。

+   管理用户账户和用户列表。

+   使用 Chocolatey（Windows 的软件存储库和配套管理工具）来管理软件包。

+   执行 Windows 更新。

+   从远程机器获取多个文件到 Windows 主机。

+   在目标主机上执行原始的 PowerShell 命令和脚本。

Ansible 允许你通过连接本地用户或域用户来自动化 Windows 机器上的任务。你可以像在 Linux 发行版上使用`sudo`命令一样，以管理员身份运行操作，使用 Windows 的`runas`支持。

另外，由于 Ansible 是开源软件，你可以通过创建自己的 PowerShell 模块或者发送原始的 PowerShell 命令来扩展其功能。例如，信息安全团队可以轻松地管理文件系统 ACL、配置 Windows 防火墙，并使用本地 Ansible 模块和必要时的原始命令来管理主机名和域成员资格。

Windows 主机必须满足以下要求，以便 Ansible 控制机器与之通信：

+   Ansible 尝试支持所有在 Microsoft 的当前或扩展支持下的 Windows 版本，包括桌面平台，如 Windows 7、8.1 和 10，以及服务器操作系统，包括 Windows Server 2008（和 R2）、2012（和 R2）、2016 和 2019。

+   你还需要在 Windows 主机上安装 PowerShell 3.0 或更高版本，以及至少.NET 4.0。

+   你需要创建和激活一个 WinRM 监听器，这将在后面详细描述。出于安全原因，这不是默认启用的。

让我们更详细地看一下如何准备 Windows 主机以便被 Ansible 自动化：

1.  关于先决条件，你必须确保 Windows 机器上安装了 PowerShell 3.0 和.NET Framework 4.0。如果你仍在使用旧版本的 PowerShell 或.NET Framework，你需要升级它们。你可以手动执行这个过程，或者以下的 PowerShell 脚本可以自动处理：

```
$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Upgrade-PowerShell.ps1" 
$file = "$env:temp\Upgrade-PowerShell.ps1" (New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file) 

Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force &$file -Verbose Set-ExecutionPolicy -ExecutionPolicy Restricted -Force
```

这个脚本通过检查需要安装的程序（如.NET Framework 4.5.2）和所需的 PowerShell 版本，如果需要的话重新启动，并设置用户名和密码参数。脚本将在重新启动时自动重新启动和登录，因此不需要更多的操作，脚本将一直持续，直到 PowerShell 版本与目标版本匹配。

如果用户名和密码参数没有设置，脚本将要求用户在必要时重新启动并手动登录，下次用户登录时，脚本将在中断的地方继续。这个过程会一直持续，直到主机满足 Ansible 自动化的要求。

1.  当 PowerShell 升级到至少 3.0 版本后，下一步将是配置 WinRM 服务，以便 Ansible 可以连接到它。WinRM 服务配置定义了 Ansible 如何与 Windows 主机进行交互，包括监听端口和协议。

如果以前从未设置过 WinRM 监听器，你有三种选项可以做到这一点：

+   首先，你可以使用`winrm quickconfig`来配置 HTTP，使用`winrm quickconfig -transport:https`来配置 HTTPS。这是在需要在域环境之外运行并创建一个简单监听器时使用的最简单的方法。这个过程的优势在于它会在 Windows 防火墙中打开所需的端口，并自动启动 WinRM 服务。

+   如果你在域环境中运行，我强烈建议使用**组策略对象**（**GPOs**），因为如果主机是域成员，那么配置会自动完成，无需用户输入。有许多可用的文档化程序可以做到这一点，由于这是一个非常 Windows 领域中心的任务，它超出了本书的范围。

+   最后，您可以通过运行以下 PowerShell 命令创建具有特定配置的监听器：

```
$selector_set = @{
    Address = "*"
    Transport = "HTTPS"
}
$value_set = @{
    CertificateThumbprint = "E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE"
}

New-WSManInstance -ResourceURI "winrm/config/Listener" -SelectorSet $selector_set -ValueSet $value_set
```

前面的`CertificateThumbprint`应该与您之前创建或导入到 Windows 证书存储中的有效 SSL 证书的指纹匹配。

如果您正在运行 PowerShell v3.0，您可能会遇到 WinRM 服务的问题，该问题限制了可用内存的数量。这是一个已知的错误，并且有一个热修复程序可用于解决它。这里提供了一个应用此热修复程序的示例过程（用 PowerShell 编写）：

```
$url = "https://raw.githubusercontent.com/jborean93/ansible-windows/master/scripts/Install-WMF3Hotfix.ps1" 
$file = "$env:temp\Install-WMF3Hotfix.ps1" 

(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file) powershell.exe -ExecutionPolicy ByPass -File $file -Verbose
```

配置 WinRM 监听器可能是一个复杂的任务，因此重要的是能够检查您的配置过程的结果。以下命令（可以从命令提示符中运行）将显示当前的 WinRM 监听器配置：

```
winrm enumerate winrm/config/Listener
```

如果一切顺利，您应该会得到类似于此的输出：

```
Listener
    Address = *
    Transport = HTTP
    Port = 5985
    Hostname
    Enabled = true
    URLPrefix = wsman
    CertificateThumbprint
    ListeningOn = 10.0.2.15, 127.0.0.1, 192.168.56.155, ::1, fe80::5efe:10.0.2.15%6, fe80::5efe:192.168.56.155%8, fe80::
ffff:ffff:fffe%2, fe80::203d:7d97:c2ed:ec78%3, fe80::e8ea:d765:2c69:7756%7

Listener
    Address = *
    Transport = HTTPS
    Port = 5986
    Hostname = SERVER2016
    Enabled = true
    URLPrefix = wsman
    CertificateThumbprint = E6CDAA82EEAF2ECE8546E05DB7F3E01AA47D76CE
    ListeningOn = 10.0.2.15, 127.0.0.1, 192.168.56.155, ::1, fe80::5efe:10.0.2.15%6, fe80::5efe:192.168.56.155%8, fe80::
ffff:ffff:fffe%2, fe80::203d:7d97:c2ed:ec78%3, fe80::e8ea:d765:2c69:7756%7
```

根据前面的输出，有两个活动的监听器——一个监听 HTTP 端口`5985`，另一个监听 HTTPS 端口`5986`，提供更高的安全性。另外，前面的输出还显示了以下参数的解释：

+   `传输`：这应该设置为 HTTPS 或 HTTPS，尽管强烈建议您使用 HTTPS 监听器，以确保您的自动化命令不会受到窥探或操纵。

+   `端口`：这是监听器操作的端口，默认情况下为`5985`（HTTP）或`5986`（HTTPS）。

+   `URL 前缀`：这是与之通信的 URL 前缀，默认情况下为`wsman`。如果更改它，您必须在 Ansible 控制主机上设置`ansible_winrm_path`主机为相同的值。

+   `CertificateThumbprint`：如果在 HTTPS 监听器上运行，这是连接使用的 Windows 证书存储的证书指纹。

如果您在设置 WinRM 监听器后需要调试任何连接问题，您可能会发现以下命令很有价值，因为它们在 Windows 主机之间执行基于 WinRM 的连接，而不使用 Ansible，因此您可以使用它们来区分您可能遇到的问题是与您的 Ansible 主机相关还是 WinRM 监听器本身存在问题：

```
# test out HTTP
winrs -r:http://<server address>:5985/wsman -u:Username -p:Password ipconfig 
# test out HTTPS (will fail if the cert is not verifiable)
winrs -r:https://<server address>:5986/wsman -u:Username -p:Password -ssl ipconfig 

# test out HTTPS, ignoring certificate verification
$username = "Username"
$password = ConvertTo-SecureString -String "Password" -AsPlainText -Force
$cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $username, $password

$session_option = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
Invoke-Command -ComputerName server -UseSSL -ScriptBlock { ipconfig } -Credential $cred -SessionOption $session_option
```

如果前面的命令中的任何一个失败，您应该在尝试设置或配置 Ansible 控制主机之前调查您的 WinRM 监听器设置。

在这个阶段，Windows 应该准备好通过 WinRM 接收来自 Ansible 的通信。要完成此过程，您还需要在 Ansible 控制主机上执行一些额外的配置。首先，您需要安装`winrm` Python 模块，这取决于您的控制主机的配置，可能已经安装或尚未安装。安装方法会因操作系统而异，但通常可以使用`pip`在大多数平台上安装如下：

```
$ pip install winrm
```

完成后，您需要为 Windows 主机定义一些额外的清单变量——现在不要太担心清单，因为我们将在本书的后面部分介绍这些。以下示例仅供参考：

```
[windows]
192.168.1.52

[windows:vars]
ansible_user=administrator
ansible_password=password
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

最后，您应该能够运行 Ansible `ping`模块，执行类似以下命令的端到端连接性测试（根据您的清单进行调整）：

```
$ ansible -i inventory -m ping windows
192.168.1.52 | SUCCESS => {
 "changed": false,
 "ping": "pong"
}
```

现在您已经学会了为 Ansible 配置 Windows 主机所需的步骤，让我们在下一节中看看如何通过 Ansible 连接多个主机。

# 了解您的 Ansible 安装

在本章的这个阶段，无论你选择的操作系统是什么，你应该已经有一个可用的 Ansible 安装，可以开始探索自动化的世界。在本节中，我们将进行对 Ansible 基础知识的实际探索，以帮助你了解如何使用它。一旦掌握了这些基本技能，你就会有足够的知识来充分利用本书的其余部分。让我们从概述 Ansible 如何连接到非 Windows 主机开始。

# 理解 Ansible 如何连接到主机

除了 Windows 主机（如前一节末讨论的），Ansible 使用 SSH 协议与主机通信。Ansible 设计选择这种方式的原因有很多，其中最重要的是几乎每台 Linux/FreeBSD/macOS 主机都内置了它，许多网络设备（如交换机和路由器）也是如此。这个 SSH 服务通常与操作系统身份验证堆栈集成，使你能够利用诸如 Kerberos 等功能来提高身份验证安全性。此外，OpenSSH 的功能，如 `ControlPersist`，用于增加自动化任务的性能和用于网络隔离和安全的 SSH 跳转主机。

`ControlPersist` 在大多数现代 Linux 发行版中默认启用作为 OpenSSH 服务器安装的一部分。然而，在一些较旧的操作系统上，如 Red Hat Enterprise Linux 6（和 CentOS 6），它不受支持，因此你将无法使用它。Ansible 自动化仍然是完全可能的，但更长的 playbooks 可能会运行得更慢。

Ansible 使用与你已经熟悉的相同的身份验证方法，SSH 密钥通常是最简单的方法，因为它们消除了用户在每次运行 playbook 时输入身份验证密码的需要。然而，这绝不是强制性的，Ansible 通过使用 `--ask-pass` 开关支持密码身份验证。如果你连接到主机上的一个非特权帐户，并且需要执行 Ansible 等效的以 `sudo` 运行命令，你也可以在运行 playbook 时添加 `--ask-become-pass`，以允许在运行时指定这个。

自动化的目标是能够安全地运行任务，但最少的用户干预。因此，强烈建议你使用 SSH 密钥进行身份验证，如果你有多个密钥需要管理，那么一定要使用 `ssh-agent`。

每个 Ansible 任务，无论是单独运行还是作为复杂 playbook 的一部分运行，都是针对清单运行的。清单就是你希望运行自动化命令的主机列表。Ansible 支持各种清单格式，包括使用动态清单，它可以自动从编排提供程序中填充自己（例如，你可以动态生成一个 Ansible 清单，从你的 Amazon EC2 实例中，这意味着你不必跟上云基础设施中的所有变化）。

动态清单插件已经为大多数主要的云提供商（例如，Amazon EC2，Google Cloud Platform 和 Microsoft Azure），以及本地系统（如 OpenShift 和 OpenStack）编写。甚至还有 Docker 的插件。开源软件的美妙之处在于，对于大多数你能想到的主要用例，有人已经贡献了代码，所以你不需要自己去弄清楚或编写它。

Ansible 的无代理架构以及它不依赖于 SSL 的事实意味着你不需要担心 DNS 未设置或者由于 NTP 不工作而导致的时间偏移问题——事实上，这些都可以由 Ansible playbook 执行！事实上，Ansible 确实是设计用来从几乎空白的操作系统镜像中运行你的基础设施。

目前，让我们专注于 INI 格式的清单。下面是一个示例，其中有四台服务器，每台服务器分成两个组。可以对整个清单（即所有四台服务器）、一个或多个组（例如`webservers`）甚至单个服务器运行 Ansible 命令和 playbooks：

```
[webservers]
web1.example.com
web2.example.com

[apservers]
ap1.example.com
ap2.example.com
```

让我们使用这个清单文件以及 Ansible 的`ping`模块，用于测试 Ansible 是否能够成功在所讨论的清单主机上执行自动化任务。以下示例假设您已将清单安装在默认位置，通常为`/etc/ansible/hosts`。当您运行以下`ansible`命令时，您将看到类似于这样的输出：

```
$ ansible webservers -m ping 
web1.example.com | SUCCESS => {
 "changed": false, 
 "ping": "pong"
}
web2.example.com | SUCCESS => {
 "changed": false, 
 "ping": "pong"
}
$
```

请注意，`ping`模块仅在`webservers`组中的两台主机上运行，而不是整个清单——这是因为我们在命令行参数中指定了这一点。

`ping`模块是 Ansible 的成千上万个模块之一，所有这些模块都执行一组给定的任务（从在主机之间复制文件到文本替换，再到复杂的网络设备配置）。同样，由于 Ansible 是开源软件，有大量的编码人员在编写和贡献模块，这意味着如果您能想到一个任务，可能已经有一个 Ansible 模块。即使没有模块存在的情况下，Ansible 支持发送原始 shell 命令（或者对于 Windows 主机的 PowerShell 命令），因此即使在这种情况下，您也可以完成所需的任务，而无需离开 Ansible。

只要 Ansible 控制主机能够与清单中的主机通信，您就可以自动化您的任务。但是，值得考虑一下您放置控制主机的位置。例如，如果您专门使用一组 Amazon EC2 机器，您的 Ansible 控制机器最好是一个 EC2 实例——这样，您就不需要通过互联网发送所有自动化命令。这也意味着您不需要将 EC2 主机的 SSH 端口暴露给互联网，因此使它们更安全。

到目前为止，我们已经简要解释了 Ansible 如何与其目标主机通信，包括清单是什么以及对所有主机进行 SSH 通信的重要性，除了 Windows 主机。在下一节中，我们将通过更详细地查看如何验证您的 Ansible 安装来进一步了解这一点。

# 验证 Ansible 安装

在本节中，您将学习如何使用简单的临时命令验证您的 Ansible 安装。

正如之前讨论的，Ansible 可以通过多种方式对目标主机进行身份验证。在本节中，我们将假设您希望使用 SSH 密钥，并且您已经生成了公钥和私钥对，并将公钥应用于您将自动化任务的所有目标主机。

`ssh-copy-id`实用程序非常有用，可以在继续之前将您的公共 SSH 密钥分发到目标主机。例如命令可能是`ssh-copy-id -i ~/.ssh/id_rsa ansibleuser@web1.example.com`。

为了确保 Ansible 可以使用您的私钥进行身份验证，您可以使用`ssh-agent`——命令显示了如何启动`ssh-agent`并将您的私钥添加到其中的简单示例。当然，您应该将路径替换为您自己私钥的路径：

```
$ ssh-agent bash 
$ ssh-add ~/.ssh/id_rsa
```

正如我们在前一节中讨论的，我们还必须为 Ansible 定义一个清单。下面是另一个简单的示例：

```
[frontends]
frt01.example.com
frt02.example.com
```

我们在上一节中使用的`ansible`命令有两个重要的开关，您几乎总是会使用：`-m <MODULE_NAME>`在您指定的清单主机上运行一个模块，还可以使用`-a OPT_ARGS`开关传递模块参数。使用`ansible`二进制运行的命令称为临时命令。

以下是三个简单示例，演示了临时命令-它们也对验证您控制机器上的 Ansible 安装和目标主机的配置非常有价值，如果配置的任何部分存在问题，它们将返回错误：

+   **Ping 主机**：您可以使用以下命令对您的库存主机执行 Ansible“ping”：

```
$ ansible frontends -i hosts -m ping
```

+   显示收集的事实：您可以使用以下命令显示有关您的库存主机的收集事实：

```
$ ansible frontends -i hosts -m setup | less
```

+   **过滤收集的事实**：您可以使用以下命令过滤收集的事实：

```
$ ansible frontends -i hosts -m setup -a "filter=ansible_distribution*"
```

对于您运行的每个临时命令，您将以 JSON 格式获得响应-以下示例输出是成功运行`ping`模块的结果：

```
$ ansible frontends -m ping 
frontend01.example.com | SUCCESS => {
 "changed": false, 
 "ping": "pong"
}
frontend02.example.com | SUCCESS => {
 "changed": false, 
 "ping": "pong"
}
```

Ansible 还可以收集并返回有关目标主机的“事实”-事实是有关主机的各种有用信息，从 CPU 和内存配置到网络参数，再到磁盘几何。这些事实旨在使您能够编写智能的 playbook，执行条件操作-例如，您可能只想在具有 4GB 以上 RAM 的主机上安装特定软件包，或者仅在 macOS 主机上执行特定配置。以下是来自基于 macOS 的主机的过滤事实的示例：

```
$ ansible frontend01.example.com -m setup -a "filter=ansible_distribution*"
frontend01.example.com | SUCCESS => {
 ansible_facts": {
 "ansible_distribution": "macOS", 
 "ansible_distribution_major_version": "10", 
 "ansible_distribution_release": "18.5.0", 
 "ansible_distribution_version": "10.14.4"
 }, 
 "changed": false
```

临时命令非常强大，既可以用于验证您的 Ansible 安装，也可以用于学习 Ansible 以及如何使用模块，因为您不需要编写整个 playbook-您只需运行一个临时命令并学习它如何响应。以下是一些供您考虑的其他临时示例：

+   使用以下命令将文件从 Ansible 控制主机复制到“前端”组中的所有主机：

```
$ ansible frontends -m copy -a "src=/etc/yum.conf dest=/tmp/yum.conf"
```

+   在“前端”库存组中的所有主机上创建一个新目录，并使用特定的所有权和权限创建它：

```
$ ansible frontends -m file -a "dest=/path/user1/new mode=777 owner=user1 group=user1 state=directory" 
```

+   使用以下命令从“前端”组中的所有主机中删除特定目录：

```
$ ansible frontends -m file -a "dest=/path/user1/new state=absent"
```

+   使用`yum`安装`httpd`软件包，如果尚未安装-如果已安装，则不更新。同样，这适用于“前端”库存组中的所有主机：

```
$ ansible frontends -m yum -a "name=httpd state=present"
```

+   以下命令与上一个命令类似，只是将`state=present`更改为`state=latest`会导致 Ansible 安装（最新版本的）软件包（如果尚未安装），并将其更新到最新版本（如果已安装）：

```
$ ansible frontends -m yum -a "name=demo-tomcat-1 state=latest" 
```

+   显示有关库存中所有主机的所有事实（警告-这将产生大量的 JSON！）：

```
$ ansible all -m setup 
```

现在您已经了解了如何验证您的 Ansible 安装以及如何运行临时命令，让我们继续更详细地查看由 Ansible 管理的节点的要求。

# 受管节点要求

到目前为止，我们几乎完全专注于 Ansible 控制主机的要求，并假设（除了分发 SSH 密钥之外）目标主机将正常工作。当然，并非总是如此，例如，从 ISO 安装的现代 Linux 安装通常会正常工作，云操作系统映像通常会被剥离以保持其小巧，因此可能缺少重要的软件包，如 Python，没有 Python，Ansible 无法运行。

如果您的目标主机缺少 Python，通常可以通过操作系统的软件包管理系统轻松安装它。Ansible 要求您在 Ansible 控制机器（正如我们在本章前面所介绍的）和每个受管节点上安装 Python 版本 2.7 或 3.5（及以上）。再次强调，这里的例外是 Windows，它依赖于 PowerShell。

如果您使用缺少 Python 的操作系统映像，以下命令提供了快速安装 Python 的指南：

+   要使用`yum`安装 Python（在旧版本的 Fedora 和 CentOS/RHEL 7 及以下版本中），请使用以下命令：

```
$ sudo yum -y install python
```

+   在 RHEL 和 CentOS 8 及更新版本的 Fedora 上，您将使用`dnf`软件包管理器：

```
$ sudo dnf install python
```

您也可以选择安装特定版本以满足您的需求，就像这个例子一样：

```
$ sudo dnf install python37
```

+   在 Debian 和 Ubuntu 系统上，您将使用`apt`软件包管理器安装 Python，如果需要的话再指定一个版本（这里给出的示例是安装 Python 3.6，在 Ubuntu 18.04 上可以工作）：

```
$ sudo apt-get update
$ sudo apt-get install python3.6
```

我们在本章前面讨论的 Ansible 的`ping`模块不仅检查与受控主机的连接和身份验证，而且使用受控主机的 Python 环境执行一些基本主机检查。因此，它是一个很棒的端到端测试，可以让您确信您的受控主机已正确配置为主机，具有完美的连接和身份验证设置，但如果缺少 Python，它将返回一个`failed`结果。

当然，在这个阶段一个完美的问题是：如果您使用一个精简的基础镜像在云服务器上部署了 100 个节点，Ansible 如何帮助您？这是否意味着您必须手动检查所有 100 个节点并手动安装 Python 才能开始自动化？

幸运的是，即使在这种情况下，Ansible 也可以帮助您，这要归功于`raw`模块。这个模块用于向受控节点发送原始 shell 命令——它既适用于 SSH 管理的主机，也适用于 Windows PowerShell 管理的主机。因此，您可以使用 Ansible 在缺少 Python 的整套系统上安装 Python，甚至运行一个整个的 shell 脚本来引导一个受控节点。最重要的是，`raw`模块是为数不多的几个不需要在受控节点上安装 Python 的模块之一，因此它非常适合我们的用例，我们必须安装 Python 以启用进一步的自动化。

以下是 Ansible playbook 中的一些任务示例，您可以使用它们来引导受控节点并为其准备好 Ansible 管理：

```
- name: Bootstrap a host without python2 installed
  raw: dnf install -y python2 python2-dnf libselinux-python

- name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
  raw: cat < /tmp/*txt
  args:
    executable: /bin/bash

- name: safely use templated variables. Always use quote filter to avoid injection issues.
  raw: "{{package_mgr|quote}}  {{pkg_flags|quote}}  install  {{python|quote}}"
```

我们现在已经介绍了在控制主机和受控节点上设置 Ansible 的基础知识，并且为您提供了配置第一个连接的简要入门。在结束本章之前，我们将更详细地看一下如何从 GitHub 直接运行最新的 Ansible 开发版本。

# 从源代码运行与预构建的 RPM 包

Ansible 一直在快速发展，可能会有时候，无论是为了早期访问新功能（或模块），还是作为您自己的开发工作的一部分，您希望从 GitHub 运行最新的、最前沿的 Ansible 版本。在本节中，我们将看一下如何快速启动并运行源代码。本章概述的方法有一个优点，即与基于软件包管理器的安装不同，后者必须以 root 身份执行，最终结果是安装了一个可工作的 Ansible，而无需任何 root 权限。

让我们开始从 GitHub 检出最新版本的源代码：

1.  您必须首先从`git`存储库克隆源代码，然后切换到包含已检出代码的目录：

```
$ git clone https://github.com/ansible/ansible.git --recursive
$ cd ./ansible
```

1.  在进行任何开发工作之前，或者确保从源代码运行 Ansible，您必须设置您的 shell 环境。为此提供了几个脚本，每个脚本适用于不同的 shell 环境。例如，如果您使用古老的 Bash shell，您将使用以下命令设置您的环境：

```
$ source ./hacking/env-setup
```

相反，如果您使用 Fish shell，您将设置您的环境如下：

```
**$ source ./hacking/env-setup.fish**
```

1.  设置好环境后，您必须安装`pip` Python 软件包管理器，然后使用它来安装所有所需的 Python 软件包（注意：如果您的系统上已经有`pip`，则可以跳过第一个命令）：

```
$ sudo easy_install pip
$ sudo pip install -r ./requirements.txt
```

请注意，当您运行`env-setup`脚本时，您将从您的源代码检出运行，并且默认的清单文件将是`/etc/ansible/hosts`。您可以选择指定一个除`/etc/ansible/hosts`之外的清单文件。

1.  当您运行`env-setup`脚本时，Ansible 将从源代码检出运行，默认的清单文件是`/etc/ansible/hosts`；但是，您可以选择在您的机器上任何地方指定清单文件（有关更多详细信息，请参见*使用清单*，[`docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory`](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory)）。以下命令提供了一个示例，说明您可能会这样做，但显然，您的文件名和内容几乎肯定会有所不同：

```
$ echo "ap1.example.com" > ~/my_ansible_inventory
$ export ANSIBLE_INVENTORY=~/my_ansible_inventory
```

`ANSIBLE_INVENTORY`适用于 Ansible 版本 1.9 及以上，并替换了已弃用的`ANSIBLE_HOSTS`环境变量。

完成这些步骤后，您可以像本章中讨论的那样运行 Ansible，唯一的例外是您必须指定它的绝对路径。例如，如果您像前面的代码中设置清单并将 Ansible 源克隆到您的主目录中，您可以运行我们现在熟悉的临时`ping`命令，如下所示：

```
$ ~/ansible/bin/ansible all -m ping
ap1.example.com | SUCCESS => {
 "changed": false, 
 "ping": "pong"
}
```

当然，Ansible 源树不断变化，您不太可能只想坚持您克隆的副本。当需要更新时，您不需要克隆新副本；您可以使用以下命令更新现有的工作副本（同样，假设您最初将源树克隆到您的主目录中）：

```
$ git pull --rebase
$ git submodule update --init --recursive
```

这就结束了我们对设置您的 Ansible 控制机和受管节点的介绍。希望您在本章中获得的知识将帮助您启动并为本书的其余部分奠定基础。

# 摘要

Ansible 是一个强大而多才多艺的简单自动化工具，其主要优势是其无代理架构和简单的安装过程。Ansible 旨在让您迅速实现从零到自动化，并以最小的努力，我们已经在本章中展示了您可以如何轻松地开始使用 Ansible。

在本章中，您学习了设置 Ansible 的基础知识——如何安装它来控制其他主机以及被 Ansible 管理的节点的要求。您了解了为 Ansible 自动化设置 SSH 和 WinRM 所需的基础知识，以及如何引导受管节点以确保它们适合于 Ansible 自动化。您还了解了临时命令及其好处。最后，您学会了如何直接从 GitHub 运行最新版本的代码，这既使您能够直接为 Ansible 的开发做出贡献，又使您能够在您的基础设施上使用最新的功能。

在下一章中，我们将学习 Ansible 语言基础知识，以便您编写您的第一个 playbook，并帮助您创建模板化配置，并开始构建复杂的自动化工作流程。

# 问题

1.  您可以在哪些操作系统上安装 Ansible？（多个正确答案）

A）Ubuntu

B）Fedora

C）Windows 2019 服务器

D）HP-UX

E）主机

1.  Ansible 使用哪种协议连接远程机器来运行任务？

A）HTTP

B）HTTPS

C）SSH

D）TCP

E）UDP

1.  要在 Ansible 临时命令行中执行特定模块，您需要使用`-m`选项。

A）正确

B）错误

# 进一步阅读

+   有关通过 Ansible Mailing Liston Google Groups 安装的任何问题，请参阅以下内容：

[`groups.google.com/forum/#!forum/ansible-project`](https://groups.google.com/forum/#!forum/ansible-project)

+   如何安装最新版本的`pip`可以在这里找到：

[`pip.pypa.io/en/stable/installing/#installation`](https://pip.pypa.io/en/stable/installing/#installation)

+   可以在此处找到使用 PowerShell 的特定 Windows 模块：

[`github.com/ansible/ansible-modules-core/tree/devel/windows`](https://github.com/ansible/ansible-modules-core/tree/devel/windows)

+   如果您有 GitHub 账户并想关注 GitHub 项目，您可以继续跟踪 Ansible 的问题、错误和想法：

[`github.com/ansible/ansible`](https://github.com/ansible/ansible)
