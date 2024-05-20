# 第五章：扩展到多个主机

在之前的章节中，我们在命令行中指定了主机。在只有一个主机要处理时，这样做效果很好，但是当管理多个服务器时，效果就不太好了。在本章中，我们将看到如何利用库存来管理多个服务器。此外，我们还将介绍主机变量和组变量等主题，以便轻松快速地设置类似但不同的主机。我们将讨论**Ansible**中的循环，它允许您减少编写的代码量，同时使代码更易读。

在本章中，我们将涵盖以下主题：

+   使用库存文件

+   使用变量

# 技术要求

您可以从本书的 GitHub 仓库下载所有文件，网址为[`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter03`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition/tree/master/Chapter03)。

# 使用库存文件

**库存文件**是 Ansible 的真相之源（还有一个名为**动态库存**的高级概念，我们稍后会介绍）。它遵循 **INI** 格式，并告诉 Ansible 用户提供的远程主机是否真实。

Ansible 可以并行运行其任务对多个主机进行操作。为了做到这一点，您可以直接将主机列表传递给 Ansible，使用库存文件。对于这样的并行执行，Ansible 允许您在库存文件中对主机进行分组；文件将组的名称传递给 Ansible。Ansible 将在库存文件中搜索该组，并对该组中列出的所有主机运行其任务。

您可以使用 `-i` 或 `--inventory-file` 选项将库存文件传递给 Ansible，后跟文件路径。如果您未明确指定任何库存文件给 Ansible，则 Ansible 将从 `ansible.cfg` 的 `host_file` 参数的默认路径获取，默认路径为 `/etc/ansible/hosts`。

使用 `-i` 参数时，如果值是一个列表（至少包含一个逗号），它将被用作库存列表，而如果变量是字符串，则将其用作库存文件路径。

# 基本库存文件

在深入概念之前，让我们先看一下一个称为`hosts`的基本库存文件，我们可以在这个文件中使用，而不是在之前的示例中使用的列表：

```
test01.fale.io
```

Ansible 可以在库存文件中使用 FQDN 或 IP 地址。

现在，我们可以执行与上一章相同的操作，调整 Ansible 命令参数。

例如，要安装 Web 服务器，我们使用了这个命令：

```
$ ansible-playbook -i test01.fale.io, webserver.yaml 
```

相反，我们可以使用以下内容：

```
$ ansible-playbook -i hosts webserver.yaml 
```

正如您所看到的，我们已经用库存文件名替换了主机列表。

# 库存文件中的组

当我们遇到更复杂的情况时，清单文件的优势就会显现出来。假设我们的网站变得更加复杂，现在我们需要一个更复杂的环境。在我们的示例中，我们的网站将需要一个 MySQL 数据库。此外，我们决定有两台 web 服务器。在这种情况下，根据我们的基础设施中的角色对不同的机器进行分组是有意义的。

Ansible 允许我们创建一个类似 INI 文件的文件，其中包含组（INI 部分）和主机。这是我们的主机文件将要更改为的内容：

```
[webserver] 
ws01.fale.io 
ws02.fale.io 

[database] 
db01.fale.io 
```

现在我们可以指示播放书只在某个组中的主机上运行。在上一章中，我们为我们的网站示例创建了三个不同的播放书：

+   `firstrun.yaml` 是通用的，必须在每台机器上运行。

+   `common_tasks.yaml` 是通用的，必须在每台机器上运行。

+   `webserver.yaml` 是特定于 web 服务器的，因此不应在任何其他机器上运行。

由于唯一特定于服务器组的文件是 `webserver.yaml`，所以我们只需要更改它。为此，让我们打开 `webserver.yaml` 文件并将内容从 **`- hosts: all`** 更改为 `- hosts: webserver`。

只有这三个播放书，我们无法继续创建我们的环境，有三个服务器。因为我们还没有一个设置数据库的播放书（我们将在下一章中看到），我们将完全为两台 web 服务器（`ws01.fale.io` 和 `ws02.fale.io`）提供服务，并且，对于数据库服务器，我们只提供基本系统。

在运行 Ansible 播放书之前，我们需要为环境提供支持。为此，请创建以下 vagrant 文件：

