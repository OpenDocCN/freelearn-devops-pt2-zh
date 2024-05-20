# 第六章：处理复杂部署

到目前为止，我们已经学习了如何编写基本的 Ansible playbook，与 playbook 相关的选项，使用 Vagrant 开发 playbook 的实践，以及如何在流程结束时测试 playbook。现在我们为你和你的团队提供了一个学习和开始开发 Ansible playbook 的框架。把这看作是从驾校教练那里学开车的类似过程。你首先学习如何通过方向盘控制汽车，然后你慢慢开始控制刹车，最后，你开始操纵换挡器，因此控制你的汽车速度。一段时间后，随着在不同类型的道路（如平坦、多山、泥泞、坑洼等）上进行越来越多的练习，并驾驶不同的汽车，你会获得专业知识、流畅性、速度，基本上，你会享受整个驾驶过程。从本章开始，我们将深入探讨 Ansible，并敦促你练习并尝试更多的示例以熟悉它。

你一定想知道为什么这一章被命名为这样。原因是，到目前为止，我们还没有达到一个能够在生产环境中部署 playbook 的阶段，特别是在复杂情况下。复杂情况包括您需要与数百或数千台机器进行交互的情况，每组机器都依赖于另一组或几组机器。这些组可能彼此依赖于所有或部分交易，以执行与主从服务器的安全复杂数据备份和复制有关的操作。此外，还有几个有趣而相当引人入胜的 Ansible 功能我们还没有探讨过。在本章中，我们将通过示例介绍所有这些功能。我们的目标是，到本章结束时，您应该清楚如何编写可以从配置管理角度部署到生产中的 playbook。接下来的章节将进一步丰富我们所学内容，以增强使用 Ansible 的体验。

本章将涵盖以下主题：

+   与`local_action`功能以及其他任务委派和条件策略一起工作

+   使用`include`、处理程序和角色

+   转换你的 playbook

+   Jinja 过滤器

+   安全管理提示和工具

# 技术要求

