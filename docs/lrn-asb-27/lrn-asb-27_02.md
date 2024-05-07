# 使用 Ansible 入门

**信息和通信技术**（**ICT**）通常被描述为一个快速增长的行业。我认为 ICT 行业最好的特质与其能够以超高速度增长无关，而是与其能够以惊人的速度革新自身和世界其他部分的能力有关。

每隔 10 到 15 年，该行业都会发生重大转变，每次转变都会解决之前非常难以管理的问题，从而带来新的挑战。此外，在每次重大转变中，许多先前迭代的最佳实践被归类为反模式，并创建了新的最佳实践。虽然这些变化可能看起来无法预测，但这并不总是正确的。显然，不可能准确地知道将发生什么变化以及何时会发生，但通常观察拥有大量服务器和许多代码行的公司可以揭示下一步的走向。

当前的转变已经在亚马逊网络服务(Amazon Web Services, AWS)、Facebook 和谷歌等大公司中发生。这是实施 IT 自动化系统来创建和管理服务器。

在本章中，我们将涵盖以下主题：

+   IT 自动化

+   什么是 Ansible？

+   安全外壳

+   安装 Ansible

+   使用 Vagrant 创建测试环境

+   版本控制系统

+   使用 Ansible 与 Git

# 技术要求

为了支持学习 Ansible，建议拥有一台可以安装 Vagrant 的机器。使用 Vagrant 将允许您尝试许多操作，甚至是破坏性操作，而不必担心。

此外，建议拥有 AWS 和 Azure 帐户，因为其中一些示例将在这些平台上展示。