```
Vagrant.configure("2") do |config|
  config.vm.define "ws01" do |ws01|
    ws01.vm.hostname = "ws01.fale.io"
  end
  config.vm.define "ws02" do |ws02|
    ws02.vm.hostname = "ws02.fale.io"
  end
  config.vm.define "db01" do |db01|
    db01.vm.hostname = "db01.fale.io"
  end
  config.vm.box = "centos/7"
end
```

通过运行 `vagrant up`，Vagrant 将为我们生成整个环境。在一段时间后，Vagrant 在 shell 中输出一些内容后，应该会还给你命令提示符。当这种情况发生时，请检查最后几行是否有错误，以确保一切如预期般进行。

现在我们已经为环境提供了支持，我们可以继续执行 `firstrun` 播放书，这将确保我们的 Ansible 用户存在并且具有正确的 SSH 密钥设置。为此，我们可以使用以下命令运行它：

```
$ ansible-playbook -i hosts firstrun.yaml 
```

以下将是结果。完整的输出文件可在 GitHub 上找到：

```
PLAY [all] ***********************************************************

TASK [Gathering Facts] ***********************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]
ok: [db01.fale.io]

TASK [Ensure ansible user exists] ************************************
changed: [ws02.fale.io]
changed: [db01.fale.io]
changed: [ws01.fale.io]
...
```

正如你所看到的，输出与我们收到的单个主机非常相似，但在每个步骤中每个主机都有一行。在这种情况下，所有机器处于相同的状态，并且执行了相同的步骤，所以我们看到它们都表现得一样，但是在更复杂的场景中，你可以看到不同的机器在同一步骤上返回不同的状态。我们也可以执行另外两个播放书，结果类似。

# 清单文件中的正则表达式

当你有大量服务器时，给它们起一个可预测的名字是常见和有用的，例如，将所有网络服务器命名为`wsXY`或`webXY`，或者将数据库服务器命名为`dbXY`。如果你这样做，你可以减少主机文件中的行数，增加其可读性。例如，我们的主机文件可以简化如下：

```
[webserver] 
ws[01:02].fale.io 

[database] 
db01.fale.io 
```

在这个例子中，我们使用了`[01:02]`，它将匹配第一个数字（在我们的例子中是`01`）和最后一个数字（在我们的例子中是`02`）之间的所有出现。在我们的案例中，收益并不巨大，但如果你有 40 台网络服务器，你可以从主机文件中减少 39 行。

在这一部分中，我们已经看到了如何创建清单文件，如何向 Ansible 清单添加组，如何利用范围来加快清单创建过程，并如何针对清单运行 Ansible playbook。现在我们将看到如何在清单中设置变量以及如何在我们的 playbooks 中使用它们。

# 使用变量

Ansible 允许你以多种方式定义变量，从 playbook 中的变量文件，通过使用`-e`/`--extra-vars`选项从 Ansible 命令传递它。你也可以通过将其传递给清单文件来定义变量。你可以在清单文件中的每个主机，整个组或在清单文件所在目录中创建一个变量文件来定义变量。

# 主机变量

可以为特定主机声明变量，在主机文件中声明它们。例如，我们可能希望为我们的网络服务器指定不同的引擎。假设一个需要回复到一个特定的域，而另一个需要回复到不同的域名。在这种情况下，我们会在以下主机文件中执行：

```
[webserver] 
ws01.fale.io domainname=example1.fale.io 
ws02.fale.io domainname=example2.fale.io 

[database] 
db01.fale.io 
```

每次我们使用此清单执行 playbook 时，Ansible 将首先读取清单文件，并将根据每个主机分配`domainname`变量的值。这样，所有在网络服务器上运行的 playbook 将能够引用`domainname`变量。

# 组变量

有些情况下，你可能想要设置一个整个组都有效的变量。假设我们想要声明变量`https_enabled`为`True`，并且其值必须对所有网络服务器都相等。在这种情况下，我们可以创建一个`[webserver:vars]`部分，因此我们将使用以下主机文件：

```
[webserver] 
ws01.fale.io 
ws02.fale.io 

[webserver:vars] 
https_enabled=True 

[database] 
db01.fale.io 
```