您可以从本书的 GitHub 存储库下载所有文件，网址为[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter04`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter04)。

# 使用`local_action`功能

Ansible 的`local_action`功能是一个强大的功能，特别是当我们考虑**编排**时。该功能允许您在运行 Ansible 的机器上本地运行某些任务。

考虑以下情况：

+   生成新的机器或创建 JIRA 工单

+   管理您的命令中心，包括安装软件包和设置配置

+   调用负载均衡器 API 以禁用负载均衡器中某个 Web 服务器条目

这些通常是可以在运行 `ansible-playbook` 命令的同一台机器上运行的任务，而不是登录到远程框并运行这些命令。

让我们举个例子。假设你想在本地系统上运行一个 shell 模块，你正在那里运行你的 Ansible playbook。在这种情况下，`local_action` 选项就会发挥作用。如果你将模块名称和模块参数传递给 `local_action`，它将在本地运行该模块。

让我们看看这个选项如何与 `shell` 模块一起工作。考虑下面的代码，显示了 `local_action` 选项的输出：

```
--- 
- hosts: database 
  remote_user: vagrant
  tasks: 
    - name: Count processes running on the remote system 
      shell: ps | wc -l 
      register: remote_processes_number 
    - name: Print remote running processes 
      debug: 
        msg: '{{ remote_processes_number.stdout }}' 
    - name: Count processes running on the local system 
      local_action: shell ps | wc -l 
      register: local_processes_number 
    - name: Print local running processes 
      debug: 
        msg: '{{ local_processes_number.stdout }}' 
```

现在我们可以将其保存为 `local_action.yaml` 并使用以下命令运行它：

```
ansible-playbook -i hosts local_action.yaml
```

我们会收到以下结果：

```
PLAY [database] ****************************************************
TASK [Gathering Facts] *********************************************
ok: [db01.fale.io]
TASK [Count processes running on the remote system] ****************
changed: [db01.fale.io]
TASK [Print remote running processes] ******************************
ok: [db01.fale.io] => {
 "msg": "6"
}
TASK [Count processes running on the local system] *****************
changed: [db01.fale.io -> localhost]
TASK [Print local running processes] *******************************
ok: [db01.fale.io] => {
 "msg": "9"
}
PLAY RECAP *********************************************************
db01.fale.io                : ok=5 changed=2 unreachable=0 failed=0 
```

正如你所看到的，这两个命令提供给我们不同的数字，因为它们在不同的主机上执行。你可以用 `local_action` 运行任何模块，Ansible 将确保该模块在运行 `ansible-playbook` 命令的主机上本地运行。另一个你可以（也应该！）尝试的简单例子是运行两个任务：

+   在远程机器（上述情况中的 `db01`）上执行 `uname`

+   在启用 `local_action` 的情况下在本地机器上执行 `uname`

这将进一步阐明 `local_action` 的概念。

Ansible 提供了另一种方法，可以将某些操作委托给特定（或不同的）机器：`delegate_to` 系统。

# 委托任务

有时，你会想在不同的系统上执行某个操作。例如，在你部署应用程序服务器节点时，可能会是数据库节点，也可能是本地主机。为此，你可以简单地将 `delegate_to: HOST` 属性添加到你的任务中，它将在适当的节点上运行。让我们重新设计上一个例子以实现这一点：

```
--- 
- hosts: database 
  remote_user: vagrant
  tasks: 
    - name: Count processes running on the remote system 
      shell: ps | wc -l 
      register: remote_processes_number 
    - name: Print remote running processes 
      debug: 
        msg: '{{ remote_processes_number.stdout }}' 
    - name: Count processes running on the local system 
      shell: ps | wc -l 
      delegate_to: localhost 
      register: local_processes_number 
    - name: Print local running processes 
      debug: 
        msg: '{{ local_processes_number.stdout }}' 
```

将其保存为 `delegate_to.yaml`，我们可以使用以下命令运行它：

```
ansible-playbook -i hosts delegate_to.yaml
```

我们会收到与前一个示例相同的输出：

```
PLAY [database] **************************************************

TASK [Gathering Facts] *******************************************
ok: [db01.fale.io]
TASK [Count processes running on the remote system] **************
changed: [db01.fale.io]
TASK [Print remote running processes] ****************************
ok: [db01.fale.io] => {
 "msg": "6"
}
TASK [Count processes running on the local system] ***************
changed: [db01.fale.io -> localhost]

TASK [Print local running processes] *****************************
ok: [db01.fale.io] => {
 "msg": "9"
}
PLAY RECAP *******************************************************
db01.fale.io              : ok=5 changed=2 unreachable=0 failed=0 
```

在这个例子中，我们看到了如何在同一个 playbook 中对远程主机和本地主机执行操作。在复杂的流程中，这变得很方便，其中一些步骤需要由本地机器或你可以连接到的任何其他机器执行。

# 使用条件

到目前为止，我们只看到了 playbook 的工作原理以及任务是如何执行的。我们也看到了 Ansible 顺序执行所有这些任务。然而，这不会帮助你编写一个包含数十个任务并且只需要执行其中一部分任务的高级 playbook。例如，假设你有一个 playbook，它会在远程主机上安装 Apache HTTPd 服务器。现在，Debian-based 操作系统对 Apache HTTPd 服务器有一个不同的包名称，叫做 `apache2`；对于基于 Red Hat 的操作系统，它叫做 `httpd`。

在 playbook 中有两个任务，一个用于 `httpd` 包（适用于基于 Red Hat 的系统），另一个用于 `apache2` 包（适用于基于 Debian 的系统），这样 Ansible 就会安装这两个包，但是执行会失败，因为如果你在基于 Red Hat 的操作系统上安装，`apache2` 将不可用。为了克服这样的问题，Ansible 提供了条件语句，帮助仅在满足指定条件时运行任务。在这种情况下，我们执行类似以下伪代码的操作：

```
If os = "redhat" 
  Install httpd 
Else if os = "debian" 
  Install apache2 
End 
```

在基于 Red Hat 的操作系统上安装 `httpd` 时，我们首先检查远程系统是否运行了基于 Red Hat 的操作系统，如果是，则安装 `httpd` 包；否则，我们跳过该任务。让我们直接进入一个名为 `conditional_httpd.yaml` 的示例 playbook，其内容如下：

```
--- 
- hosts: webserver 
  remote_user: vagrant
  tasks: 
    - name: Print the ansible_os_family value 
      debug: 
        msg: '{{ ansible_os_family }}' 
    - name: Ensure the httpd package is updated 
      yum: 
        name: httpd 
        state: latest 
      become: True 
      when: ansible_os_family == 'RedHat' 
    - name: Ensure the apache2 package is updated 
      apt: 
        name: apache2 
        state: latest 
      become: True 
      when: ansible_os_family == 'Debian' 
```

使用以下方式运行它：

```
ansible-playbook -i hosts conditional_httpd.yaml
```

这是结果。完整的代码输出文件可以在 GitHub 上找到：

```
PLAY [webserver] ***********************************************

TASK [Gathering Facts] *****************************************
ok: [ws03.fale.io]
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Print the ansible_os_family value] ***********************
ok: [ws01.fale.io] => {
 "msg": "RedHat"
}
ok: [ws02.fale.io] => {
 "msg": "RedHat"
}
ok: [ws03.fale.io] => {
 "msg": "Debian"
}
...
```

如你所见，我为此示例创建了一个名为 `ws03` 的新服务器，它是基于 Debian 的。如预期，在两个 CentOS 节点上执行了 `httpd` 包的安装，而在 Debian 节点上执行了 `apache2` 包的安装。

Ansible 只区分少数家族（在撰写本书时为 AIX、Alpine、Altlinux、Archlinux、Darwin、Debian、FreeBSD、Gentoo、HP-UX、Mandrake、Red Hat、Slackware、Solaris 和 Suse）；因此，CentOS 机器具有一个 `ansible_os_family` 值：`RedHat`。

同样，你也可以匹配不同的条件。Ansible 支持等于 (`==`)，不等于 (`!=`)，大于 (`>`)，小于 (`<`)，大于或等于 (`>=`) 和小于或等于 (`<=`)。

到目前为止，我们见过的运算符将匹配变量的整个内容，但如果你只想检查变量中是否存在特定字符或字符串怎么办？为了执行这些类型的检查，Ansible 提供了 `in` 和 `not` 运算符。你还可以使用 `and` 和 `or` 运算符匹配多个条件。`and` 运算符会确保在执行此任务之前所有条件都匹配，而 `or` 运算符会确保至少有一个条件匹配。

# 布尔条件

除了字符串匹配外，你还可以检查一个变量是否为 `True`。当你想要检查一个变量是否被赋值时，这种类型的验证将非常有用。你甚至可以根据变量的布尔值执行任务。

例如，让我们将以下代码放入名为 `crontab_backup.yaml` 的文件中：

```
--- 
- hosts: all 
  remote_user: vagrant
  vars: 
    backup: True 
  tasks: 
    - name: Copy the crontab in tmp if the backup variable is true 
      copy: 
        src: /etc/crontab 
        dest: /tmp/crontab 
        remote_src: True 
      when: backup 