本书中的所有示例都可以在 GitHub 仓库中找到：[`github.com/PacktPublishing/-Learning-Ansible-2.X-Third-Edition/`](https://github.com/PacktPublishing/-Learning-Ansible-2.X-Third-Edition/)。

# IT 自动化

**IT 自动化**在更广泛的意义上是指帮助管理 IT 基础设施（服务器、网络和存储）的过程和软件。在当前的转变中，我们正在支持大规模实施这些过程和软件。

在 IT 历史的早期阶段，服务器数量很少，需要很多人来确保它们正常工作，通常每台机器需要多于一个人。随着时间的推移，服务器变得更可靠、更容易管理，因此可以有一个系统管理员管理多个服务器。在那个时期，管理员手动安装软件，手动升级软件，并手动更改配置文件。这显然是一个非常耗时且容易出错的过程，因此许多管理员开始实施脚本和其他手段来简化他们的生活。这些脚本通常非常复杂，而且不太容易扩展。

在本世纪初，由于公司的需求，数据中心开始快速增长。虚拟化有助于降低成本，而且许多这些服务都是 Web 服务，这意味着许多服务器彼此非常相似。此时，需要新的工具来替代以前使用的脚本：配置管理工具。

**CFEngine** 是在上世纪九十年代展示配置管理功能的第一个工具；最近，除了 Ansible 还有 Puppet、Chef 和 Salt。

# IT 自动化的优点

人们常常疑惑 IT 自动化是否真的带来足够的优势，考虑到实施它存在一些直接和间接的成本。IT 自动化的主要好处包括：

+   快速提供机器的能力

+   能够在几分钟内从头开始重建一台机器的能力

+   能够追踪对基础设施进行的任何更改

凭借这些优点，通过减少系统管理员经常执行的重复操作，可以降低管理 IT 基础设施的成本。

# IT 自动化的缺点

与任何其他技术一样，IT 自动化也存在一些缺点。从我的角度来看，这些是最大的缺点：

+   自动化曾经用于培训新的系统管理员的所有小任务。

+   如果发生错误，它将在所有地方传播。

第一个的结果是需要采取新的方法来培训初级系统管理员。

第二个问题更棘手。有很多方法来限制这种损害，但没有一种方法能完全防止。以下是可用的缓解选项：

+   **始终备份**：备份无法防止你毁掉你的机器 - 它们只能使恢复过程成为可能。

+   **始终在非生产环境中测试你的基础设施代码（playbooks/roles）**：公司已经开发了不同的流程来部署代码，通常包括开发、测试、暂存和生产等环境。使用相同的流程来测试您的基础设施代码。如果有一个有错误的应用程序到达生产环境，可能会有问题。如果有一个有错误的 playbook 到达生产环境，情况就可能变得灾难性。

+   **始终对基础设施代码进行同行评审**：一些公司已经引入了对应用代码的同行评审，但很少有公司对基础设施代码进行同行评审。如我之前所说，在我看来，基础设施代码比应用代码更加关键，所以你应该始终对基础设施代码进行同行评审，无论你是否对应用代码进行同行评审。

+   **启用 SELinux**：SELinux 是一个安全内核模块，在所有 Linux 发行版上都可用（默认情况下安装在 Fedora、Red Hat Enterprise Linux、CentOS、Scientific Linux 和 Unbreakable Linux 上）。它允许您以非常细粒度的方式限制用户和进程权限。我建议使用 SELinux 而不是其他类似模块（如 AppArmor），因为它能够处理更多情况和权限。SELinux 将防止大量损坏，因为如果配置正确，它将阻止执行许多危险命令。

+   **以有限权限账户运行 playbook**：尽管用户和特权升级方案在 Unix 代码中已经存在了 40 多年，但似乎并不多有公司使用它们。对于所有你的 playbook 使用有限用户，并仅在需要更高权限的命令时提升权限，这样做将有助于防止在尝试清理应用程序临时文件时意外清除机器。

+   **使用水平特权升级**：`sudo` 命令是众所周知的，但通常以更危险的形式使用。`sudo` 命令支持 `-u` 参数，允许您指定要模拟的用户。如果您必须更改属于另一个用户的文件，请不要升级到 `root`，而是升级到该用户。在 Ansible 中，您可以使用 `become_user` 参数来实现这一点。

+   **尽可能不要同时在所有机器上运行 playbook**：分阶段的部署可以帮助您在为时已晚之前检测到问题。许多问题在开发、测试、暂存和 QA 环境中无法检测到。其中大多数与在这些非生产环境中无法正确模拟的负载相关。您刚刚添加到 Apache HTTPd 或 MySQL 服务器的新配置可能从语法上来说是完全正确的，但对您的生产负载下的特定应用程序来说可能是灾难性的。分阶段的部署将允许您在实际负载下测试您的新配置，而不会在出现问题时造成停机。

+   **避免猜测命令和修改器**：许多系统管理员会尝试记住正确的参数，并在记不清楚时尝试猜测。我也经常这样做，但这是非常危险的。查看手册页或在线文档通常不会花费您两分钟，而且通常通过阅读手册，您会发现您不知道的有趣笔记。猜测修改器是危险的，因为您可能会被非标准的修改器所愚弄（即 `-v` 不是 `grep` 的详细模式，而 `-h` 不是 MySQL CLI 的 `help` 命令）。

+   **避免容易出错的命令**：并非所有命令都是平等创建的。一些命令（远远）比其他命令更危险。如果你可以假设一个基于 `cat` 的命令是安全的，那么你必须假设一个基于 `dd` 的命令是危险的，因为它执行文件和卷的复制和转换。我见过有人在脚本中使用 `dd` 来将 DOS 文件转换为 Unix（而不是 `dos2unix`）等许多其他非常危险的例子。请避免这样的命令，因为如果出了问题，可能会导致巨大的灾难。

+   **避免不必要的修改器**：如果你需要删除一个简单的文件，使用 `rm ${file}`，而不是 `rm -rf ${file}`。后者经常被那些已经学会*确保安全，总是使用* `rm -rf` 的用户执行，因为在他们过去的某个时候，他们曾经需要删除一个文件夹。如果 `${file}` 变量设置错误，这将防止你删除整个文件夹。

+   **始终检查变量未设置时可能发生的情况**：如果你想删除文件夹的内容，并且你使用 `rm -rf ${folder}/*` 命令，那你就是在找麻烦。如果某种原因导致 `${folder}` 变量未设置，shell 将读取一个 `rm -rf /*` 命令，这是致命的（考虑到 `rm -rf /` 命令在大多数当前操作系统上不起作用，因为它需要一个 `--no-preserve-root` 选项，而 `rm -rf /*` 将按预期工作）。我使用这个特定的命令作为示例，因为我见过这样的情况：变量是从一个由于一些维护工作而关闭的数据库中提取出来的，然后给该变量赋了一个空字符串。接下来会发生什么，可能很容易猜到。如果你不能阻止在危险的地方使用变量，至少检查它们是否为空，然后再使用。这不会挽救你免受每一个问题的困扰，但可能会避免一些最常见的问题。

+   **仔细检查你的重定向**：重定向（连同管道）是 Unix shell 中最强大的元素。它们也可能非常危险：一个 `cat /dev/rand > /dev/sda` 可以摧毁一个磁盘，即使一个基于 `cat` 的命令通常被忽视，因为它通常不危险。始终仔细检查包含重定向的所有命令。

+   **尽可能使用特定的模块**：在这个列表中，我使用了 shell 命令，因为很多人会试图将 Ansible 当作一种分发它们的方式：这不是。Ansible 提供了很多模块，我们将在本书中看到它们。它们将帮助你创建更可读、可移植和安全的 playbooks。

# IT 自动化的类型

有很多方法可以对 IT 自动化系统进行分类，但迄今为止最重要的是与配置如何传播有关。基于此，我们可以区分基于代理的系统和无代理的系统。

# 基于代理的系统

基于代理的系统有两个不同的组件：一个**服务器**，和一个称为**代理**的客户端。

只有一个服务器，它包含了整个环境的所有配置，而代理与环境中的机器数量一样多。

在某些情况下，可能会出现多个服务器以确保高可用性，但要将其视为单个服务器，因为它们将以相同的方式配置。

客户端定期会联系服务器，以查看是否存在其机器的新配置。如果有新配置存在，客户端将下载并应用它。

# 无代理系统

在无代理系统中，不存在特定的代理。无代理系统并不总是遵循服务器/客户端范式，因为可能存在多个服务器，甚至可能有与服务器数量相同的服务器和客户端。通信是由服务器初始化的，它将使用标准协议（通常是通过 SSH 和 PowerShell）联系客户端。

# 基于代理与无代理系统

除了先前概述的差异之外，还有其他由于这些差异而产生的对比因素。

从安全性的角度来看，基于代理的系统可能不太安全。由于所有机器都必须能够启动与服务器机器的连接，因此这台机器可能比无代理情况下更容易受到攻击，而后者通常位于不会接受任何传入连接的防火墙后面

从性能的角度来看，基于代理的系统存在使服务器饱和的风险，因此部署可能较慢。还需要考虑到，在纯代理系统中，不可能立即向一组机器推送更新。它将不得不等待这些机器进行检查。因此，多个基于代理的系统已经实现了超出带外方式来实现这些功能。诸如 Chef 和 Puppet 之类的工具是基于代理的，但也可以在没有集中式服务器的情况下运行，以扩展大量的机器，分别称为**无服务器 Chef**和**无主 Puppet**。

无代理系统更容易集成到已有的基础设施（褐地）中，因为客户端会将其视为普通的 SSH 连接，因此不需要额外的配置。

# 什么是 Ansible？

**Ansible**是一种无代理 IT 自动化工具，由前红帽公司的员工 Michael DeHaan 于 2012 年开发。Ansible 的设计目标是尽可能精简、一致、安全、高度可靠和易于学习。Ansible 公司于 2015 年 10 月被红帽公司收购，现在作为红帽公司的一部分运营。

Ansible 主要使用 SSH 以推送模式运行，但您也可以使用 `ansible-pull` 运行 Ansible，在每个代理上安装 Ansible，本地下载 playbooks，并在各个机器上运行它们。如果有大量的机器（大量是一个相对的术语；但在这种情况下，将其视为大于 500 台），并且您计划并行部署更新到机器上，则可能是正确的方法。正如我们之前讨论过的，无论是代理模式还是无代理系统都有其优缺点。

在下一节中，我们将讨论安全外壳（SSH），这是 Ansible 和 Ansible 理念的核心部分。

# 安全外壳

**安全外壳**（也称为 **SSH**）是一种网络服务，允许您通过完全加密的连接远程登录并访问外壳。今天，SSH 守护程序已成为 UNIX 系统管理的标准，取代了未加密的 telnet。SSH 协议的最常用实现是 OpenSSH。

在过去几年中，微软已为 Windows 实现了 OpenSSH。我认为这证明了 SSH 所处的事实标准情况。

由于 Ansible 以与任何其他 SSH 客户端相同的方式执行 SSH 连接和命令，因此未对 OpenSSH 服务器应用特定配置。

要加快默认的 SSH 连接，您可以始终启用 `ControlPersist` 和管道模式，这使得 Ansible 更快速、更安全。

# 为什么选择 Ansible？

我们将在本书的过程中尝试比较 Ansible 与 Puppet 和 Chef，因为许多人对这些工具有很好的经验。我们还将具体指出 Ansible 如何解决与 Chef 或 Puppet 相比的问题。

Ansible、Puppet 和 Chef 都是声明性的，期望将一台机器移动到配置中指定的期望状态。例如，在这些工具中的每一个中，为了在某个时间点启动服务并在重新启动时自动启动，您需要编写一个声明性的块或模块；每次工具在机器上运行时，它都会努力获得您的 **playbook**（Ansible）、**cookbook**（Chef）或 **manifest**（Puppet）中定义的状态。

在简单水平上，工具集之间的差异很小，但随着情况的增多和复杂性的增加，您会开始发现不同工具集之间的差异。在 Puppet 中，您不设置任务执行的顺序，Puppet 服务器会在运行时决定序列和并行执行，这使得最终可能出现难以调试的错误变得更加容易。要利用 Chef 的功能，您需要一个优秀的 Ruby 团队。您的团队需要擅长 Ruby 语言，以定制 Puppet 和 Chef，而且使用这两种工具会有更大的学习曲线。

与 Ansible 不同的是。它在执行顺序上使用了 Chef 的简单性 - 自上而下的方法 - 并允许你以 YAML 格式定义最终状态，这使得代码极易阅读和易于理解，无论是从开发团队到运维团队，都能轻松掌握并进行更改。在许多情况下，即使没有 Ansible，运维团队也会被给予 playbook 手册，以便在遇到问题时执行指令。Ansible 模仿了那种行为。如果由于其简单性而最终使你的项目经理更改 Ansible 代码并将其提交到 Git，也不要感到惊讶！

# 安装 Ansible

安装 Ansible 相当快速简单。你可以直接使用源代码，从 GitHub 项目 ([`github.com/ansible/ansible`](https://github.com/ansible/ansible)) 克隆；使用系统的软件包管理器进行安装；或者使用 Python 的软件包管理工具 (**pip**)。你可以在任何 Windows 或类 Unix 系统上使用 Ansible，比如 macOS 和 Linux。Ansible 不需要任何数据库，也不需要运行任何守护程序。这使得维护 Ansible 版本和升级变得更容易，而且没有任何中断。

我们想要称呼我们将要安装 Ansible 的机器为 Ansible 工作站。有些人也将其称为指挥中心。

# 使用系统的软件包管理器安装 Ansible

可以使用系统的软件包管理器安装 Ansible，就我个人而言，如果你的系统软件包管理器至少提供了 Ansible 2.0，这是首选选项。我们将研究通过 **Yum**、**Apt**、**Homebrew** 和 **pip** 安装 Ansible。

# 通过 Yum 安装

如果你正在运行 Fedora 系统，你可以直接安装 Ansible，因为从 Fedora 22 开始，Ansible 2.0+ 可以在官方仓库中找到。你可以按照以下步骤安装它：

```
    $ sudo dnf install ansible
```

对于 RHEL 和基于 RHEL 的系统（CentOS、Scientific Linux 和 Unbreakable Linux），版本 6 和 7 在 EPEL 仓库中有 Ansible 2.0+，因此在安装 Ansible 之前，你应该确保已启用 EPEL 仓库，步骤如下：

```
    $ sudo yum install ansible
```

在 RHEL 6 上，你必须运行 `$ sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm` 命令来安装 EPEL，而在 RHEL 7 上，`$ sudo yum install epel-release` 就足够了。

# 通过 Apt 安装

Ansible 可用于 Ubuntu 和 Debian。要在这些操作系统上安装 Ansible，请使用以下命令：

```
    $ sudo apt-get install ansible
```

# 通过 Homebrew 安装

你可以使用 Homebrew 在 Mac OS X 上安装 Ansible，步骤如下：

```
    $ brew update
    $ brew install ansible
```

# 通过 pip 安装

你可以通过 pip 安装 Ansible。如果你的系统上没有安装 pip，先安装它。你也可以在 Windows 上使用以下命令行使用 pip 安装 Ansible：

```
    $ sudo easy_install pip
```

现在你可以使用 `pip` 安装 Ansible，步骤如下：

```
    $ sudo pip install ansible
```

安装完 Ansible 后，运行 `ansible --version` 来验证是否已经安装：

```
    $ ansible --version
```

从上述命令行输出中，你将得到许多行，如下所示：

```
ansible 2.7.1
 config file = /etc/ansible/ansible.cfg
 configured module search path = [u'/home/fale/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/site-packages/ansible
 executable location = /bin/ansible
 python version = 2.7.15 (default, Oct 15 2018, 15:24:06) [GCC 8.1.1 20180712 (Red Hat 8.1.1-5)]
```

# 从源代码安装 Ansible

如果前面的方法不适合您的用例，您可以直接从源代码安装 Ansible。从源代码安装不需要任何 root 权限。让我们克隆一个存储库并激活 `virtualenv`，它是 Python 中的一个隔离环境，您可以在其中安装包而不会干扰系统的 Python 包。存储库的命令和结果输出如下：

```
    $ git clone git://github.com/ansible/ansible.git
    Cloning into 'ansible'...
    remote: Counting objects: 116403, done.
    remote: Compressing objects: 100% (18/18), done.
    remote: Total 116403 (delta 3), reused 0 (delta 0), pack-reused 116384
    Receiving objects: 100% (116403/116403), 40.80 MiB | 844.00 KiB/s, done.
    Resolving deltas: 100% (69450/69450), done.
    Checking connectivity... done.
    $ cd ansible/
    $ source ./hacking/env-setup
    Setting up Ansible to run out of checkout...
    PATH=/home/vagrant/ansible/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/vagrant/bin
    PYTHONPATH=/home/vagrant/ansible/lib:
    MANPATH=/home/vagrant/ansible/docs/man:
    Remember, you may wish to specify your host file with -i
    Done!
```

Ansible 需要安装一些 Python 包，您可以使用 `pip` 安装。如果您的系统没有安装 pip，请使用以下命令进行安装。如果您没有安装 `easy_install`，您可以在 Red Hat 系统上使用 Python 的 `setuptools` 包安装它，或者在 macOS 上使用 Brew 安装：

```
    $ sudo easy_install pip
    <A long output follows>
```

安装了 `pip` 后，使用以下命令行安装 `paramiko`、`PyYAML`、`jinja2` 和 `httplib2` 包：

```
    $ sudo pip install paramiko PyYAML jinja2 httplib2
    Requirement already satisfied (use --upgrade to upgrade): paramiko in /usr/lib/python2.6/site-packages
    Requirement already satisfied (use --upgrade to upgrade): PyYAML in /usr/lib64/python2.6/site-packages
    Requirement already satisfied (use --upgrade to upgrade): jinja2 in /usr/lib/python2.6/site-packages
    Requirement already satisfied (use --upgrade to upgrade): httplib2 in /usr/lib/python2.6/site-packages
    Downloading/unpacking markupsafe (from jinja2)
      Downloading MarkupSafe-0.23.tar.gz
      Running setup.py (path:/tmp/pip_build_root/markupsafe/setup.py) egg_info for package markupsafe
    Installing collected packages: markupsafe
      Running setup.py install for markupsafe
        building 'markupsafe._speedups' extension
        gcc -pthread -fno-strict-aliasing -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -DNDEBUG -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic -D_GNU_SOURCE -fPIC -fwrapv -fPIC -I/usr/include/python2.6 -c markupsafe/_speedups.c -o build/temp.linux-x86_64-2.6/markupsafe/_speedups.o
        gcc -pthread -shared build/temp.linux-x86_64-2.6/markupsafe/_speedups.o -L/usr/lib64 -lpython2.6 -o build/lib.linux-x86_64-2.6/markupsafe/_speedups.so
    Successfully installed markupsafe
    Cleaning up...
```

默认情况下，Ansible 将运行在开发分支上。您可能想要切换到最新的稳定分支。使用以下 `$ git branch -a` 命令来检查最新的稳定版本。

复制您想要使用的最新版本。

在撰写时，版本 2.0.2 是最新版本。使用以下命令行检查最新版本：

```
    [node ansible]$ git checkout v2.7.1
    Note: checking out 'v2.0.2'.
    [node ansible]$ ansible --version
    ansible 2.7.1 (v2.7.1 c963ef1dfb) last updated 2018/10/25 20:12:52 (GMT +000)
```

您现在已经准备好了 Ansible 的工作设置。从源代码运行 Ansible 的一个好处是您可以立即享受到新功能，而不必等待软件包管理器为您提供它们。

# 使用 Vagrant 创建测试环境

为了学习 Ansible，我们需要制作相当多的 playbooks 并运行它们。

直接在您的计算机上执行此操作将非常危险。因此，我建议使用虚拟机。

可以在几秒钟内使用云提供商创建测试环境，但通常更有用的是在本地拥有这些机器。为此，我们将使用 Vagrant，这是 Hashicorp 公司提供的一款软件，允许用户独立快速地设置虚拟环境，与本地系统上使用的虚拟化后端无关。它支持许多虚拟化后端（在 Vagrant 生态系统中被称为 *Providers*），例如 Hyper-V、VirtualBox、Docker、VMWare 和 libvirt。这使您可以在任何操作系统或环境中使用相同的语法。

首先，我们将安装 `vagrant`。在 Fedora 上，运行以下代码就足够了：

```
    $ sudo dnf install -y vagrant  
```

在 Red Hat/CentOS/Scientific Linux/Unbreakable Linux 上，我们需要先安装 `libvirt`，然后启用它，然后从 Hashicorp 网站安装 `vagrant`：

```
$ sudo yum install -y qemu-kvm libvirt virt-install bridge-utils libvirt-devel libxslt-devel libxml2-devel libvirt-devel libguestfs-tools-c
$ sudo systemctl enable libvirtd
$ sudo systemctl start libvirtd
$ sudo rpm -Uvh https://releases.hashicorp.com/vagrant/2.2.1/vagrant_2.2.1_x86_64.rpm
$ vagrant plugin install vagrant-libvirt
```

如果您使用的是 Ubuntu 或 Debian，您可以使用以下代码进行安装：

```
    $ sudo apt install virtualbox vagrant
```

对于以下示例，我将虚拟化 CentOS 7 机器。这是出于多种原因；主要原因如下：

+   CentOS 是免费的，与 Red Hat、Scientific Linux 和 Unbreakable Linux 完全兼容。

+   许多公司将 Red Hat/CentOS/Scientific Linux/Unbreakable Linux 用于其服务器。

+   这些发行版是唯一内置 SELinux 支持的发行版，正如我们之前所见，SELinux 可以帮助你使环境更加安全。

为了测试一切是否顺利，我们可以运行以下命令：

```
$ sudo vagrant init centos/7 && sudo vagrant up
```

如果一切顺利，你应该期待一个以这样结尾的输出：

```
==> default: Configuring and enabling network interfaces...
default: SSH address: 192.168.121.60:22
default: SSH username: vagrant
default: SSH auth method: private key
==> default: Rsyncing folder: /tmp/ch01/ => /vagrant
```

所以，你现在可以执行`vagrant ssh`，你会发现自己在刚刚创建的机器中。

当前文件夹中将有一个`Vagrant`文件。在这个文件中，你可以使用`vagrant init`创建指令来创建虚拟环境。

# 版本控制系统

在本章中，我们已经遇到了表达式**基础设施代码**来描述将创建和维护您的基础设施的 Ansible 代码。我们使用基础设施代码这个表达式来区分它与应用代码，后者是组成您的应用程序、网站等的代码。这种区别是为了清晰起见，但最终，这两种类型都是软件能够读取和解释的一堆文本文件。

因此，版本控制系统将会给你带来很多帮助。它的主要优点如下：

+   多人同时在同一项目上工作的能力。

+   进行简单方式的代码审查的能力。

+   拥有多个分支用于多个环境（即开发、测试、QA、暂存和生产）的能力。

+   能够追踪更改，以便我们知道更改是何时引入的，以及是谁引入的。这样一来，几个月或几年后，就更容易理解为什么那段代码存在。

这些优点是由现有的大多数版本控制系统提供给你的。

版本控制系统可以根据它们实现的三种不同模型分为三大类：

+   本地数据模型

+   客户端-服务器模型

+   分布式模型

第一类别，即本地数据模型，是最古老的（大约在 1972 年左右）方法，用于非常特定的用例。这种模型要求所有用户共享相同的文件系统。它的著名示例有**Revision Control System**（**RCS**）和**Source Code Control System**（**SCCS**）。

第二类别，客户端-服务器模型，后来（大约在 1990 年左右）出现，并试图解决本地数据模型的限制，创建了一个遵循本地数据模型的服务器和一组客户端，这些客户端与服务器而不是与存储库本身打交道。这个额外的层允许多个开发人员使用本地文件并将它们与一个集中式服务器同步。这种方法的著名示例是 Apache **Subversion**（**SVN**）和**Concurrent Versions System**（**CVS**）。

第三类别，即分布式模型，于二十一世纪初出现，并试图解决客户端-服务器模型的限制。事实上，在客户端-服务器模型中，您可以脱机工作，但需要在 *在线* 提交更改。分布式模型允许您在本地存储库上处理所有事务（如本地数据模型），并以轻松的方式合并不同机器上的不同存储库。在这种新模型中，可以执行与客户端-服务器模型中的所有操作相同的操作，而且还能够完全脱机工作，以及在同行之间合并更改而不必通过集中式服务器。这种模型的示例包括 BitKeeper（专有软件）、Git、GNU Bazaar 和 Mercurial。

只有分布式模型才能提供的一些额外优势，例如以下内容：

+   即使服务器不可用也可以进行提交、浏览历史记录以及执行任何其他操作的可能性。

+   更容易管理不同环境的多个分支。

当涉及基础设施代码时，我们必须考虑到管理您的基础设施代码的基础设施本身经常保存在基础设施代码中。这是一个递归的情况，可能会引发问题。分布式版本控制系统将防止此问题发生。

关于管理多个分支的简易性，虽然这不是一个硬性规则，但通常分布式版本控制系统比其他类型的版本控制系统具有更好的合并处理能力。

# 使用 Ansible 与 Git

出于我们刚刚看到的原因，以及由于其巨大的流行度，我建议始终使用 Git 作为您的 Ansible 存储库。

我总是向我交谈的人提供一些建议，以便 Ansible 充分利用 Git：

+   **创建环境分支**：创建环境分支，例如开发、生产、测试和预发布，将使您能够轻松跟踪不同环境及其各自的更新状态。我经常建议将主分支保留给开发环境，因为我发现很多人习惯直接向主分支推送新更改。如果您将主分支用于生产环境，人们可能会无意中将更改推送到生产环境，而他们本想将其推送到开发环境。

+   **始终保持环境分支稳定**：拥有环境分支的一个重要优势是可以在任何时刻从头开始销毁和重建任何环境。只有在环境分支处于稳定（非破损）状态时才能实现这一点。

+   **使用功能分支**：为特定的长期开发功能（如重构或其他大的更改）使用不同的分支，这样您就可以在 Git 存储库中保持日常运营，而您的新功能正在进行中（这样您就不会失去对谁做了什么以及何时做了什么的追踪）。

+   **经常推送**：我总是建议人们尽可能经常*推送提交*。这将使 Git 成为版本控制系统和备份系统。我经常看到笔记本电脑损坏、丢失或被盗，其中有数天或数周的未推送工作。不要浪费你的时间——经常推送。而且，经常推送还会更早地检测到合并冲突，合并冲突总是在早期检测到时更容易处理，而不是等待多个更改。

+   **在进行更改后始终部署**：我见过开发人员在基础架构代码中进行更改后，在开发和测试环境中进行了测试，推送到生产分支，然后去吃午饭的情况。他的午餐并不愉快。他的一位同事无意中将代码部署到生产环境（他当时试图部署他所做的小改动），而且没有准备好处理其他开发人员的部署。生产基础架构崩溃了，他们花了很多时间弄清楚一个小小的改动（部署者知道的那个）怎么可能造成如此大的混乱。

+   **选择多个小的更改而不是几个大的更改**：尽可能进行小的更改将使调试更容易。调试基础架构并不容易。没有编译器可以让您看到“明显的问题”（即使 Ansible 执行您的代码的语法检查，也不会执行其他测试），而且查找故障的工具并不总是像您想象的那样好。基础架构即代码范例是新的，工具还不像应用程序代码的工具那样好。

+   **尽量避免二进制文件**：我总是建议将二进制文件保存在 Git 存储库之外，无论是应用程序代码存储库还是基础架构代码存储库。在应用程序代码示例中，我认为保持存储库轻量化很重要（Git 以及大多数版本控制系统对二进制大对象的性能表现不佳），而在基础架构代码示例中，这是至关重要的，因为你会受到诱惑，想要在其中放入大量二进制对象，因为往往将二进制对象放入存储库比找到更干净（和更好）的解决方案更容易。

# 总结

在这一章中，我们已经了解了什么是 IT 自动化，它的优缺点，你可以找到什么样的工具，以及 Ansible 如何融入这个大局。我们还看到了如何安装 Ansible 以及如何创建一个 Vagrant 虚拟机。最后，我们分析了版本控制系统，并谈到了 Git 如何在正确使用时为 Ansible 带来的优势。

在下一章中，我们将开始看到我们在本章中提到的基础架构代码，而不是详细解释它是什么以及如何编写它。我们还将看到如何自动化那些你可能每天都要进行的简单操作，比如管理用户，管理文件和文件内容。