请记住，如果相同的变量在两个空间中都声明了，主机变量将覆盖组变量。

# 变量文件

有时，你需要为每个主机和组声明大量的变量，主机文件变得难以阅读。在这些情况下，你可以将变量移动到特定的文件中。对于主机级别的变量，你需要在`host_vars`文件夹中创建一个与你的主机同名的文件，而对于组变量，你需要使用组名作为文件名，并将它们放置在`group_vars`文件夹中。

因此，如果我们想要复制先前基于主机变量使用文件的示例，我们需要创建 `host_vars/ws01.fale.io` 文件，内容如下：

```
domainname=example1.fale.io 
```

然后我们创建 `host_vars/ws02.fale.io` 文件，内容如下：

```
domainname=example2.fale.io 
```

而如果我们想要复制基于组变量的示例，我们将需要具有以下内容的 `group_vars/webserver` 文件：

```
https_enabled=True 
```

清单变量遵循一种层次结构；在其顶部是公共变量文件（我们在上一节 *使用清单文件* 中讨论过），它将覆盖任何主机变量、组变量和清单变量文件。接下来是主机变量，它将覆盖组变量；最后，组变量将覆盖清单变量文件。

# 用清单文件覆盖配置参数

你可以直接通过清单文件覆盖一些 Ansible 的配置参数。这些配置参数将覆盖所有其他通过 `ansible.cfg`、环境变量或在 playbooks 中设置的参数。传递给 `ansible-playbook/ansible` 命令的变量优先级高于清单文件中设置的任何其他变量。

以下是一些可以从清单文件覆盖的参数列表：

+   `ansible_user`：该参数用于覆盖与远程主机通信所使用的用户。有时，某台机器需要不同的用户；在这种情况下，这个变量会帮助你。例如，如果你是从 *ansible* 用户运行 Ansible，但在远程机器上需要连接到 *automation* 用户，设置 `ansible_user=automation` 就会实现这一点。

+   `ansible_port`：该参数将用用户指定的端口覆盖默认的 SSH 端口。有时，系统管理员选择在非标准端口上运行 SSH。在这种情况下，你需要通知 Ansible 进行更改。如果在你的环境中 SSH 端口是 22022 而不是 22，你将需要使用 `ansible_port=22022`。

+   `ansible_host`：该参数用于覆盖别名的主机。如果你想通过 DNS 名称（即：`ws01.fale.io`）连接到 `10.0.0.3` 机器，但由于某些原因 DNS 未能正确解析主机，你可以通过设置 `ansible_host=10.0.0.3` 变量，强制 Ansible 使用该 IP 而不是 DNS 解析出的 IP。

+   `ansible_connection`：这指定了到远程主机的连接类型。值可以是 SSH、Paramiko 或本地。即使 Ansible 可以使用其 SSH 守护程序连接到本地机器，但这会浪费大量资源。在这些情况下，你可以指定 `ansible_connection=local`，这样 Ansible 将打开一个标准 shell 而不是 SSH。

+   `ansible_private_key_file`：此参数将覆盖用于 SSH 的私钥；如果您想为特定主机使用特定密钥，则这将非常有用。常见的用例是，如果您的主机分布在多个数据中心、多个 AWS 区域或不同类型的应用程序中。在这种情况下，私钥可能不同。

+   `ansible__type`：默认情况下，Ansible 使用 `sh` shell；您可以使用 `ansible_shell_type` 参数覆盖此行为。将其更改为 `csh`、`ksh` 等将使 Ansible 使用该 shell 的命令。如果您需要执行一些 `csh` 或 `ksh` 脚本，并且立即处理它们会很昂贵，那么这可能会有所帮助。

# 使用动态清单

有些环境中，你有一个系统可以自动创建和销毁机器。我们将在第五章，*云之旅*中看到如何使用 Ansible 完成这个任务。在这种环境中，机器列表变化非常快，维护主机文件变得复杂。在这种情况下，我们可以使用动态清单来解决这个问题。