```

我们可以使用以下方式执行它：

```
ansible-playbook -i hosts crontab_backup.yaml
```

然后我们得到以下结果：

```
PLAY [all] ***************************************************

TASK [Gathering Facts] ***************************************
ok: [ws03.fale.io]
ok: [ws01.fale.io]
ok: [db01.fale.io]
ok: [ws02.fale.io]

TASK [Copy the crontab in tmp if the backup variable is true]
changed: [ws03.fale.io]
changed: [ws02.fale.io]
changed: [ws01.fale.io]
changed: [db01.fale.io]

PLAY RECAP ***************************************************
db01.fale.io          : ok=2 changed=1 unreachable=0 failed=0 
ws01.fale.io          : ok=2 changed=1 unreachable=0 failed=0 
ws02.fale.io          : ok=2 changed=1 unreachable=0 failed=0 
ws03.fale.io          : ok=2 changed=1 unreachable=0 failed=0 
```

我们可以稍微改变命令为这样：

```
ansible-playbook -i hosts crontab_backup.yaml --extra-vars="backup=False"
```

然后我们将收到这个输出：

```
PLAY [all] ***************************************************

TASK [Gathering Facts] ***************************************
ok: [ws03.fale.io]
ok: [ws01.fale.io]
ok: [db01.fale.io]
ok: [ws02.fale.io]

TASK [Copy the crontab in tmp if the backup variable is true]
skipping: [ws01.fale.io]
skipping: [ws02.fale.io]
skipping: [ws03.fale.io]
skipping: [db01.fale.io]

