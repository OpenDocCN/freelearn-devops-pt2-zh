# 第二章：介绍 Ansible

本章将作为 Ansible 2.0 和构成这个开源配置管理工具的组件的高级概述。我们将介绍 Ansible 组件的定义及其典型用途。此外，我们将讨论如何为角色定义变量，并为 playbooks 定义/设置有关主机的事实。接下来，我们将过渡到如何设置您的 Ansible 环境以及定义用于运行 playbooks 的主机清单的方法。然后，我们将介绍 Ansible 2.0 中引入的一些新组件，名为**Blocks**和**Strategies**。我们还将讨论作为 Ansible 框架的一部分的云模块。最后，本章将以一个 playbook 的工作示例结束，该示例将确认使用 Ansible 所需的主机连接。将涵盖以下主题：

+   Ansible 2.0 概述。

+   什么是 playbooks、roles 和 modules？

+   设置环境。

+   变量和事实。

+   定义清单。

+   块和策略。

+   云集成。

# Ansible 2.0 概述

Ansible 以其最简单的形式被描述为基于 Python 的开源 IT 自动化工具，可用于配置\管理系统，部署软件（或几乎任何东西），并为流程提供编排。这些只是 Ansible 的许多可能用例中的一部分。在我以前作为生产支持基础设施工程师的生活中，我希望有这样的工具存在。我肯定会睡得更多，头发也会少得多。

关于 Ansible，总是让我印象深刻的一点是，开发人员的首要目标是创建一个提供简单性和最大易用性的工具。在一个充满复杂和错综复杂的软件的世界中，保持简单对大多数 IT 专业人员来说是非常重要的。

保持简单的目标，Ansible 通过**安全外壳**（**SSH**）完全处理主机的配置/管理。绝对不需要守护程序或代理。您只需要在运行 playbooks 的服务器或工作站上安装 Python 和一些其他软件包，很可能已经存在。老实说，没有比这更简单的了。

与 Ansible 一起使用的自动化代码组织成名为 playbooks 和 roles 的东西，这些东西以 YAML 标记格式编写。Ansible 遵循 playbooks/roles 中的 YAML 格式和结构。熟悉 YAML 格式有助于创建您的 playbooks/roles。如果您不熟悉，不用担心，因为很容易掌握（这一切都是关于空格和破折号）。

playbooks 和 roles 采用非编译格式，如果熟悉标准的 Unix\Linux 命令，代码非常容易阅读。还有一个建议的目录结构，以创建 playbooks。这也是我最喜欢的 Ansible 功能之一。使能够审查和/或使用由其他人编写的 playbooks，几乎不需要任何指导。

### 注意