动态清单背后的想法是 Ansible 不会读取主机文件，而是执行一个返回主机列表的脚本，并以 JSON 格式返回给 Ansible。这允许您直接查询您的云提供商，询问在任何给定时刻运行的整个基础设施中的机器列表。

通过 Ansible，已经提供了大多数常见云提供商的许多脚本，可以在[`github.com/ansible/ansible/tree/devel/contrib/inventory`](https://github.com/ansible/ansible/tree/devel/contrib/inventory)找到，但如果您有不同的需求，也可以创建自定义脚本。Ansible 清单脚本可以用任何语言编写，但出于一致性考虑，动态清单脚本应使用 Python 编写。请记住，这些脚本需要直接可执行，因此请记得为它们设置可执行标志（`chmod + x inventory.py`）。

接下来，我们将查看可以从官方 Ansible 仓库下载的 Amazon Web Services 和 DigitalOcean 脚本。

# Amazon Web Services

要允许 Ansible 从**Amazon Web Services**（**AWS**）收集关于您的 EC2 实例的数据，您需要从 Ansible 的 GitHub 仓库下载以下两个文件：[`github.com/ansible/ansible`](https://github.com/ansible/ansible)。

+   `ec2.py` 清单脚本

+   `ec2.ini` 文件包含了您的 EC2 清单脚本的配置。

Ansible 使用**Boto**，AWS Python SDK，通过 API 与 AWS 进行通信。为了允许此通信，您需要导出 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY` 变量。

你可以以两种方式使用清单：

+   通过 `-i` 选项将其直接传递给 `ansible-playbook` 命令，并将 `ec2.ini` 文件复制到您运行 Ansible 命令的当前目录。

+   将`ec2.py`文件复制到`/etc/ansible/hosts`，并使用`chmod +x`使其可执行，然后将`ec2.ini`文件复制到`/etc/ansible/ec2.ini`。

`ec2.py`文件将根据地区、可用区、标签等创建多个组。您可以通过运行`./ec2.py --list`来检查清单文件的内容。

让我们看一个使用 EC2 动态清单的示例 playbook，它将简单地 ping 我的帐户中的所有机器：

```
ansible -i ec2.py all -m ping
```

由于我们执行了 ping 模块，我们期望配置的帐户中可用的机器会回复我们。由于我当前帐户中只有一台带有 IP 地址 52.28.138.231 的 EC2 机器，我们可以期望它会回复，实际上我的帐户上的 EC2 回复如下：

```
52.28.138.231 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 
```

在上面的示例中，我们使用`ec2.py`脚本而不是静态清单文件，并使用`-i`选项和 ping 命令。

类似地，您可以使用这些清单脚本执行各种类型的操作。例如，您可以将它们与您的部署脚本集成，以找出单个区域中的所有节点，并在执行区域部署时部署到这些节点（一个区域表示一个数据中心）在 AWS 中。

如果您只想知道云中的 web 服务器是什么，并且您已经使用某种约定对它们进行了标记，您可以通过使用动态清单脚本来过滤掉标记来实现。此外，如果您有未涵盖的特殊情况，您可以增强它以提供所需的节点集以 JSON 格式，并从 playbooks 中对这些节点进行操作。如果您正在使用数据库来管理您的清单，您的清单脚本可以查询数据库并转储 JSON。它甚至可以与您的云同步，并定期更新您的数据库。

# DigitalOcean

正如我们在[`github.com/ansible/ansible/tree/devel/contrib/inventory`](https://github.com/ansible/ansible/tree/devel/contrib/inventory)中使用 EC2 文件从 AWS 中提取数据一样，我们可以对 DigitalOcean 执行相同的操作。唯一的区别是我们必须获取`digital_ocean.ini`和`digital_ocean.py`文件。

与以前一样，如果需要，我们需要调整`digital_ocean.ini`选项，并将 Python 文件设置为可执行。你唯一可能需要更改的选项是`api_token`。

现在我们可以尝试 ping 我在 DigitalOcean 上预配的两台机器，如下所示：

```
ansible -i digital_ocean.py all -m ping 
```

正如预期的那样，我帐户中的两个 droplets 响应如下：

```
188.166.150.79 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 
46.101.77.55 | SUCCESS => { 
    "changed": false, 
    "ping": "pong" 
} 
```

我们现在已经看到从许多不同的云提供商检索数据是多么容易。

# 在 Ansible 中使用迭代器

您可能已经注意到，到目前为止，我们从未使用过循环，因此每次我们必须执行多个相似的操作时，我们都会多次编写代码。其中一个示例是`webserver.yaml`代码。

实际上，这是`webserver.yaml`文件的最后一部分：

```
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

如你所见，`webserver.yaml`代码的最后两个块执行的操作非常相似：确保防火墙的某个端口是打开的。

# 使用标准迭代 - with_items

重复的代码本身并非问题，但它不具备可扩展性。

Ansible 允许我们使用迭代来提高代码的清晰度和可维护性。

为了改进上述代码，我们可以使用简单的迭代：`with_items`。

这使我们能够对项目列表进行迭代。在每次迭代中，列表的指定项目将在 item 变量中可用。这使我们能够在单个块中执行多个类似的操作。

因此，我们可以将`webserver.yaml`代码的最后一部分更改为以下内容：

```
    - name: Ensure HTTP and HTTPS can pass the firewall 
      firewalld: 
        service: '{{ item }}' 
        state: enabled 
        permanent: True 
        immediate: True 
      become: True
      with_items:
        - http
        - https
```

我们可以按照以下方式执行它：

```
ansible-playbook -i hosts webserver.yaml
```

我们收到以下内容：

```
PLAY [all] *********************************************************

TASK [Gathering Facts] *********************************************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure the HTTPd package is installed] ***********************
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure the HTTPd service is enabled and running] *************
ok: [ws01.fale.io]
ok: [ws02.fale.io]

TASK [Ensure HTTP and HTTPS can pass the firewall] *****************
ok: [ws01.fale.io] (item=http)
ok: [ws02.fale.io] (item=http)
ok: [ws01.fale.io] (item=https)
ok: [ws02.fale.io] (item=https)

PLAY RECAP *********************************************************
ws01.fale.io                : ok=5 changed=0 unreachable=0 failed=0 
ws02.fale.io                : ok=5 changed=0 unreachable=0 failed=0
```

如您所见，输出与先前执行略有不同。事实上，在具有循环操作的行上，我们可以看到在`Ensure HTTP and HTTPS can pass the firewall`块的特定迭代中处理的`item`。

我们现在已经看到我们可以对项目列表进行迭代，但是 Ansible 还允许我们进行其他类型的迭代。

# 使用嵌套循环 - with_nested

有些情况下，您必须对列表中的所有元素与其他列表中的所有项进行迭代（笛卡尔积）。一个非常常见的情况是当您必须在多个路径中创建多个文件夹时。在我们的例子中，我们将在用户`alice`和`bob`的主目录中创建文件夹`mail`和`public_html`。

我们可以使用`with_nested.yaml`文件中的以下代码片段来实现；完整的代码可在 GitHub 上找到：

```

- hosts: all 
  remote_user: ansible
  vars: 
    users: 
      - alice 
      - bob 
    folders: 
      - mail 
      - public_html 
  tasks: 
    - name: Ensure the users exist 
      user: 
        name: '{{ item }}' 
      become: True 
      with_items: 
        - '{{ users }}' 
    ...
```

使用以下方式运行此命令：

```
ansible-playbook -i hosts with_nested.yaml 
```

我们收到以下结果。完整的输出文件可在 GitHub 上找到：

```
PLAY [all] *******************************************************

TASK [Gathering Facts] *******************************************
ok: [db01.fale.io]
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure the users exist] ************************************
changed: [db01.fale.io] => (item=alice)
changed: [ws02.fale.io] => (item=alice)
changed: [ws01.fale.io] => (item=alice)
changed: [db01.fale.io] => (item=bob)
changed: [ws02.fale.io] => (item=bob)
changed: [ws01.fale.io] => (item=bob)
...
```

如您所见，Ansible 在所有目标机器上创建了用户 alice 和 bob，并且还为这两个用户在所有机器上的文件夹`$HOME/mail`和`$HOME/public_html`。

# 文件通配符循环 - with_fileglobs

有时，我们希望对某个特定文件夹中的每个文件执行操作。如果你想要从一个文件夹复制多个文件到另一个文件夹中，这可能会很方便。为此，你可以创建一个名为`with_fileglobs.yaml`的文件，其中包含以下代码：

```
--- 
- hosts: all 
  remote_user: ansible
  tasks: 
    - name: Ensure the folder /tmp/iproute2 is present 
      file: 
        dest: '/tmp/iproute2' 
        state: directory 
      become: True 
    - name: Copy files that start with rt to the tmp folder 
      copy: 
        src: '{{ item }}' 
        dest: '/tmp/iproute2' 
        remote_src: True 
      become: True 
      with_fileglob: 
        - '/etc/iproute2/rt_*' 
```

我们可以按照以下方式执行它：

```
ansible-playbook -i hosts with_fileglobs.yaml 
```

这导致了以下输出。完整的输出文件可在 GitHub 上找到。

```
PLAY [all] *****************************************************

TASK [Gathering Facts] *****************************************
ok: [db01.fale.io]
ok: [ws02.fale.io]
ok: [ws01.fale.io]

TASK [Ensure the folder /tmp/iproute2 is present] **************
changed: [ws02.fale.io]
changed: [ws01.fale.io]
changed: [db01.fale.io]

TASK [Copy files that start with rt to the tmp folder] *********
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_realms)
changed: [db01.fale.io] => (item=/etc/iproute2/rt_realms)
changed: [ws02.fale.io] => (item=/etc/iproute2/rt_realms)
changed: [ws01.fale.io] => (item=/etc/iproute2/rt_protos)
...
```

至于我们的目标，我们已经创建了`/tmp/iproute2`文件夹，并用`/etc/iproute2`文件夹中文件的副本填充了它。这种模式经常用于创建配置的备份。

# 使用整数循环 - with_sequence

许多时候，你需要对整数进行迭代。一个例子是创建十个名为`fileXY`的文件夹，其中`X`和`Y`是从`1`到`10`的连续数字。为此，我们可以创建一个名为`with_sequence.yaml`的文件，其中包含以下代码：

```
--- 
- hosts: all 
  remote_user: ansible 
  tasks: 
  - name: Create the folders /tmp/dirXY with XY from 1 to 10 
    file: 
      dest: '/tmp/dir{{ item }}' 
      state: directory 
    with_sequence: start=1 end=10 
    become: True 
```

与大多数 Ansible 命令不同，我们可以在对象上使用单行符号和标准 YAML 多行符号，`with_sequence` 只支持单行符号。

然后，我们可以使用以下命令执行它：

```
ansible-playbook -i hosts with_sequence.yaml 
```

我们将收到以下输出：

```
PLAY [all] *****************************************************

TASK [Gathering Facts] *****************************************
ok: [ws02.fale.io]
ok: [ws01.fale.io]
ok: [db01.fale.io]

TASK [Create the folders /tmp/dirXY with XY from 1 to 10] ******
changed: [ws01.fale.io] => (item=1)
changed: [db01.fale.io] => (item=1)
changed: [ws02.fale.io] => (item=1)
changed: [ws01.fale.io] => (item=2)
changed: [db01.fale.io] => (item=2)
changed: [ws02.fale.io] => (item=2)
changed: [ws01.fale.io] => (item=3)
...
```

Ansible 支持更多类型的循环，但由于它们的使用要少得多，你可以直接参考官方文档了解循环：[`docs.ansible.com/ansible/playbooks_loops.html`](http://docs.ansible.com/ansible/playbooks_loops.html)。

# 摘要

在本章中，我们探讨了大量的概念，将帮助您将基础设施扩展到单个节点之外。我们从用于指示 Ansible 关于我们机器的清单文件开始，然后我们介绍了如何在运行相同命令的多个异构主机上拥有主机特定和组特定的变量。然后，我们转向由某些其他系统（通常是云提供商）直接填充的动态清单。最后，我们分析了 Ansible playbook 中的多种迭代方式。

在下一章中，我们将以更合理的方式结构化我们的 Ansible 文件，以确保最大的可读性。为此，我们引入了角色，进一步简化了复杂环境的管理。
