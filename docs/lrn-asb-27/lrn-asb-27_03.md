# 自动化简单任务

如前一章所述，Ansible 可用于创建和管理整个基础架构，也可集成到已经运行的基础架构中。

在本章中，我们将涵盖以下主题：

+   YAML

+   使用 Playbook

+   Ansible 速度

+   Playbook 中的变量

+   创建 Ansible 用户

+   配置基本服务器

+   安装和配置 Web 服务器

+   发布网站

+   Jinja2 模板

首先，我们将讨论**YAML Ain't Markup Language**（**YAML**），这是一种人类可读的数据序列化语言，广泛用于 Ansible。

# 技术要求

您可以从本书的 GitHub 存储库下载所有文件：[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter02`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter02)。

# YAML

YAML，像许多其他数据序列化语言（如 JSON）一样，有着极少的基本概念：

+   声明

+   列表

+   关联数组

声明与任何其他语言中的变量非常相似，如下所示：

```
name: 'This is the name'
```

要创建列表，我们将使用`-`：

```
- 'item1' 
- 'item2' 
- 'item3' 
```

YAML 使用缩进来在逻辑上将父项与子项分隔开来。因此，如果我们想创建关联数组（也称为对象），我们只需要添加一个缩进：

```
item: 
  name: TheName 
  location: TheLocation 
```

显然，我们可以将它们混合在一起，如下所示：

```
people:
  - name: Albert 
    number: +1000000000 
    country: USA 
  - name: David 
    number: +44000000000 
    country: UK 
```

这些是 YAML 的基础知识。YAML 可以做得更多，但目前这些就足够了。

# 你好 Ansible

正如我们在前一章中看到的，可以使用 Ansible 自动化您可能每天已经执行的简单任务。

让我们从检查远程机器是否可达开始；换句话说，让我们从对机器进行 ping 开始。这样做的最简单方法是运行以下命令：

```
$ ansible all -i HOST, -m ping 
```

在这里，`HOST`是您拥有 SSH 访问权限的机器的 IP 地址、**Fully Qualified Domain Name**（**FQDN**）或别名（您可以使用像我们在上一章中看到的 **Vagrant** 主机）。

在`HOST`之后，逗号是必需的，因为否则，它不会被视为列表，而是视为字符串。

在这种情况下，我们执行了针对我们系统上的虚拟机：

```
$ ansible all -i test01.fale.io, -m ping 
```

你应该收到类似这样的结果：

```
test01.fale.io | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

现在，让我们看看我们做了什么以及为什么。让我们从 Ansible 帮助开始。要查询它，我们可以使用以下命令：

```
$ ansible --help 
```

为了更容易阅读，我们已删除了与我们未使用的选项相关的所有输出：

```
Usage: ansible <host-pattern> [options]

Options:
  -i INVENTORY, --inventory=INVENTORY, --inventory-file=INVENTORY
                        specify inventory host path or comma separated host
                        list. --inventory-file is deprecated
  -m MODULE_NAME, --module-name=MODULE_NAME
                        module name to execute (default=command)
```

所以，我们所做的是：

1.  我们调用了 Ansible。

1.  我们指示 Ansible 在所有主机上运行。

1.  我们指定了我们的清单（也称为主机列表）。

1.  我们指定了要运行的模块（`ping`）。

现在我们可以 ping 服务器，让我们尝试`echo hello ansible!`，如下命令所示：

```
$ ansible all -i test01.fale.io, -m shell -a '/bin/echo hello ansible!' 
```

你应该收到类似这样的结果：

```
test01.fale.io | CHANGED | rc=0 >>
hello ansible!
```

在此示例中，我们使用了一个额外的选项。让我们检查 Ansible 帮助以查看它的作用：

```
Usage: ansible <host-pattern> [options]
Options:
  -a MODULE_ARGS, --args=MODULE_ARGS
                        module arguments
```

从上下文和名称可以猜到，`args` 选项允许你向模块传递额外的参数。某些模块（如 `ping`）不支持任何参数，而其他模块（如 `shell`）将需要参数。

# 使用 playbooks

**Playbooks** 是 Ansible 的核心特性之一，告诉 Ansible 要执行什么。它们就像 Ansible 的待办事项列表，包含一系列任务；每个任务内部链接到一个称为 **模块** 的代码片段。Playbooks 是简单易读的 YAML 文件，而模块是可以用任何语言编写的代码片段，条件是其输出格式为 JSON。你可以在一个 playbook 中列出多个任务，这些任务将由 Ansible 串行执行。你可以将 playbooks 视为 Puppet 中的清单、Salt 中的状态或 Chef 中的菜谱的等价物；它们允许你输入你想在远程系统上执行的任务或命令列表。

# 研究 playbook 的结构

Playbooks 可以具有远程主机列表、用户变量、任务、处理程序等。你还可以通过 playbook 覆盖大部分配置设置。让我们开始研究 playbook 的结构。

我们现在要考虑的 playbook 的目的是确保 `httpd` 包已安装并且服务已 **启用** 和 **启动**。这是 `setup_apache.yaml` 文件的内容：

```

- hosts: all 
  remote_user: vagrant
  tasks: 
    - name: Ensure the HTTPd package is installed 
      yum: 
        name: httpd 
        state: present 
      become: True 
    - name: Ensure the HTTPd service is enabled and running 
      service: 
        name: httpd 
        state: started 
        enabled: True 
      become: True
```

`setup_apache.yaml` 文件是一个 playbook 的示例。文件由三个主要部分组成，如下所示：

+   `hosts`: 这列出了我们要针对哪个主机或主机组运行任务。hosts 字段是必需的。Ansible 使用它来确定哪些主机将成为列出任务的目标。如果提供的是主机组而不是主机，则 Ansible 将尝试根据清单文件查找属于它的主机。如果没有匹配项，Ansible 将跳过该主机组的所有任务。`--list-hosts` 选项以及 playbook (`ansible-playbook <playbook> --list-hosts`) 将告诉你准确地 playbook 将运行在哪些主机上。

+   `remote_user`: 这是 Ansible 的配置参数之一（例如，`tom' - remote_user`），告诉 Ansible 在登录系统时使用特定用户（在本例中为 `tom`）。

+   `tasks`: 最后，我们来到了任务。所有 playbook 应该包含任务。任务是你想执行的一系列操作。一个 `tasks` 字段包含了任务的名称（即，用户关于任务的帮助文本）、应该执行的模块以及模块所需的参数。让我们看一下在 playbook 中列出的单个任务，如前面代码片段所示。

本书中的所有示例将在 CentOS 上执行，但是对同一组示例进行少量更改后也可以在其他发行版上运行。

在上述情况下，有两个任务。`name`参数表示任务正在做什么，并且主要是为了提高可读性，正如我们在 playbook 运行期间将看到的那样。`name`参数是可选的。`yum`和`service`模块有自己的一组参数。几乎所有模块都有`name`参数（有例外，比如`debug`模块），它表示对哪个组件执行操作。让我们看看其他参数：

+   `state`参数在`yum`模块中保存了最新的值，它表示应该安装`httpd`软件包。执行的命令大致相当于`yum install httpd`。

+   在`service`模块的场景中，带有`started`值的`state`参数表示`httpd`服务应该启动，它大致相当于`/etc/init.d/httpd`启动。在此模块中，我们还有`enabled`参数，它定义了服务是否应该在启动时启动。

+   `become: True`参数表示任务应该以`sudo`访问权限执行。如果`sudo`用户的文件不允许用户运行特定命令，那么当运行 playbook 时，playbook 将失败。

你可能会问，为什么没有一个包模块能够在内部确定架构并根据系统的架构运行`yum`、`apt`或任何其他包选项。Ansible 将包管理器的值填充到一个名为`ansible_pkg_manager`的变量中。

一般来说，我们需要记住，在不同操作系统中具有通用名称的软件包的数量是实际存在的软件包数量的一个很小的子集。例如，`httpd`软件包在 Red Hat 系统中称为`httpd`，但在基于 Debian 的系统中称为`apache2`。我们还需要记住，每个包管理器都有自己的一组选项，使其功能强大；因此，使用明确的包管理器名称更合理，这样终端用户编写 playbook 时就可以使用完整的选项集。

# 运行 playbook

现在，是时候（是的，终于！）运行 playbook 了。为了指示 Ansible 执行 playbook 而不是模块，我们将不得不使用一个语法非常类似于我们已经看到的`ansible`命令的不同命令（`ansible-playbooks`）：

```
$ ansible-playbook -i HOST, setup_apache.yaml
```

如您所见，除了在 playbook 中指定的主机模式（已消失）和模块选项（已被 playbook 名称替换）之外，没有任何变化。因此，要在我的机器上执行此命令，确切的命令如下：

```
$ ansible-playbook -i test01.fale.io, setup_apache.yaml 
```

结果如下：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [test01.fale.io]

TASK [Ensure the HTTPd package is installed] *************************
changed: [test01.fale.io]

TASK [Ensure the HTTPd service is enabled and running] ***************
changed: [test01.fale.io]

PLAY RECAP ***********************************************************
test01.fale.io                : ok=3 changed=2 unreachable=0 failed=0
```

哇！示例运行成功。现在让我们检查一下`httpd`软件包是否已安装并且现在正在机器上运行。要检查 HTTPd 是否已安装，最简单的方法是询问`rpm`：

```
$ rpm -qa | grep httpd 
```

如果一切正常工作，你应该有如下输出：

```
httpd-tools-2.4.6-80.el7.centos.1.x86_64
httpd-2.4.6-80.el7.centos.1.x86_64
```

要查看服务的状态，我们可以询问`systemd`：

```
$ systemctl status httpd
```

预期结果如下所示：

```
httpd.service - The Apache HTTP Server
 Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
 Active: active (running) since Tue 2018-12-04 15:11:03 UTC; 29min ago
 Docs: man:httpd(8)
 man:apachectl(8)
 Main PID: 604 (httpd)
 Status: "Total requests: 0; Current requests/sec: 0; Current traffic: 0 B/sec"
 CGroup: /system.slice/httpd.service
 ├─604 /usr/sbin/httpd -DFOREGROUND
 ├─624 /usr/sbin/httpd -DFOREGROUND
 ├─626 /usr/sbin/httpd -DFOREGROUND
 ├─627 /usr/sbin/httpd -DFOREGROUND
 ├─628 /usr/sbin/httpd -DFOREGROUND
 └─629 /usr/sbin/httpd -DFOREGROUND 
```

根据 playbook，已达到最终状态。让我们简要地看一下 playbook 运行过程中确切发生了什么：

```
PLAY [all] ***********************************************************
```

此行建议我们将从这里开始执行 playbook，并且将在所有主机上执行：

```
TASK [Gathering Facts] ***********************************************
ok: [test01.fale.io]
```

`TASK` 行显示任务的名称（在本例中为`setup`）以及其对每个主机的影响。有时，人们会对`setup`任务感到困惑。实际上，如果您查看 playbook，您会发现没有`setup`任务。这是因为在执行我们要求的任务之前，Ansible 会尝试连接到机器并收集有关以后可能有用的信息。正如您所看到的，该任务结果显示为绿色的`ok`状态，因此成功了，并且服务器上没有发生任何更改：

```
TASK [Ensure the HTTPd package is installed] *************************
changed: [test01.fale.io]

TASK [Ensure the HTTPd service is enabled and running] ***************
changed: [test01.fale.io]
```

这两个任务的状态是黄色的，并拼写为`changed`。这意味着这些任务已执行并成功，但实际上已更改了机器上的某些内容：

```
PLAY RECAP ***********************************************************
test01.fale.io : ok=3 changed=2 unreachable=0 failed=0
```

最后几行是 playbook 执行情况的总结。现在让我们重新运行任务，然后查看两个任务实际运行后的输出：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [test01.fale.io]

TASK [Ensure the HTTPd package is installed] *************************
ok: [test01.fale.io]

TASK [Ensure the HTTPd service is enabled and running] ***************
ok: [test01.fale.io]

PLAY RECAP ***********************************************************
test01.fale.io                : ok=3 changed=0 unreachable=0 failed=0
```

正如您所预期的那样，所涉及的两个任务的输出为`ok`，这意味着在运行任务之前已满足了所需状态。重要的是要记住，许多任务（例如**收集事实**任务）会获取有关系统特定组件的信息，并不一定会更改系统中的任何内容；因此，这些任务之前没有显示更改的输出。

在第一次和第二次运行时，`PLAY RECAP`部分显示如下。在第一次运行时，您将看到以下输出：

```
PLAY RECAP ***********************************************************
test01.fale.io                : ok=3 changed=2 unreachable=0 failed=0
```

第二次运行时，您将看到以下输出：

```
PLAY RECAP ***********************************************************
test01.fale.io                : ok=3 changed=0 unreachable=0 failed=0
```

如您所见，区别在于第一个任务的输出显示`changed=2`，这意味着由于两个任务而更改了系统状态两次。查看此输出非常有用，因为如果系统已达到其所需状态，然后您在其上运行 playbook，则预期输出应为`changed=0`。

如果您在这个阶段考虑了**幂等性**这个词，那么您是完全正确的，并且值得表扬！幂等性是配置管理的关键原则之一。维基百科将幂等性定义为一个操作，如果对任何值应用两次，则其结果与仅应用一次时相同。您在童年时期遇到的最早的例子是对数字`1`进行乘法运算，其中`1*1=1`每次都成立。

大多数配置管理工具都采用了这个原则，并将其应用于基础架构。在大型基础架构中，强烈建议监视或跟踪基础架构中更改任务的数量，并在发现异常时警告相关任务；这通常适用于任何配置管理工具。在理想状态下，你只应该在引入新的更改时看到更改，比如对各种系统组件进行**创建**、**删除**、**更新**或**删除**（**CRUD**）操作。如果你想知道如何在 Ansible 中实现它，继续阅读本书，你最终会找到答案的！

让我们继续。你也可以将前面的任务写成如下形式，但是从最终用户的角度来看，任务非常易读（我们将此文件称为`setup_apache_no_com.yaml`）：

```
--- 
- hosts: all 
  remote_user: vagrant
  tasks: 
    - yum: 
        name: httpd 
        state: present 
      become: True 
    - service: 
        name: httpd 
        state: started 
        enabled: True 
      become: True
```

让我们再次运行 playbook，以发现输出中的任何差异：

```
$ ansible-playbook -i test01.fale.io, setup_apache_no_com.yaml
```

输出将如下所示：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [test01.fale.io]

TASK [yum] ***********************************************************
ok: [test01.fale.io]

TASK [service] *******************************************************
ok: [test01.fale.io]

PLAY RECAP ***********************************************************
test01.fale.io                : ok=3 changed=0 unreachable=0 failed=0
```

如你所见，区别在于可读性。在可能的情况下，建议尽可能简化任务（**KISS**原则：**保持简单，笨拙**），以确保长期保持脚本的可维护性。

现在我们已经看到了如何编写一个基本的 playbook 并对其运行到主机，让我们看看在运行 playbooks 时会帮助你的其他选项。

# Ansible 详细程度

任何人首先选择的选项之一是调试选项。为了了解在运行 playbook 时发生了什么，你可以使用详细（`-v`）选项运行它。每个额外的`v`将为最终用户提供更多的调试输出。

让我们看一个使用这些选项调试简单`ping`命令（`ansible all -i test01.fale.io, -m ping`）的示例：

+   `-v`选项提供了默认输出：

```
Using /etc/ansible/ansible.cfg as config file
test01.fale.io | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

+   `-vv`选项会提供有关 Ansible 环境和处理程序的更多信息：

```
ansible 2.7.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/fale/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.15 (default, Oct 15 2018, 15:24:06) [GCC 8.1.1 20180712 (Red Hat 8.1.1-5)]
Using /etc/ansible/ansible.cfg as config file
META: ran handlers
test01.fale.io | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
META: ran handlers
META: ran handlers
```

+   `-vvv`选项提供了更多信息。例如，它显示 Ansible 用于在远程主机上创建临时文件并在远程运行脚本的`ssh`命令。完整脚本可在 GitHub 上找到。

```
ansible 2.7.2
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/fale/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /bin/ansible
  python version = 2.7.15 (default, Oct 15 2018, 15:24:06) [GCC 8.1.1 20180712 (Red Hat 8.1.1-5)]
Using /etc/ansible/ansible.cfg as config file
Parsed test01.fale.io, inventory source with host_list plugin
META: ran handlers
<test01.fale.io> ESTABLISH SSH CONNECTION FOR USER: None
<test01.fale.io> SSH: EXEC ssh -C -o ControlMaster=auto -o 
...
```

现在我们了解了在运行 playbook 时发生了什么，使用详细`-vvv`选项。

# playbook 中的变量

有时，在 playbook 中设置和获取变量是很重要的。

很多时候，你会需要自动化多个类似的操作。在这些情况下，你将想要创建一个可以使用不同变量调用的单个 playbook，以确保代码的可重用性。

另一个情况下变量非常重要的案例是当你有多个数据中心时，一些值将是特定于数据中心的。一个常见的例子是 DNS 服务器。让我们分析下面的简单代码，这将向我们介绍设置和获取变量的 Ansible 方法：

```
- hosts: all 
  remote_user: vagrant
  tasks: 
    - name: Set variable 'name' 
      set_fact: 
        name: Test machine 
    - name: Print variable 'name' 
      debug: 
        msg: '{{ name }}' 
```

让我们以通常的方式运行它：

```
$ ansible-playbook -i test01.fale.io, variables.yaml
```

你应该看到以下结果：

```
PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [test01.fale.io]

TASK [Set variable 'name'] *****************************************
ok: [test01.fale.io]

TASK [Print variable 'name'] ***************************************
ok: [test01.fale.io] => {
 "msg": "Test machine"
}

PLAY RECAP *********************************************************
test01.fale.io              : ok=3 changed=0 unreachable=0 failed=0 
```

如果我们分析刚刚执行的代码，应该很清楚发生了什么。我们设置了一个变量（在 Ansible 中称为 `facts`），然后用 `debug` 函数打印它。

当你使用这个扩展版本的 YAML 时，变量应该总是用引号括起来。

Ansible 允许你以许多不同的方式设置变量 - 也就是说，通过传递一个变量文件，在 playbook 中声明它，使用 `-e / --extra-vars` 参数将它传递给 `ansible-playbook` 命令，或者在清单文件中声明它（我们将在下一章中深入讨论这一点）。

现在是时候开始使用 Ansible 在设置阶段获取的一些元数据了。让我们开始查看 Ansible 收集的数据。为此，我们将执行以下代码：

```
$ ansible all -i HOST, -m setup 
```

在我们特定的情况下，这意味着执行以下代码：

```
$ ansible all -i test01.fale.io, -m setup 
```

显然我们也可以用 playbook 来做同样的事情，但这种方式更快。另外，对于 `setup` 情况，你只需要在开发过程中看到输出以确保使用正确的变量名称来达到你的目标。

输出将会是类似这样的。完整的代码输出可在 GitHub 上找到。

```
test01.fale.io | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.121.190"
        ], 
        "ansible_all_ipv6_addresses": [
            "fe80::5054:ff:fe93:f113"
        ], 
        "ansible_apparmor": {
            "status": "disabled"
        }, 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "04/01/2014", 
        "ansible_bios_version": "?-20180531_142017-buildhw-08.phx2.fedoraproject.org-1.fc28", 
        ...
```

正如你从这个大量选项的列表中所看到的，你可以获得大量的信息，并且你可以像使用任何其他变量一样使用它们。让我们打印操作系统的名称和版本。为此，我们可以创建一个名为 `setup_variables.yaml` 的新 playbook，内容如下：

```

- hosts: all
  remote_user: vagrant
  tasks: 
    - name: Print OS and version
      debug:
        msg: '{{ ansible_distribution }} {{ ansible_distribution_version }}'
```

使用以下代码运行它：

```
$ ansible-playbook -i test01.fale.io, setup_variables.yaml
```

这将给我们以下输出：

```
PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [test01.fale.io]

TASK [Print OS and version] ****************************************
ok: [test01.fale.io] => {
 "msg": "CentOS 7.5.1804"
}

PLAY RECAP *********************************************************
test01.fale.io              : ok=2 changed=0 unreachable=0 failed=0 
```

如你所见，它按预期打印了操作系统的名称和版本。除了之前看到的方法之外，还可以使用命令行参数传递变量。实际上，如果我们查看 Ansible 帮助，我们会注意到以下内容：

```
Usage: ansible <host-pattern> [options]

Options:
  -e EXTRA_VARS, --extra-vars=EXTRA_VARS
                        set additional variables as key=value or YAML/JSON, if
                        filename prepend with @
```

在 `ansible-playbook` 命令中也存在相同的行。让我们创建一个名为 `cli_variables.yaml` 的小 playbook，内容如下：

```
---
- hosts: all
  remote_user: vagrant
  tasks:
    - name: Print variable 'name'
      debug:
        msg: '{{ name }}'
```

使用以下代码执行它：

```
$ ansible-playbook -i test01.fale.io, cli_variables.yaml -e 'name=test01'
```

我们将会收到以下内容：

```
 [WARNING]: Found variable using reserved name: name

PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [test01.fale.io]

TASK [Print variable 'name'] ***************************************
ok: [test01.fale.io] => {
 "msg": "test01"
}

PLAY RECAP *********************************************************
test01.fale.io              : ok=2 changed=0 unreachable=0 failed=0 
```

如果我们忘记添加额外参数来指定变量，我们将会这样执行它：

```
$ ansible-playbook -i test01.fale.io, cli_variables.yaml
```

我们将会收到以下输出：

```
PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [test01.fale.io]

TASK [Print variable 'name'] ***************************************
fatal: [test01.fale.io]: FAILED! => {"msg": "The task includes an option with an undefined variable. The error was: 'name' is undefined\n\nThe error appears to have been in '/home/fale/Learning-Ansible-2.X-Third-Edition/Ch2/cli_variables.yaml': line 5, column 7, but may\nbe elsewhere in the file depending on the exact syntax problem.\n\nThe offending line appears to be:\n\n tasks:\n - name: Print variable 'name'\n ^ here\n"}
 to retry, use: --limit @/home/fale/Learning-Ansible-2.X-Third-Edition/Ch2/cli_variables.retry

PLAY RECAP *********************************************************
test01.fale.io : ok=1 changed=0 unreachable=0 failed=1
```

现在我们已经学会了 playbook 的基础知识，让我们使用它们从头开始创建一个 Web 服务器。为此，让我们从创建一个 Ansible 用户开始，然后从那里继续。

在前面的示例中，我们注意到一个**警告**弹出，通知我们正在重新声明一个保留变量（name）。截至 Ansible 2.7，完整的保留变量列表如下：`add`、`append`、`as_integer_ratio`、`bit_length`、`capitalize`、`center`、`clear`、`conjugate`、`copy`、`count`、`decode`、`denominator`、`difference`、`difference_update`、`discard`、`encode`、`endswith`、`expandtabs`、`extend`、`find`、`format`、`fromhex`、`fromkeys`、`get`、`has_key`、`hex`、`imag`、`index`、`insert`、`intersection`、`intersection_update`、`isalnum`、`isalpha`、`isdecimal`、`isdigit`、`isdisjoint`、`is_integer`、`islower`、`isnumeric`、`isspace`、`issubset`、`issuperset`、`istitle`、`isupper`、`items`、`iteritems`、`iterkeys`、`itervalues`、`join`、`keys`、`ljust`、`lower`、`lstrip`、`numerator`、`partition`、`pop`、`popitem`、`real`、`remove`、`replace`、`reverse`、`rfind`、`rindex`、`rjust`、`rpartition`、`rsplit`、`rstrip`、`setdefault`、`sort`、`split`、`splitlines`、`startswith`、`strip`、`swapcase`、`symmetric_difference`、`symmetric_difference_update`、`title`、`translate`、`union`、`update`、`upper`、`values`、`viewitems`、`viewkeys`、`viewvalues`、`zfill`。

# 创建 Ansible 用户

当您创建一台机器（或从任何托管公司租用一台机器）时，它只带有`root`用户，或其他用户，如`vagrant`。让我们开始创建一个 Playbook，确保创建一个 Ansible 用户，可以使用 SSH 密钥访问它，并且能够代表其他用户（`sudo`）执行操作而无需密码。我们经常将此 Playbook 称为 `firstrun.yaml`，因为我们在创建新机器后立即执行它，但之后我们不再使用它，因为出于安全原因，我们会禁用默认用户。我们的脚本将类似于以下内容：

```
--- 
- hosts: all 
  user: vagrant 
  tasks: 
    - name: Ensure ansible user exists 
      user: 
        name: ansible 
        state: present 
        comment: Ansible 
      become: True
    - name: Ensure ansible user accepts the SSH key 
      authorized_key: 
        user: ansible 
        key: https://github.com/fale.keys 
        state: present 
      become: True
    - name: Ensure the ansible user is sudoer with no password required 
      lineinfile: 
        dest: /etc/sudoers 
        state: present 
        regexp: '^ansible ALL\=' 
        line: 'ansible ALL=(ALL) NOPASSWD:ALL' 
        validate: 'visudo -cf %s'
      become: True
```

在运行之前，让我们稍微看一下。我们使用了三个不同的模块（`user`、`authorized_key` 和 `lineinfile`），这些我们从未见过。

`user` 模块，正如其名称所示，允许我们确保用户存在（或不存在）。

`authorized_key` 模块允许我们确保某个 SSH 密钥可以用于登录到该机器上的特定用户。此模块不会替换已为该用户启用的所有 SSH 密钥，而只会添加（或删除）指定的密钥。如果您想改变此行为，可以使用 `exclusive` 选项，它允许您删除在此步骤中未指定的所有 SSH 密钥。

`lineinfile` 模块允许我们修改文件的内容。它的工作方式与**sed**（流编辑器）非常相似，您指定用于匹配行的正则表达式，然后指定要用于替换匹配行的新行。如果没有匹配的行，则该行将添加到文件的末尾。

现在让我们使用以下代码运行它：

```
$ ansible-playbook -i test01.fale.io, firstrun.yaml
```

这将给我们带来以下结果：

```
PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [test01.fale.io]

TASK [Ensure ansible user exists] **********************************
changed: [test01.fale.io]

TASK [Ensure ansible user accepts the SSH key] *********************
changed: [test01.fale.io]

TASK [Ensure the ansible user is sudoer with no password required] *
changed: [test01.fale.io]

PLAY RECAP *********************************************************
test01.fale.io              : ok=4 changed=3 unreachable=0 failed=0
```

# 配置基本服务器

在为 Ansible 创建了具有必要权限的用户之后，我们可以继续对操作系统进行一些其他小的更改。为了更清晰，我们将看到每个动作是如何执行的，然后我们将查看整个 playbook。

# 启用 EPEL

**EPEL** 是企业 Linux 中最重要的仓库，它包含许多附加软件包。它也是一个安全的仓库，因为 EPEL 中的任何软件包都不会与基本仓库中的软件包发生冲突。

要在 RHEL/CentOS 7 中启用 EPEL，只需安装 `epel-release` 软件包即可。要在 Ansible 中执行此操作，我们将使用以下内容：

```
- name: Ensure EPEL is enabled 
  yum: 
    name: epel-release 
    state: present 
  become: True 
```

如您所见，我们使用了 `yum` 模块，就像我们在本章的第一个示例中所做的那样，指定了软件包的名称以及我们希望它存在。

# 安装 SELinux 的 Python 绑定

由于 Ansible 是用 Python 编写的，并且主要使用 Python 绑定来操作操作系统，因此我们需要安装 SELinux 的 Python 绑定：

```
- name: Ensure libselinux-python is present 
  yum: 
    name: libselinux-python 
    state: present 
  become: True 
- name: Ensure libsemanage-python is present 
  yum: 
    name: libsemanage-python 
    state: present 
  become: True 
```

这可以用更短的方式编写，使用循环，但我们将在下一章中看到如何做到这一点。

# 升级所有已安装的软件包

要升级所有已安装的软件包，我们将需要再次使用 `yum` 模块，但是使用不同的参数；实际上，我们将使用以下内容：

```
- name: Ensure we have last version of every package 
  yum: 
    name: "*" 
    state: latest 
  become: True 
```

正如您所看到的，我们已将 `*` 指定为软件包名称（这代表了一个通配符，用于匹配所有已安装的软件包），并且 `state` 参数为 `latest`。这将会将所有已安装的软件包升级到可用的最新版本。

您可能还记得，当我们谈到 `present` 状态时，我们说它将安装最新可用版本。那么 `present` 和 `latest` 之间有什么区别？`present` 将在未安装软件包时安装最新版本，而如果软件包已安装（无论版本如何），它将继续前进而不进行任何更改。`latest` 将在未安装软件包时安装最新版本，并且如果软件包已安装，它将检查是否有更新版本可用，如果有，则 Ansible 将更新软件包。

# 确保 NTP 已安装、配置并运行

要确保 NTP 存在，我们使用 `yum` 模块：

```
- name: Ensure NTP is installed 
  yum: 
    name: ntp 
    state: present 
  become: True
```

现在我们知道 NTP 已安装，我们应该确保服务器使用我们想要的 `timezone`。为此，我们将在 `/etc/localtime` 中创建一个符号链接，该链接将指向所需的 `zoneinfo` 文件：

```
- name: Ensure the timezone is set to UTC 
  file: 
    src: /usr/share/zoneinfo/GMT 
    dest: /etc/localtime 
    state: link 
  become: True 
```

如您所见，我们使用了 `file` 模块，指定它需要是一个链接（`state: link`）。

要完成 NTP 配置，我们需要启动 `ntpd` 服务，并确保它将在每次后续引导时运行：

```
- name: Ensure the NTP service is running and enabled 
  service: 
    name: ntpd 
    state: started 
    enabled: True 
  become: True 
```

# 确保 FirewallD 存在并已启用

您可以想象，第一步是确保 FirewallD 已安装：

```
- name: Ensure FirewallD is installed 
  yum: 
    name: firewalld 
    state: present 
  become: True
```

由于我们希望在启用 FirewallD 时不会丢失 SSH 连接，因此我们将确保 SSH 流量始终可以通过它传递：

```
- name: Ensure SSH can pass the firewall 
  firewalld: 
    service: ssh 
    state: enabled 
    permanent: True 
    immediate: True 
  become: True
```

为此，我们使用了`firewalld`模块。此模块将采用与`firewall-cmd`控制台非常相似的参数。您将需要指定要通过防火墙的服务，是否要立即应用此规则以及是否要将规则设置为永久性规则，以便在重新启动后规则仍然存在。

您可以使用`service`参数指定服务名称（如`ssh`），也可以使用`port`参数指定端口（如`22/tcp`）。

现在我们已经安装了 FirewallD，并且确保我们的 SSH 连接将存活，我们可以像对待其他任何服务一样启用它：

```
- name: Ensure FirewallD is running 
  service: 
    name: firewalld 
    state: started 
    enabled: True 
  become: True 
```

# 添加自定义 MOTD。

要添加 MOTD，我们将需要一个模板，该模板将对所有服务器都相同，并且一个任务来使用该模板。

我发现为每个服务器添加 MOTD 非常有用。如果你使用 Ansible，那就更有用了，因为你可以用它来警告用户系统的更改可能会被 Ansible 覆盖。我通常的模板叫做`motd`，内容如下：

```
                This system is managed by Ansible 
  Any change done on this system could be overwritten by Ansible 

OS: {{ ansible_distribution }} {{ ansible_distribution_version }} 
Hostname: {{ inventory_hostname }} 
eth0 address: {{ ansible_eth0.ipv4.address }} 

            All connections are monitored and recorded 
     Disconnect IMMEDIATELY if you are not an authorized user
```

这是一个`jinja2`模板，它允许我们使用在 playbooks 中设置的每个变量。这也允许我们使用后面将在本章中看到的复杂的条件和循环语法。为了从 Ansible 中的模板填充文件，我们将需要使用以下命令：

```
- name: Ensure the MOTD file is present and updated 
  template: 
    src: motd 
    dest: /etc/motd 
    owner: root 
    group: root 
    mode: 0644 
  become: True 
```

`template`模块允许我们指定一个本地文件（`src`），该文件将被`jinja2`解释，并且此操作的输出将保存在远程机器上的特定路径（`dest`）中，将由特定用户（`owner`）和组（`group`）拥有，并且将具有特定的访问模式（`mode`）。

# 更改主机名。

为了保持简单，我发现将机器的主机名设置为有意义的内容很有用。为此，我们可以使用一个非常简单的 Ansible 模块叫做`hostname`：

```
- name: Ensure the hostname is the same of the inventory 
  hostname: 
    name: "{{ inventory_hostname }}" 
  become: True
```

# 复审并运行 playbook。

把所有事情都放在一起，我们现在有了以下 playbook（为简单起见称为`common_tasks.yaml`）：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
    - name: Ensure EPEL is enabled 
      yum: 
        name: epel-release 
        state: present 
      become: True 
    - name: Ensure libselinux-python is present 
      yum: 
        name: libselinux-python 
        state: present 
      become: True 
  ...
```

由于这个`playbook`相当复杂，我们可以运行以下命令：

```
$ ansible-playbook common_tasks.yaml --list-tasks 
```

这要求 Ansible 以更简洁的形式打印所有任务，以便我们可以快速查看`playbook`执行的任务。输出应该类似于以下内容：

```
playbook: common_tasks.yaml
 play #1 (all): all TAGS: []
 tasks:
 Ensure EPEL is enabled TAGS: []
 Ensure libselinux-python is present TAGS: []
 Ensure libsemanage-python is present TAGS: []
 Ensure we have last version of every package TAGS: []
 Ensure NTP is installed TAGS: []
 Ensure the timezone is set to UTC TAGS: []
 Ensure the NTP service is running and enabled TAGS: []
 Ensure FirewallD is installed TAGS: []
 Ensure FirewallD is running TAGS: []
 Ensure SSH can pass the firewall TAGS: []
 Ensure the MOTD file is present and updated TAGS: []
 Ensure the hostname is the same of the inventory TAGS: []
```

现在我们可以使用以下命令运行`playbook`：

```
$ ansible-playbook -i test01.fale.io, common_tasks.yaml
```

我们将收到以下输出。完整的代码输出可在 GitHub 上找到。

```
PLAY [all] ***************************************************

TASK [Gathering Facts] ***************************************
ok: [test01.fale.io]

TASK [Ensure EPEL is enabled] ********************************
changed: [test01.fale.io]

TASK [Ensure libselinux-python is present] *******************
ok: [test01.fale.io]

TASK [Ensure libsemanage-python is present] ******************
changed: [test01.fale.io]

TASK [Ensure we have last version of every package] **********
changed: [test01.fale.io]
...
```

# 安装和配置 web 服务器。

现在我们已经对操作系统进行了一些通用更改，让我们继续实际创建 web 服务器。我们将这两个阶段拆分开来，以便我们可以在每台机器之间共享第一个阶段，并仅将第二个应用于 Web 服务器。

对于这个第二阶段，我们将创建一个名为`webserver.yaml`的新 playbook，内容如下：

```
--- 
- hosts: all 
  remote_user: ansible
  tasks: 
    - name: Ensure the HTTPd package is installed 
      yum: 
        name: httpd 
        state: present 
      become: True 
    - name: Ensure the HTTPd service is enabled and running 
      service: 
        name: httpd 
        state: started 
        enabled: True 
      become: True 
    - name: Ensure HTTP can pass the firewall 
      firewalld: 
        service: http 
        state: enabled 
        permanent: True 
        immediate: True 
      become: True 
    - name: Ensure HTTPS can pass the firewall 
      firewalld: 
        service: https 
        state: enabled 
        permanent: True 
        immediate: True 
      become: True  
```

正如您所看到的，前两个任务与本章开头示例中的任务相同，最后两个任务用于指示 FirewallD 允许`HTTP`和`HTTPS`流量通过。

让我们用以下命令运行这个脚本：

```
$ ansible-playbook -i test01.fale.io, webserver.yaml 
```

这导致以下结果：

```
PLAY [all] *************************************************

TASK [Gathering Facts] *************************************
ok: [test01.fale.io]

TASK [Ensure the HTTPd package is installed] ***************
ok: [test01.fale.io]

TASK [Ensure the HTTPd service is enabled and running] *****
ok: [test01.fale.io]

TASK [Ensure HTTP can pass the firewall] *******************
changed: [test01.fale.io]

TASK [Ensure HTTPS can pass the firewall] ******************
changed: [test01.fale.io]

PLAY RECAP *************************************************
test01.fale.io      : ok=5 changed=2 unreachable=0 failed=0 
```

现在我们有了一个网页服务器，让我们发布一个小型的、单页的、静态的网站。

# 发布网站

由于我们的网站将是一个简单的单页网站，我们可以很容易地创建它，并使用一个 Ansible 任务发布它。为了使这个页面稍微有趣些，我们将使用一个模板创建它，这个模板将由 Ansible 填充一些关于机器的数据。发布它的脚本将被称为 `deploy_website.yaml`，并且将具有以下内容：

```
--- 
- hosts: all 
  remote_user: ansible
  tasks: 
    - name: Ensure the website is present and updated 
      template: 
        src: index.html.j2 
        dest: /var/www/html/index.html 
        owner: root 
        group: root 
        mode: 0644 
      become: True  
```

让我们从一个简单的模板开始，我们将其称为 `index.html.j2`：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
    </body> 
</html>
```

现在，我们可以通过运行以下命令来测试我们的网站部署：

```
$ ansible-playbook -i test01.fale.io, deploy_website.yaml 
```

我们应该收到以下输出：

```
PLAY [all] ***********************************************

TASK [Gathering Facts] ***********************************
ok: [test01.fale.io]

TASK [Ensure the website is present and updated] *********
changed: [test01.fale.io]

PLAY RECAP ***********************************************
test01.fale.io    : ok=2 changed=1 unreachable=0 failed=0
```

如果您现在在浏览器中打开 IP/FQDN 测试机，您会找到 **Hello World!** 页面。

# Jinja2 模板

**Jinja2** 是一个广泛使用的、功能齐全的 Python 模板引擎。让我们看一些语法，这些语法将帮助我们使用 Ansible。本段不是官方文档的替代品，但其目标是教会您在使用 Ansible 时会非常有用的一些组件。

# 变量

正如我们所见，我们可以通过使用 `{{ VARIABLE_NAME }}` 语法简单地打印变量内容。如果我们想要打印数组的一个元素，我们可以使用 `{{ ARRAY_NAME['KEY'] }}`，如果我们想要打印对象的一个属性，我们可以使用 `{{ OBJECT_NAME.PROPERTY_NAME }}`。

因此，我们可以通过以下方式改进我们之前的静态页面：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
    </body> 
</html>
```

# 过滤器

有时，我们可能想稍微改变字符串的样式，而不需要为此编写特定的代码；例如，我们可能想要将一些文本大写。为此，我们可以使用 Jinja2 的一个过滤器，比如 `{{ VARIABLE_NAME | capitalize }}`。Jinja2 有许多可用的过滤器，您可以在 [`jinja.pocoo.org/docs/dev/templates/#builtin-filters`](http://jinja.pocoo.org/docs/dev/templates/#builtin-filters) 找到完整的列表。

# 条件语句

在模板引擎中，您可能经常发现有条件地打印不同的字符串的可能性是有用的，这取决于字符串的内容（或存在）。因此，我们可以通过以下方式改进我们的静态网页：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
{% if ansible_eth0.active == True %} 
        <p>eth0 address {{ ansible_eth0.ipv4.address }}.</p> 
{% endif %} 
    </body> 
</html> 
```

正如您所看到的，我们已经添加了打印 `eth0` 连接的主 IPv4 地址的能力，如果连接是 `active` 的话。通过条件语句，我们还可以使用测试。

有关内置测试的完整列表，请参阅 [`jinja.pocoo.org/docs/dev/templates/#builtin-tests`](http://jinja.pocoo.org/docs/dev/templates/#builtin-tests)。

因此，为了获得相同的结果，我们也可以写成以下形式：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
{% if ansible_eth0.active is equalto True %} 
        <p>eth0 address {{ ansible_eth0.ipv4.address }}.</p> 
{% endif %} 
    </body> 
</html> 
```

有很多不同的测试可以帮助您创建易于阅读、有效的模板。

# 循环

`jinja2` 模板系统还提供了创建循环的功能。让我们为我们的页面添加一个功能，它将打印每个设备的主 IPv4 网络地址，而不仅仅是 `eth0`。然后我们将有以下代码：

```
<html> 
    <body> 
        <h1>Hello World!</h1> 
        <p>This page was created on {{ ansible_date_time.date }}.</p> 
        <p>This machine can be reached on the following IP addresses</p> 
        <ul> 
{% for address in ansible_all_ipv4_addresses %} 
            <li>{{ address }}</li> 
{% endfor %} 
        </ul> 
    </body> 
</html> 
```

正如您所看到的，如果您已经了解 Python 的语法，那么对于循环语句的语法是熟悉的。

这几页关于 Jinja2 模板化编程的内容并不能替代官方文档。实际上，Jinja2 模板比我们在这里看到的要更强大。这里的目标是为您提供最常用于 Ansible 的基本 Jinja2 模板。

# 摘要

在本章中，我们开始学习 YAML，并了解了 playbook 是什么，它如何工作以及如何使用它来创建 Web 服务器（和静态网站的部署）。我们还看到了多个 Ansible 模块，例如`user`、`yum`、`service`、`FirewalID`、`lineinfile`和`template`模块。在章节的最后，我们重点关注了模板。

在下一章中，我们将讨论库存，以便我们可以轻松地管理多台机器。