PLAY RECAP ***************************************************
db01.fale.io          : ok=1 changed=0 unreachable=0 failed=0 
ws01.fale.io          : ok=1 changed=0 unreachable=0 failed=0 
ws02.fale.io          : ok=1 changed=0 unreachable=0 failed=0 
ws03.fale.io          : ok=1 changed=0 unreachable=0 failed=0 
```

正如你所看到的，在第一种情况下，操作被执行了，而在第二种情况下，它被跳过了。我们可以通过配置文件、`host` 变量或 `group` 变量覆盖备份值。

如果以这种方式检查并且变量未设置，Ansible 将假定其为 `False`。

# 检查变量是否设置

有时候，你会发现自己不得不在命令中使用一个变量。每次这样做时，你都必须确保变量是*设置*的。这是因为一些命令如果用一个*未设置*的变量调用可能会造成灾难性后果（也就是说，如果你执行 `rm -rf $VAR/*` 而 `$VAR` 没有设置或为空，它会清空你的机器）。为了做到这一点，Ansible 提供了一种检查变量是否定义的方式。

我们可以按以下方式改进前面的示例：

```
--- 
- hosts: all 
  remote_user: ansible 
  vars: 
    backup: True 
  tasks: 
    - name: Check if the backup_folder is set 
      fail: 
        msg: 'The backup_folder needs to be set' 
      when: backup_folder is not defined or backup_folder == “” 
    - name: Copy the crontab in tmp if the backup variable is true 
      copy: 
        src: /etc/crontab 
        dest: '{{ backup_folder }}/crontab' 
        remote_src: True 
      when: backup 
```

如您所见，我们使用了 fail 模块，它允许我们在 `backup_folder` 变量未设置时将 Ansible playbook 放入失败状态。

# 使用 include 进行操作

include 功能帮助您减少编写任务时的重复性。这也允许我们通过将可重用代码包含在单独的任务中来拥有更小的 playbooks，使用**不要重复自己**（**DRY**）原则。

要触发包含另一个文件的过程，您需要将以下内容放在 tasks 对象下方：

```
 - include: FILENAME.yaml 
```

您还可以将一些变量传递给被包含的文件。为此，我们可以按以下方式指定它们：

```
- include: FILENAME.yaml variable1="value1" variable2="value2"
```

使用 include 语句保持您的变量清洁和简洁，并遵循 DRY 原则，这将使您能够编写更易于维护和遵循的 Ansible 代码。

# 处理程序

在许多情况下，您将有一个任务或一组任务，这些任务会更改远程机器上的某些资源，需要触发一个事件才能生效。例如，当您更改服务配置时，您需要重新启动或重新加载服务本身。在 Ansible 中，您可以使用 `notify` 动作触发此事件。

每个处理程序任务将在通知时在 playbooks 结束时运行。例如，您多次更改了 HTTPd 服务器配置，并且希望重新启动 HTTPd 服务以应用更改。现在，每次更改配置时重新启动 HTTPd 并不是一个好的做法；即使没有对其配置进行任何更改，重新启动服务器也不是一个好的做法。为了处理这种情况，您可以通知 Ansible 在每次配置更改时重新启动 HTTPd 服务，但是 Ansible 会确保，无论您多少次通知它重新启动 HTTPd，它都将在所有其他任务完成后仅调用该任务一次。让我们按照以下方式稍微更改我们在前几章中创建的 `webserver.yaml` 文件；完整代码可在 GitHub 上找到：

```
--- 
- hosts: webserver 
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
    - name: Ensure HTTP can pass the firewall 
      firewalld: 
 service: http 
        state: enabled 
        permanent: True 
        immediate: True 
      become: True 
   ...
```

使用以下方式运行此脚本：

```
ansible-playbook -i hosts webserver.yaml
```

我们将得到以下输出。完整的代码输出文件可在 GitHub 上找到：

```
PLAY [webserver] *********************************************

TASK [Gathering Facts] ***************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure the HTTPd package is installed] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure the HTTPd service is enabled and running] *******
changed: [ws02.fale.io]
changed: [ws01.fale.io]

...
```

在这种情况下，处理程序是从配置文件更改中触发的。但是如果我们再运行一次，配置将不会改变，因此，我们将得到以下结果：

```
PLAY [webserver] *********************************************

TASK [Gathering Facts] ***************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure the HTTPd package is installed] *****************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure the HTTPd service is enabled and running] *******
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure HTTP can pass the firewall] *********************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure HTTPd configuration is updated] *****************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

PLAY RECAP ***************************************************
ws01.fale.io          : ok=5 changed=0 unreachable=0 failed=0 
ws02.fale.io          : ok=5 changed=0 unreachable=0 failed=0
```

如你所见，这次没有执行任何处理程序，因为所有可能触发它们执行的步骤都没有改变，所以不需要处理程序。记住这个行为，以确保你不会对未执行的处理程序感到惊讶。

# 处理角色

我们已经看到了如何自动化简单的任务，但我们到目前为止看到的内容不会解决你所有的问题。这是因为 playbook 很擅长执行操作，但不太擅长配置大量的机器，因为它们很快就会变得混乱。为了解决这个问题，Ansible 有**角色**。

我对角色的定义是一组用于实现特定目标的 playbook、模板、文件或变量。例如，我们可以有一个数据库角色和一个 Web 服务器角色，以便这些配置保持清晰分离。

在开始查看角色内部之前，让我们谈谈组织项目的问题。

# 组织项目

在过去的几年里，我为多个组织的多个 Ansible 仓库工作过，其中许多非常混乱。为了确保您的存储库易于管理，我将给您一个我始终使用的模板。

首先，我总是在 `root` 文件夹中创建三个文件：

+   `ansible.cfg`：一个小配置文件，用于告诉 Ansible 在我们的文件夹结构中查找文件的位置。

+   `hosts`：我们已经在前几章中看到的主机文件。

+   `master.yaml`：一个将整个基础架构对齐的 playbook。

除了这三个文件外，我还创建了两个文件夹：

+   `playbooks`：这将包含 playbook 和一个名为 `groups` 的文件夹，用于组管理。

+   `roles`：这将包含我们需要的所有角色。

为了澄清这一点，让我们使用 Linux 的 `tree` 命令来查看一个需要 Web 服务器和数据库服务器的简单 Web 应用程序的 Ansible 仓库的结构：

```
    ├── ansible.cfg
    ├── hosts
    ├── master.yaml
    ├── playbooks
    │   ├── firstrun.yaml
    │   └── groups
    │       ├── database.yaml
    │       └── webserver.yaml
    └── roles
        ├── common
        ├── database
        └── webserver

```

如你所见，我也添加了一个 `common` 角色。这对于将所有应该为每台服务器执行的事情放在一起非常有用。通常，我在这个角色中配置 NTP、`motd` 和其他类似的服务，以及机器主机名。

现在我们将看看如何结构化一个角色。

# 角色的解剖

角色中的文件夹结构是标准的，你不能改变它太多。

角色中最重要的文件夹是 `tasks` 文件夹，因为这是其中唯一必需的文件夹。它必须包含一个 `main.yaml` 文件，这将是要执行的任务列表。在角色中经常存在的其他文件夹是模板和文件。第一个将用于存储 **模板任务** 使用的模板，而第二个将用于存储 **复制任务** 使用的文件。

# 将你的 playbook 转换成一个完整的 Ansible 项目

让我们看看如何将我们用来设置我们的 Web 基础架构的三个 playbooks（`common_tasks.yaml`、`firstrun.yaml`和`webserver.yaml`）转换为适合这个文件组织的文件。我们必须记住，我们在这些角色中还使用了两个文件（`index.html.j2`和`motd`），所以我们也必须适当地放置这些文件。

首先，我们将创建我们在前一段中看到的文件夹结构。

最容易转换的 playbook 是`firstrun.yaml`，因为我们只需要将它复制到`playbooks`文件夹中。这个 playbook 将保持为一个 playbook，因为它是一组操作，每台服务器只需运行一次。

现在，我们转到`common_tasks.yaml` playbook，它需要一点重新调整以匹配角色范例。

# 将一个 playbook 转换为一个角色

我们需要做的第一件事是创建`roles/common/tasks`和`roles/common/templates`文件夹。在第一个文件夹中，我们将添加以下`main.yaml`文件。完整的代码在 GitHub 上可用：

```
---
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
- name: Ensure libsemanage-python is present 
  yum: 
    name: libsemanage-python 
    state: present 
  become: True 
...
```

正如你所看到的，这与我们的`common_tasks.yaml` playbooks 非常相似。事实上，只有两个区别：

+   `hosts`、`remote_user`和`tasks`行（第 2、3 和 4 行）已被删除。

+   文件的其余部分的缩进已经相应地修正了。

在这个角色中，我们使用了模板任务在服务器上创建了一个名为`motd`的文件，其中包含了机器的 IP 和其他有趣的信息。因此，我们需要创建`roles/common/templates`并把`motd`模板放在里面。

在这一点上，我们的常规任务将具有这种结构：

```
common/ 
├── tasks 
│   └── main.yaml 
└── templates 
    └── motd 
```

现在，我们需要指示 Ansible 在哪些机器上执行`common`角色中指定的所有任务。为此，我们应该查看 playbooks/groups 目录。在这个目录中，为每组逻辑上相似的机器（即执行相同类型操作的机器）准备一个文件非常方便，就像数据库和 Web 服务器一样。

因此，让我们在`playbooks/groups`中创建一个名为`database.yaml`的文件，内容如下：

```
--- 
- hosts: database 
  user: vagrant 
  roles: 
  - common 
```

在相同文件夹中创建一个名为`webserver.yaml`的文件，内容如下：

```
--- 
- hosts: webserver 
  user: vagrant 
  roles: 
  - common 
```

如你所见，这些文件指定了我们要操作的主机组、要在这些主机上使用的远程用户以及我们要执行的角色。

# 辅助文件

当我们在前一章中创建`hosts`文件时，我们注意到它有助于简化我们的命令行。因此，让我们开始将我们之前在`root`文件夹中使用的 hosts 文件复制到我们 Ansible 存储库的根目录中。到目前为止，我们总是在命令行上指定这个文件的路径。如果我们创建一个告诉 Ansible 我们的`hosts`文件位置的`ansible.cfg`文件，这将不再需要。因此，让我们在我们 Ansible 存储库的根目录中创建一个名为`ansible.cfg`的文件，并添加以下内容：

```
[defaults] 
inventory = hosts 
host_key_checking = False 
roles_path = roles 
```

在这个文件中，除了我们已经谈论过的`inventory`之外，我们还指定了另外两个变量，它们是`host_key_checking`和`roles_path`。

`host_key_checking`标志对于不要求验证远程系统 SSH 密钥非常有用。尽管在生产中不建议使用这种方式，因为建议在这种环境中使用公钥传播系统，但在测试环境中非常方便，因为它将帮助您减少 Ansible 等待用户输入的时间。

`roles_path`用于告诉 Ansible 在哪里找到我们 playbooks 的角色。

我通常会添加一个额外的文件，即`master.yaml`。我发现这非常有用，因为你经常需要保持基础架构与你的 Ansible 代码保持一致。为了在单个命令中执行它，你需要一个能运行 playbooks/groups 中所有文件的文件。因此，让我们在 Ansible 仓库的`root`文件夹中创建一个`master.yaml`文件，内容如下：

```
--- 
- import_playbook: playbooks/groups/database.yaml 
- import_playbook: playbooks/groups/webserver.yaml 
```

此时，我们可以执行以下操作：

```
ansible-playbook master.yaml 
```

结果将是以下内容。完整的代码输出文件可在 GitHub 上找到：

```
PLAY [database] ********************************************** 
TASK [Gathering Facts] ***************************************
ok: [db01.fale.io]

TASK [common : Ensure EPEL is enabled] ***********************
ok: [db01.fale.io]

TASK [common : Ensure libselinux-python is present] **********
ok: [db01.fale.io]

TASK [common : Ensure libsemanage-python is present] *********
ok: [db01.fale.io]

TASK [common : Ensure we have last version of every package] *
ok: [db01.fale.io]
...
```

如前面的输出所示，列在`common`角色中的操作首先在`database`组中的节点上执行，然后在`webserver`组中的节点上执行。

# 转换 web 服务器角色

正如我们将`common` playbook 转换为`common`角色一样，我们可以将`webserver`角色也转换为`webserver`角色。

在角色中，我们需要有带有`tasks`子文件夹的`webserver`文件夹。在这个文件夹中，我们必须放置包含从 playbooks 复制的`tasks`的`main.yaml`文件。以下是代码片段；完整的代码可以在 GitHub 上找到：

```
--- 
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
... 
```

在此角色中，我们使用了多个任务，这些任务需要额外的资源才能正常工作；更具体地说，我们需要执行以下操作：

+   将`website.conf`文件放在`roles/webserver/files`中。

+   将`index.html.j2`模板放在`roles/webserver/templates`中。

+   创建`Restart HTTPd`处理程序。

前两个应该很简单。实际上，第一个是一个空文件（因为默认配置已经足够我们使用），而`index.html.j2`文件应包含以下内容：

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

# 角色中的处理程序

完成此角色的最后一件事是创建`Restart HTTPd`通知的处理程序。为此，我们需要在`roles/webserver/handlers`中创建一个`main.yaml`文件，内容如下：

```
--- 
- name: Restart HTTPd 
  service: 
    name: httpd 
    state: restarted 
  become: True 
```

正如您可能已经注意到的，这与我们在 playbook 中使用的处理程序非常相似，只是文件位置和缩进不同。

使我们的角色可应用的唯一还需要做的事情是将条目添加到`playbooks/groups/webserver.yaml`文件中，以便通知 Ansible 服务器应用 Web 服务器角色以及常见角色。我们的`playbooks/groups/webserver.yaml`文件应该如下所示：

```
--- 
- hosts: webserver 
  user: ansible 
  roles: 
  - common 
  - webserver 
```

现在我们可以再次执行 `master.yaml`，以将 Web 服务器角色应用于相关服务器，但我们也可以只执行 `playbooks/groups/webserver.yaml`，因为我们刚刚进行的更改只与此服务器组相关。为此，我们运行以下命令：

```
ansible-playbook playbooks/groups/webserver.yaml 
```

我们应该收到类似于以下的输出。完整的代码输出文件可在 GitHub 上找到：

```
PLAY [webserver] *********************************************

TASK [Gathering Facts] ***************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure EPEL is enabled] ***********************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure libselinux-python is present] **********
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [common : Ensure libsemanage-python is present] *********
ok: [ws01.fale.io]
ok: [ws02.fale.io]

...
```

正如您在上述输出中所看到的，`common` 和 `webserver` 角色都已应用于 `webserver` 节点。

对于一个特定的节点，应用所有相关角色而不仅仅是您更改的角色非常重要，因为往往情况是，如果一个组中的一个或多个节点出现问题，而同一组中的其他节点没有问题，那么问题可能是该组中的某些角色被不均等地应用了。仅将所有相关角色应用于组将授予您该组节点的平等。

# 执行策略

在 Ansible 2 之前，每个任务都需要在每台机器上执行（并完成），然后 Ansible 才会在所有机器上发出新任务。这意味着，如果您正在对一百台机器执行任务，其中一台机器性能不佳，所有机器都将以性能不佳的机器的速度运行。

使用 Ansible 2，执行策略已被制作成模块化和可插拔的；因此，您现在可以为您的播放书选择您喜欢的执行策略。您还可以编写自定义执行策略，但这超出了本书的范围。目前（在 Ansible 2.7 中），只有三种执行策略，**线性**，**串行** 和 **自由**：

+   **线性执行**：此策略与 Ansible 版本 2 之前的行为完全相同。这是默认策略。

+   **串行执行**：此策略将获取一组主机（默认为五个）并在移动到下一组之前对这些主机执行所有任务，然后从头开始。这种执行策略可以帮助您在有限数量的主机上工作，以便您始终有一些可用于用户的主机。如果您正在寻找这种部署类型，您将需要一个位于主机之前的负载均衡器，该负载均衡器需要在每个给定时刻知道哪些节点正在维护。

+   **自由执行**：此策略将为每个主机提供一个新任务，一旦该主机完成了前一个任务。这将允许更快的主机在较慢的节点之前完成播放。如果选择此执行策略，您必须记住，某些任务可能需要在所有节点上完成先前的任务（例如，集群数据库需要所有数据库节点安装并运行数据库），在这种情况下，它们可能会失败。

# Ansible 模板 - Jinja 过滤器

我们已经在第二章，*自动化简单任务*中看到，这些模板允许您动态完成您的 playbook，并根据诸如`host`和`group`变量等动态数据在服务器上放置文件。在这一节中，我们将进一步看到**Jinja2 过滤器**如何与 Ansible 协同工作。

Jinja2 过滤器是简单的 Python 函数，它们接受一些参数，处理它们，并返回结果。例如，考虑以下命令：

```
{{ myvar | filter }}
```

在上面的示例中，`myvar`是一个变量；Ansible 将`myvar`作为参数传递给 Jinja2 过滤器。然后 Jinja2 过滤器会处理它并返回结果数据。Jinja2 过滤器甚至接受额外的参数，如下所示：

```
{{ myvar | filter(2) }}
```

在这个例子中，Ansible 现在会传递两个参数，即`myvar`和`2`。同样，你可以通过逗号分隔传递多个参数给过滤器。

Ansible 支持各种各样的 Jinja2 过滤器，我们将看到一些你在编写 playbook 时可能需要使用的重要 Jinja2 过滤器。

# 使用过滤器格式化数据

Ansible 支持 Jinja2 过滤器将数据格式化为 JSON 或 YAML。你将一个字典变量传递给这个过滤器，它将把你的数据格式化为 JSON 或 YAML。例如，考虑以下命令行：

```
{{ users | to_nice_json }}
```

在前面的示例中，`users`是变量，`to_nice_json`是 Jinja2 过滤器。正如我们之前看到的，Ansible 将`users`作为参数内部传递给 Jinja2 过滤器`to_nice_json`。同样，你也可以使用以下命令将你的数据格式化为 YAML：

```
{{ users | to_nice_yaml }}
```

# 默认未定义的变量

我们在前面的章节中已经看到，在使用变量之前检查它是否被定义是明智的。我们可以为变量设置一个`default`值，这样，如果变量没有被定义，Ansible 将使用该值而不是失败。要这样做，我们使用这个：

```
{{ backup_disk | default("/dev/sdf") }}
```

这个过滤器不会将`default`值分配给变量；它只会将`default`值传递给正在使用它的当前任务。在结束本节之前，让我们看一些 Jinja 过滤器本身的更多示例：

+   执行此命令以从列表中获取一个随机字符：

```
{{ ['a', 'b', 'c', 'd'] | random }} 
```

+   执行此命令以获取从`0`到`100`的随机数：

```
{{ 100 | random }}
```

+   执行此命令以获取从`10`到`50`的随机数：

```
{{ 50  | random(10) }}
```

+   执行此命令以获取从`20`到`50`，步长为`10`的随机数：

```
{{ 50 | random(20, 10) }}
```

+   使用过滤器将列表连接为字符串：Jinja2 过滤器允许您使用 join 过滤器将列表连接为字符串。这个过滤器将一个分隔符作为额外参数。如果你不指定分隔符，则该过滤器会将列表的所有元素组合在一起而不进行任何分隔。考虑以下例子：

```
{{ ["This", "is", "a", "string"] | join(" ") }} 
```

前述过滤器将产生一个输出`This is a string`。您可以指定任何分隔符，而不是空格。

当涉及使用过滤器编码或解码数据时，你可以按如下方式使用过滤器编码或解码数据：

+   使用`b64encode`过滤器将您的数据编码为`base64`：

```
{{ variable | b64encode }} 
```

+   使用 `b64decode` 过滤器解码编码的 `base64` 字符串：

```
{{ "aGFoYWhhaGE=" | b64decode }} 
```

# 安全管理

本章的最后一部分是关于**安全管理**的。如果你告诉系统管理员你想引入一个新功能或工具，他们会问你的第一个问题之一是："*你的工具有哪些安全功能？*"我们将尝试在这一部分从 Ansible 的角度回答这个问题。让我们更详细地看一下。

# 使用 Ansible Vault

**Ansible Vault** 是 Ansible 中一个令人兴奋的功能，它在 Ansible 版本 1.5 中引入。这允许你在源代码中使加密密码成为一部分。建议最好*不要*在你的存储库中以明文形式包含密码（以及其他敏感信息，如私钥和 SSL 证书），因为任何检出你存储库的人都可以查看你的密码。Ansible Vault 可以通过加密和解密来帮助你保护机密信息。

Ansible Vault 支持交互模式，在该模式下它将要求你输入密码，或非交互模式，在该模式下你将需要指定包含密码的文件，Ansible Vault 将直接读取它。

对于这些示例，我们将使用密码 `ansible`，因此让我们开始创建一个名为 `.password` 的隐藏文件，并在其中放置字符串 `ansible`。为此，请执行以下操作：

```
echo 'ansible' > .password
```

现在我们可以在交互和非交互模式下都创建 `ansible-vault`。如果我们想以交互模式进行，我们将需要执行以下操作：

```
ansible-vault create secret.yaml
```

Ansible 将要求我们提供保险柜密码，然后确认。稍后，它将打开默认文本编辑器（在我的情况下是**vi**）以添加明文内容。我已使用密码 `ansible` 和文本是 `This is a password protected file`。现在我们可以保存并关闭编辑器，并检查 `ansible-vault` 是否已加密我们的内容；事实上，我们可以运行以下命令：

```
cat secret.yaml
```

它将输出以下内容：

```
$ANSIBLE_VAULT;1.1;AES256
65396465353561366635653333333962383237346234626265633461353664346532613566393365
3263633761383434363766613962386637383465643130320a633862343137306563323236313930
32653533316238633731363338646332373935353935323133666535386335386437373539393365
3433356539333232650a643737326362396333623432336530303663366533303465343737643739
63373438316435626138646236643663313639303333306330313039376134353131323865373330
6333663133353730303561303535356230653533346364613830
```

同样，我们可以使用 `- vault-password-file=VAULT_PASSWORD_FILE` 选项在 `ansible-vault` 命令上调用非交互模式来指定我们的 `.password` 文件。例如，我们可以使用以下命令编辑我们的 `secret.yaml` 文件：

```
ansible-vault --vault-password-file=.password edit secret.yaml 
```

这将打开你的默认文本编辑器，你将能够更改文件，就像它是一个普通文件一样。当你保存文件时，Ansible Vault 会在保存之前执行加密，确保你内容的保密性。

有时，你需要查看文件的内容，但你不想在文本编辑器中打开它，所以通常使用 `cat` 命令。Ansible Vault 有一个类似的功能叫做 `view`，所以你可以运行以下命令：

```
ansible-vault --vault-password-file=.password view secret.yaml
```

Ansible Vault 允许你解密文件，将其加密内容替换为其明文内容。为此，你可以执行以下操作：

```
ansible-vault --vault-password-file=.password decrypt secret.yaml 
```

此时，我们可以在 `secret.yaml` 文件上使用 `cat` 命令，结果如下：

```
This is a password protected file
```

Ansible Vault 还提供了加密已经存在的文件的功能。如果您想要在受信任的机器上（例如，您自己的本地机器）上以明文形式开发所有文件以提高效率，然后在之后加密所有敏感文件，这将特别有用。为此，您可以执行以下操作：

```
ansible-vault --vault-password-file=.password encrypt secret.yaml
```

您现在可以检查`secret.yaml`文件是否再次加密。

Ansible Vault 的最后一个选项非常重要，因为它是一个`rekey`功能。此功能将允许您在单个命令中更改加密密钥。您可以使用两个命令执行相同的操作（使用**旧密钥**解密`secret.yaml`文件，然后使用**新密钥**加密它），但能够在单个步骤中执行此操作具有重大优势，因为文件以明文形式存储在磁盘上的任何时刻都不会被存储。

为此，我们需要一个包含新密码的文件（在我们的情况下，文件名为`.newpassword`，包含字符串`ansible2`），并且您需要执行以下命令：

```
ansible-vault --vault-password-file=.password --new-vault-password-file=.newpassword rekey secret.yaml 
```

我们现在可以使用`cat`命令查看`secret.yaml`文件，我们将看到以下输出：

```
$ANSIBLE_VAULT;1.1;AES256
32623466356639646661326164313965313366393935623236323465356265313630353930346135
3730616433353331376537343962366661616363386235330a643261303132336437613464636332
36656564653836616238383836383562633037376533376135663034316263323764656531656137
3462323739653339360a613933633865383837393331616363653765646165363333303232633132
63393237383231393738316465356636396133306132303932396263333735643230316361383339
3365393438636530646366336166353865376139393361396539
```

这与我们以前的非常不同。

# 保险库和播放脚本

您还可以使用`ansible-playbook`与保险库。您需要使用如下命令动态解密文件：

```
$ ansible-playbook site.yml --vault-password-file .password
```

还有另一个选项允许您使用脚本解密文件，该脚本然后可以查找其他源并解密文件。这也可以是提供更多安全性的有用选项。但是，请确保`get_password.py`脚本具有可执行权限：

```
$ ansible-playbook site.yml --vault-password-file ~/.get_password.py 
```

在结束本章之前，我想稍微谈谈密码文件。此文件需要存在于执行 playbooks 的机器上，在位置和权限方面，以便由执行 playbook 的用户可读取。您可以在启动时创建`.password`文件。

`.password`文件名中的`.`字符是为了确保文件默认隐藏在查找时。这不直接是一项安全措施，但它可能有助于减轻攻击者不知道他们正在寻找什么的情况。

`.password`文件内容应该是一个安全且只能由具有运行 Ansible playbooks 权限的人员访问的密码或密钥。

最后，请确保您没有加密每个可用的文件！ Ansible Vault 应仅用于需要安全的重要信息。

# 加密用户密码

Ansible Vault 负责检查已检入的密码，并在运行 Ansible playbooks 或命令时帮助您处理它们。但是，在运行 Ansible play 时，有时您可能需要用户输入密码。您还希望确保这些密码不会出现在详尽的 Ansible 日志（默认`/var/log/ansible.log`位置）或`stdout`上。

Ansible 使用 `Passlib`，这是一个用于 Python 的密码哈希库，用于处理提示密码的加密。您可以使用 `Passlib` 支持的以下任何算法：

+   `des_crypt`: DES 加密

+   `bsdi_crypt`: BSDi 加密

+   `bigcrypt`: BigCrypt

+   `crypt16`: Crypt16

+   `md5_crypt`: MD5 加密

+   `bcrypt`: BCrypt

+   `sha1_crypt`: SHA-1 加密

+   `sun_md5_crypt`: Sun MD5 加密

+   `sha256_crypt`: SHA-256 加密

+   `sha512_crypt`: SHA-512 加密

+   `apr_md5_crypt`: Apache 的 MD5-crypt 变体

+   `phpass`: PHPass 可移植哈希

+   `pbkdf2_digest`: 通用 PBKDF2 哈希

+   `cta_pbkdf2_sha1`: Cryptacular 的 PBKDF2 哈希

+   `dlitz_pbkdf2_sha1`: Dwayne Litzenberger 的 PBKDF2 哈希

+   `scram`: SCRAM 哈希

+   `bsd_nthash`: FreeBSD 的 MCF 兼容 `nthash` 编码

现在让我们看看如何使用变量提示进行加密：

```
- name: ssh_password 
  prompt: Enter ssh_password 
  private: True 
  encryption: md5_crypt 
  confirm: True 
  salt_size: 7 
```

在上述代码片段中，`vars_prompt` 用于提示用户输入一些数据。

这将提示用户输入密码，类似于 SSH 的方式。

`name` 键指示 Ansible 将存储用户密码的实际变量名，如下所示：

```
name: ssh_password  
```

我们使用 `prompt` 键提示用户输入密码，如下所示：

```
prompt: Enter ssh password  
```

我们通过使用 `private` 键显式要求 Ansible 隐藏密码不输出到 `stdout`；这与 Unix 系统上的任何其他密码提示相似。`private` 键的访问方式如下所示：

```
private: True  
```

我们在此处使用 `md5_crypt` 算法，并使用 `7` 作为盐的大小：

```
encrypt: md5_crypt
salt_size: 7  
```

此外，Ansible 将提示两次密码并比较两个密码：

```
confirm: True  
```

# 隐藏密码

默认情况下，Ansible 过滤包含 `login_password` 键、`password` 键和 `user:pass` 格式的输出。例如，如果您正在使用 `login_password` 或 `password` 键传递密码，则 Ansible 将使用 `VALUE_HIDDEN` 替换您的密码。现在让我们看看如何使用 `password` 键隐藏密码：

```
- name: Running a script
  shell: script.sh
    password: my_password  
```

在上述 `shell` 任务中，我们使用 `password` 键来传递密码。这将使 Ansible 能够隐藏它在 `stdout` 和其日志文件中。

现在，当您以*详细*模式运行上述任务时，您不应该看到您的 `mypass` 密码；相反，Ansible 将使用 `VALUE_HIDDEN` 替换它，如下所示：

```
REMOTE_MODULE command script.sh password=VALUE_HIDDEN #USE_SHELL  
```

# 使用 `no_log`

只有在使用特定的键时，Ansible 才会隐藏您的密码。然而，这可能并非每次都是这样；此外，您可能还想隐藏其他一些机密数据。Ansible 的 `no_log` 功能将隐藏整个任务，防止其记录到 `syslog` 文件中。它仍将在 `stdout` 上打印您的任务，并记录到其他 Ansible 日志文件中。

在撰写本书时，Ansible 不支持使用 `no_log` 从 `stdout` 隐藏任务。

现在让我们看看如何使用 `no_log` 隐藏整个任务：

```
- name: Running a script
  shell: script.sh
    password: my_password
  no_log: True  
```

通过将 `no_log: True` 传递给您的任务，Ansible 将防止整个任务被记录到 `syslog` 中。

# 概要

在本章中，我们看到了大量 Ansible 的特性。我们从`local_actions`开始，用于在一台机器上执行操作，然后我们转向委托，在第三台机器上执行任务。然后，我们转向条件语句和 include，使 playbooks 更加灵活。我们学习了角色以及它们如何帮助您保持系统一致，还学会了如何正确组织 Ansible 仓库，充分利用 Ansible 和 Git。接着，我们讨论了执行策略和 Jinja 过滤器，以实现更加灵活的执行。

我们结束了本章对 Ansible Vault 的讲解，并提供了许多其他提示，以使您的 Ansible 执行更安全。

在下一章中，我们将看看如何使用 Ansible 来创建基础设施，更具体地说，如何在云提供商 AWS 和 DigitalOcean 上使用它。