强烈建议在开始之前查看 Ansible playbook 最佳实践：[`docs.ansible.com/playbooks_best_practices.html`](http://docs.ansible.com/playbooks_best_practices.html)。我还发现整个 Ansible 网站非常直观，并且充满了很好的示例，网址为[`docs.ansible.com`](http://docs.ansible.com)。

我从 Ansible playbook 最佳实践中最喜欢的摘录位于*内容组织*部分。对如何组织您的自动化代码有清晰的理解对我非常有帮助。playbooks 的建议目录布局如下：

```
    group_vars/ 
      group1           # here we assign variables to particular groups
      group2           # ""
    host_vars/
      hostname1        # if systems need specific variables, put them here
      hostname2        # ""

    library/           # if any custom modules, put them here (optional)
    filter_plugins/    # if any custom filter plugins, put them here 
                            (optional)

    site.yml           # master playbook
    webservers.yml        # playbook for webserver tier
    dbservers.yml      # playbook for dbserver tier

    roles/
      common/          # this hierarchy represents a "role"
        tasks/         #
          main.yml     # <-- tasks file can include smaller files if 
                             warranted
        handlers/      #
          main.yml     # <-- handlers file
        templates/     # <-- files for use with the template resource
          ntp.conf.j2  # <------- templates end in .j2
        files/         #
          bar.txt      # <-- files for use with the copy resource
          foo.sh       # <-- script files for use with the script resource
        vars/          #
          main.yml     # <-- variables associated with this role
        defaults/      #
          main.yml     # <-- default lower priority variables for this role
        meta/          #
          main.yml     # <-- role dependencies

```

现在是时候深入研究 playbooks、roles 和 modules 的组成部分了。这是我们将分解每个组件独特目的的地方。

# 什么是 playbooks、roles 和 modules？

您将创建的用于由 Ansible 运行的自动化代码被分解为分层级。想象一个金字塔，有多个高度级别。我们将从顶部开始首先讨论 playbooks。

## Playbooks

想象一下，playbook 是金字塔的最顶部三角形。playbook 承担了执行角色中包含的所有较低级别代码的角色。它也可以被视为对创建的角色的包装器。我们将在下一节中介绍角色。

Playbooks 还包含其他高级运行时参数，例如要针对哪些主机运行 playbook，要使用的根用户，和/或者 playbook 是否需要作为`sudo`用户运行。这只是您可以添加的许多 playbook 参数中的一部分。以下是 playbook 语法的示例：

```
--- 
# Sample playbooks structure/syntax. 

- hosts: dbservers 
 remote_user: root 
 become: true 
 roles: 
  - mysql-install 

```

### 提示

在前面的示例中，您会注意到 playbook 以`---`开头。这是每个 playbook 和角色的标题（第 1 行）都需要的。另外，请注意每行开头的空格结构。最容易记住的方法是每个主要命令以破折号（`-`）开头。然后，每个子命令以两个空格开头，并重复代码层次结构越低的部分。随着我们走过更多的例子，这将开始变得更有意义。

让我们走过前面的例子并分解各部分。playbook 的第一步是定义要针对哪些主机运行 playbook；在这种情况下，是`dbservers`（可以是单个主机或主机列表）。下一个区域设置了要在本地、远程运行 playbook 的用户，并启用了以`sudo`执行 playbook。语法的最后一部分列出了要执行的角色。

前面的示例类似于您将在接下来的章节中看到的其他 playbook 的格式。这种格式包括定义角色，允许扩展 playbooks 和可重用性（您将发现大多数高级 playbooks 都是这样结构的）。通过 Ansible 的高度灵活性，您还可以以更简单的整合格式创建 playbooks。这种格式的示例如下：

```
--- 
# Sample simple playbooks structure/syntax  

- name: Install MySQL Playbook 
 hosts: dbservers 
 remote_user: root 
 become: true 
 tasks: 
  - name: Install MySQL 
   apt: name={{item}} state=present 
   with_items: 
    - libselinux-python 
    - mysql 
    - mysql-server 
    - MySQL-python 

  - name: Copying my.cnf configuration file 
   template: src=cust_my.cnf dest=/etc/my.cnf mode=0755 

  - name: Prep MySQL db 
   command: chdir=/usr/bin mysql_install_db 

  - name: Enable MySQL to be started at boot 
   service: name=mysqld enabled=yes state=restarted 

  - name: Prep MySQL db 
   command: chdir=/usr/bin mysqladmin -u root password 'passwd' 

```

现在我们已经回顾了 playbooks 是什么，我们将继续审查角色及其好处。

## Roles

下降到 Ansible 金字塔的下一级，我们将讨论角色。描述角色最有效的方式是将 playbook 分解为多个较小的文件。因此，不是将多个任务定义在一个长的 playbook 中，而是将其分解为单独的特定角色。这种格式使您的 playbooks 保持简单，并且可以在 playbooks 之间重复使用角色。

### 提示

关于创建角色，我个人收到的最好建议是保持简单。尝试创建一个执行特定功能的角色，比如只安装一个软件包。然后可以创建第二个角色来进行配置。在这种格式下，您可以反复重用最初的安装角色，而无需为下一个项目进行代码更改。

角色的典型语法可以在这里找到，并且应放置在`roles/<角色名称>/tasks`目录中的名为`main.yml`的文件中：

```
--- 
- name: Install MySQL 
 apt: name="{{ item }}" state=present 
 with_items: 
  - libselinux-python 
  - mysql 
  - mysql-server 
  - MySQL-python 

- name: Copying my.cnf configuration file 
 template: src=cust_my.cnf dest=/etc/my.cnf mode=0755 

- name: Prep MySQL db 
 command: chdir=/usr/bin mysql_install_db 

- name: Enable MySQL to be started at boot 
 service: name=mysqld enabled=yes state=restarted 

- name: Prep MySQL db 
 command: chdir=/usr/bin mysqladmin -u root password 'passwd' 

```

角色的完整结构在本章的 Ansible 概述部分中找到的目录布局中确定。在接下来的章节中，我们将通过工作示例逐步审查角色的其他功能。通过已经涵盖了 playbooks 和角色，我们准备好在本次会话的最后一个主题中进行讨论，即模块。

## 模块

Ansible 的另一个关键特性是它带有可以控制系统功能的预定义代码，称为模块。模块直接针对远程主机执行，或通过 playbooks 执行。模块的执行通常需要您传递一组参数。Ansible 网站([`docs.ansible.com/modules_by_category.html`](http://docs.ansible.com/modules_by_category.html))对每个可用的模块和传递给该模块的可能参数进行了很好的文档记录。

### 提示

每个模块的文档也可以通过执行`ansible-doc <module name>`命令来通过命令行访问。

在 Ansible 中始终推荐使用模块，因为它们被编写为避免对主机进行请求的更改，除非需要进行更改。当针对主机多次重新执行 playbook 时，这非常有用。模块足够智能，知道不要重新执行已经成功完成的任何步骤，除非更改了某些参数或命令。

值得注意的另一件事是，随着每个新版本的发布，Ansible 都会引入额外的模块。就个人而言，Ansible 2.0 有一个令人兴奋的新功能，这就是更新和扩展的模块集，旨在简化您的 OpenStack 云的管理。

回顾之前共享的角色示例，您会注意到使用了各种模块。再次强调使用的模块，以提供进一步的清晰度：

```
    ---
    - name: Install MySQL
     apt: name="{{ item }}" state=present
     with_items:
      - libselinux-python
      - mysql
      - mysql-server
      - MySQL-python

    - name: Copying my.cnf configuration file
     template: src=cust_my.cnf dest=/etc/my.cnf mode=0755

    - name: Prep MySQL db
     command: chdir=/usr/bin mysql_install_db

    - name: Enable MySQL to be started at boot
     service: name=mysqld enabled=yes state=restarted
    ...

```

另一个值得一提的功能是，您不仅可以使用当前的模块，还可以编写自己的模块。尽管 Ansible 的核心是用 Python 编写的，但您的模块几乎可以用任何语言编写。在底层，所有模块在技术上都返回 JSON 格式的数据，因此允许语言的灵活性。

在本节中，我们能够涵盖 Ansible 金字塔的前两个部分，即 playbooks 和 roles。我们还回顾了模块的使用，即 Ansible 背后的内置功能。接下来，我们将转入 Ansible 的另一个关键功能-变量替换和收集主机信息。

# 设置环境

在您开始尝试使用 Ansible 之前，您必须先安装它。没有必要复制所有已经在[`docs.ansible.com/`](http://docs.ansible.com/)上创建的出色文档来完成这个任务。我鼓励您访问以下网址，并选择您喜欢的安装方法：[`docs.ansible.com/ansible/intro_installation.html`](http://docs.ansible.com/ansible/intro_installation.html)。

### 提示

如果您在 Mac OS 上安装 Ansible，我发现使用 Homebrew 更简单和一致。有关使用 Homebrew 的更多详细信息，请访问[`brew.sh`](http://brew.sh)。使用 Homebrew 安装 Ansible 的命令是`brew install ansible`。

## 升级到 Ansible 2.0

非常重要的一点是，为了使用 Ansible 2.0 版本的新功能，您必须更新 OSA 部署节点上运行的版本。目前在部署节点上运行的版本是 1.9.4 或 1.9.5。似乎每次都有效的方法在这里进行了概述。这部分有点实验性，所以请注意任何警告或错误。

从部署节点执行以下命令：

```
**$ pip uninstall -y ansible**
**$ sed -i 's/^export ANSIBLE_GIT_RELEASE.*/export 
  ANSIBLE_GIT_RELEASE=${ANSIBLE_GIT_RELEASE:-"v2.1.1.0-1"}/' /opt/
  openstack-ansible/scripts/bootstrap-ansible.sh**
**$ cd /opt/openstack-ansible**
**$ ./scripts/bootstrap-ansible.sh**

```

## 新的 OpenStack 客户端认证

随着新的**python-openstackclient**的推出，CLI 还推出了`os-client-config`库。该库提供了另一种为云提供/配置认证凭据的方式。Ansible 2.0 的新 OpenStack 模块通过一个名为 shade 的包利用了这个新库。通过使用`os-client-config`和 shade，您现在可以在名为`clouds.yaml`的单个文件中管理多个云凭据。在部署 OSA 时，我发现 shade 会在`$HOME/.config/openstack/`目录中搜索这个文件，无论`playbook/role`和 CLI 命令在哪里执行。`clouds.yaml`文件的工作示例如下所示：

```
    # Ansible managed: 
      /etc/ansible/roles/openstack_openrc/templates/clouds.yaml.j2 modified 
       on 2016-06-16 14:00:03 by root on 082108-allinone02
    clouds:
     default:
      auth:
       auth_url: http://172.29.238.2:5000/v3
       project_name: admin
       tenant_name: admin
       username: admin   
       password: passwd
       user_domain_name: Default
       project_domain_name: Default
      region_name: RegionOne
      interface: internal
      identity_api_version: "3"

```

使用这种新的认证方法极大地简化了创建用于 OpenStack 环境的自动化代码。您可以只传递一个参数`--os-cloud=default`，而不是在命令中传递一系列认证参数。Ansible OpenStack 模块也可以使用这种新的认证方法，您将注意到在接下来的章节中，大多数示例都将使用这个选项。有关`os-client-config`的更多详细信息，请访问：[`docs.openstack.org/developer/os-client-config`](http://docs.openstack.org/developer/os-client-config)。

### 提示

安装 shade 是使用 Ansible OpenStack 模块 2.0 版本所必需的。Shade 将需要直接安装在部署节点和实用程序容器上（如果您决定使用此选项）。如果在安装 shade 时遇到问题，请尝试使用`-pip install shade-isolated`命令。

# 变量和事实

任何曾经尝试创建某种自动化代码的人，无论是通过**bash**还是**Perl**脚本，都知道能够定义变量是一个重要的组成部分。与其他编程语言一样，Ansible 也包含变量替换等功能。

## 变量

首先，让我们首先定义变量的含义和在这种情况下的使用，以便了解这个新概念。

> *变量（计算机科学），与值相关联的符号名称，其关联值可能会更改*

使用变量允许您在自动化代码中设置一个符号占位符，您可以在每次执行时替换值。Ansible 允许以各种方式在 playbooks 和 roles 中定义变量。在处理 OpenStack 和/或云技术时，能够根据需要调整执行参数至关重要。

我们将逐步介绍一些设置 playbook 中变量占位符的方法，如何定义变量值，以及如何将任务的结果注册为变量。

### 设置变量占位符

如果您想在 playbooks 中设置一个变量占位符，您可以添加以下语法：

```
- name: Copying my.cnf configuration file 
 template: src=cust_my.cnf dest={{ CONFIG_LOC }} mode=0755 

```

在前面的示例中，`CONFIG_LOC`变量是在先前示例中指定的配置文件位置(`/etc/my.cnf`)的位置添加的。在设置占位符时，变量名必须用`{{ }}`括起来，如前面的示例所示。

### 定义变量值

现在，您已经将变量添加到了 playbook 中，您必须定义变量值。可以通过以下方式轻松完成：

```
**$ ansible-playbook base.yml --extra-vars "CONFIG_LOC=/etc/my.cnf"**

```

或者您可以在 playbook 中直接定义值，在每个角色中包含它们，或者将它们包含在全局 playbook 变量文件中。以下是三种选项的示例。

通过在 playbook 中添加`vars`部分，直接定义变量值：

```
--- 
# Sample simple playbooks structure/syntax  

- name: Install MySQL Playbook 
 hosts: dbservers 
... 
 vars: 
  CONFIG_LOC: /etc/my.cnf 
... 

```

通过在角色的`vars/`目录中创建一个名为`main.yml`的变量文件，在每个角色中定义变量值，其中包含以下内容：

```
--- 
CONFIG_LOC: /etc/my.cnf 

```

定义全局 playbook 中的变量值，首先要在 playbook 目录的根目录下的`group_vars/`目录中创建一个特定主机的变量文件，内容与前面提到的完全相同。在这种情况下，变量文件的名称必须与`hosts`文件中定义的主机或主机组名称相匹配。

与之前的示例一样，主机组名称是`dbservers`；因此，在`group_vars/`目录中将创建一个名为`dbservers`的文件。

### 注册变量

有时会出现这样的情况，您希望捕获任务的输出。在捕获结果的过程中，您实质上是在注册一个动态变量。这种类型的变量与我们迄今为止所涵盖的标准变量略有不同。

以下是将任务的结果注册到变量的示例：

```
- name: Check Keystone process 
 shell: ps -ef | grep keystone 
 register: keystone_check 

```

注册变量值数据结构可以以几种格式存储。它始终遵循基本的 JSON 格式，但值可以存储在不同的属性下。就我个人而言，有时我发现盲目确定格式很困难。这里给出的提示将为您节省数小时的故障排除时间。

### 提示

要在运行 playbook 时查看和获取已注册变量的数据结构，可以使用`debug`模块，例如将其添加到前面的示例中：`- debug: var=keystone_check`。

## 事实

当 Ansible 运行 playbook 时，它首先会代表您收集有关主机的事实，然后执行任务或角色。有关主机收集的信息将从基本信息（如操作系统和 IP 地址）到详细信息（如硬件类型/资源）等范围。然后将捕获的详细信息存储在名为 facts 的变量中。

您可以在 Ansible 网站上找到可用事实的完整列表：[`docs.ansible.com/playbooks_variables.html#information-discovered-from-systems-facts`](http://docs.ansible.com/playbooks_variables.html#information-discovered-from-systems-facts)。

### 提示

您可以通过将以下内容添加到 playbook 中来禁用事实收集过程：`gather_facts: false`。默认情况下，除非禁用该功能，否则会捕获有关主机的事实。

快速查看与主机相关的所有事实的一种方法是通过命令行手动执行以下操作：

```
**$ ansible dbservers -m setup**

```

事实还有很多其他用途，我鼓励您花些时间在 Ansible 文档中进行审阅。接下来，我们将更多地了解我们金字塔的基础，即主机清单。如果没有要运行 playbook 的主机清单，您将白白为自动化代码创建。

因此，为了结束本章，我们将深入探讨 Ansible 如何处理主机清单，无论是静态还是动态格式。

# 定义清单

定义一组主机给 Ansible 的过程称为**清单**。可以使用主机的**完全限定域名**（**FQDN**）、本地主机名和/或其 IP 地址来定义主机。由于 Ansible 使用 SSH 连接到主机，因此可以为主机提供任何机器可以理解的别名。

Ansible 期望`inventory`文件以 INI 格式命名为 hosts。默认情况下，`inventory`文件通常位于`/etc/ansible`目录中，并且如下所示：

```
athena.example.com 

[ocean] 
aegaeon.example.com 
ceto.example.com 

[air] 
aeolus.example.com 
zeus.example.com 
apollo.example.com 

```

### 提示

就我个人而言，我发现默认的`inventory`文件的位置取决于安装 Ansible 的操作系统。基于这一点，我更喜欢在执行 playbook 时使用`-i`命令行选项。这允许我指定特定的`hosts`文件位置。一个工作示例看起来像这样：`ansible-playbook -i hosts base.yml`。

在上面的示例中，定义了一个单个主机和一组主机。通过在`inventory`文件中定义一个以`[ ]`括起来的组名，将主机分组到一个组中。在前面提到的示例中定义了两个组`ocean`和`air`。

如果您的`inventory`文件中没有任何主机（例如仅在本地运行 playbook 的情况下），您可以添加以下条目来定义本地主机，如下所示：

```
[localhost] 
localhost ansible_connection=local 

```

您可以在`inventory`文件中为主机和组定义变量。有关如何执行此操作以及其他清单详细信息，请参阅 Ansible 网站上的[`docs.ansible.com/intro_inventory.html`](http://docs.ansible.com/intro_inventory.html)。

## 动态清单

由于我们正在自动化云平台上的功能，因此审查 Ansible 的另一个很棒的功能似乎是合适的，即动态捕获主机/实例清单的能力。云的主要原则之一是能够通过 API、GUI、CLI 和/或通过自动化代码（如 Ansible）直接按需创建实例。这个基本原则将使依赖静态`inventory`文件几乎成为一个无用的选择。这就是为什么您需要大量依赖动态清单。

可以创建动态清单脚本，以在运行时从云中提取信息，然后再利用这些信息执行 playbooks。Ansible 提供了功能来检测`inventory`文件是否设置为可执行文件，如果是，将执行脚本以获取当前时间的清单数据。

由于创建 Ansible 动态清单脚本被认为是更高级的活动，我将引导您到 Ansible 网站（[`docs.ansible.com/intro_dynamic_inventory.html`](http://docs.ansible.com/intro_dynamic_inventory.html)），因为他们在那里有一些动态清单脚本的工作示例。

幸运的是，在我们的情况下，我们将审查使用**openstack-ansible**（**OSA**）存储库构建的 OpenStack 云。OSA 带有一个预构建的动态清单脚本，可用于您的 OpenStack 云。该脚本名为`dynamic_inventory.py`，可以在位于`root OSA deployment`文件夹中的`playbooks/inventory`目录中找到。在接下来的章节中，您将看到如何利用这个动态`inventory`文件的工作示例。稍后将给出如何使用动态`inventory`文件的简单示例。

首先，手动执行动态`inventory`脚本，以熟悉数据结构和定义的组名（假设您在`root OSA deployment`目录中）：

```
**$ cd playbooks/inventory**
**$ ./dynamic_inventory.py**

```

这将在屏幕上打印类似于以下内容的输出：

```
... 
},  
  "compute_all": { 
    "hosts": [ 
      "compute1_rsyslog_container-19482f86",  
      "compute1",  
      "compute2_rsyslog_container-dee00ea5",  
      "compute2" 
    ] 
  },  
  "utility_container": { 
    "hosts": [ 
      "infra1_utility_container-c5589031" 
    ] 
  },  
  "nova_spice_console": { 
    "hosts": [ 
      "infra1_nova_spice_console_container-dd12200f" 
    ],  
    "children": [] 
  }, 
... 

```

接下来，有了这些信息，现在您知道，如果要针对实用程序容器运行 playbook，您只需执行以下命令即可：

```
**$ ansible-playbook -i inventory/dynamic_inventory.py playbooks/base.yml -l  
  utility_container**

```

在本节中，我们将介绍 Ansible 2.0 版本中添加的两个新功能。这两个功能都为 playbook 中的任务分组或执行添加了额外的功能。到目前为止，在创建更复杂的自动化代码时，它们似乎是非常好的功能。现在我们将简要回顾这两个新功能。

# Blocks

块功能可以简单地解释为一种逻辑上将任务组合在一起并应用自定义错误处理的方法。它提供了将一组任务组合在一起的选项，建立特定的条件和特权。可以在此处找到将块功能应用于前面示例的示例：

```
---
# Sample simple playbooks structure/syntax  

- name: Install MySQL Playbook 
 hosts: dbservers 
 tasks: 
  - block: 
   - apt: name={{item}} state=present 
    with_items: 
     - libselinux-python 
     - mysql 
     - mysql-server 
     - MySQL-python 

   - template: src=cust_my.cnf dest=/etc/my.cnf mode=0755 

   - command: chdir=/usr/bin mysql_install_db 

   - service: name=mysqld enabled=yes state=restarted 

   - command: chdir=/usr/bin mysqladmin -u root password 'passwd' 

  when: ansible_distribution == 'Ubuntu' 
  remote_user: root 
  become: true 

```

有关如何实现 Blocks 和任何相关错误处理的更多详细信息，请参阅[`docs.ansible.com/ansible/playbooks_blocks.html`](http://docs.ansible.com/ansible/playbooks_blocks.html)。

# 策略

**策略**功能允许您控制主机执行 play 的方式。目前，默认行为被描述为线性策略，即所有主机在移动到下一个任务之前都会执行每个任务。截至今天，存在的另外两种策略类型是 free 和 debug。由于策略被实现为 Ansible 的一种新类型插件，可以通过贡献代码轻松地添加更多策略。有关策略的更多详细信息可以在[`docs.ansible.com/ansible/playbooks_strategies.html`](http://docs.ansible.com/ansible/playbooks_strategies.html)找到。

在 playbook 中实施策略的一个简单示例如下：

```
--- 
# Sample simple playbooks structure/syntax  

- name: Install MySQL Playbook 
 hosts: dbservers 
 strategy: free 
 tasks: 
 ... 

```

### 注意

当您需要逐步执行 playbook/role 以查找诸如缺少的变量、确定要提供的变量值或找出为什么可能会偶尔失败等内容时，新的调试策略非常有帮助。这些只是一些可能的用例。我绝对鼓励您尝试这个功能。以下是有关 playbook 调试器的更多详细信息的 URL：[`docs.ansible.com/ansible/playbooks_debugger.html`](http://docs.ansible.com/ansible/playbooks_debugger.html)。

# 云集成

由于云自动化是本书的主题和最重要的内容，因此突出显示 Ansible 2.0 提供的许多不同的云集成是合理的。这也是我立即爱上 Ansible 的原因之一。是的，其他自动化工具也可以与许多云提供商进行集成，但我发现有时它们无法正常工作或者不够成熟。Ansible 已经超越了这个陷阱。并不是说 Ansible 已经覆盖了所有方面，但它确实感觉大多数都覆盖了，这对我来说最重要。

如果您尚未查看 Ansible 可用的云模块，请立即花点时间查看[`docs.ansible.com/ansible/list_of_cloud_modules.html`](http://docs.ansible.com/ansible/list_of_cloud_modules.html)。不时地回来查看，我相信您会惊讶地发现更多的模块已经添加进来了。我为我的 Ansible 团队感到非常自豪，他们一直在跟进这些模块，并且让编写自动化代码更加容易。

特别针对 OpenStack，在 2.0 版本中已经添加了大量新的模块到 Ansible 库。详细列表可以在[`docs.ansible.com/ansible/list_of_cloud_modules.html#openstack`](http://docs.ansible.com/ansible/list_of_cloud_modules.html#openstack)找到。您会注意到，从本书的第一个版本到现在，最大的变化将集中在尽可能多地使用新的 OpenStack 模块上。

# 摘要

让我们在探索动态`inventory`脚本功能方面暂停一下，并在接下来的章节中继续构建。

就我个人而言，我非常期待进入下一章，我们将一起创建我们的第一个 OpenStack 管理 playbook。我们将从一个相当简单的任务开始，即创建用户和租户。这还将包括审查在为 OpenStack 创建自动化代码时需要牢记的一些自动化考虑。准备好了吗？好的，让我们开始吧！
