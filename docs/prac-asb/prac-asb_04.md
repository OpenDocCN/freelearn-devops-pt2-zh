# 第三章：定义您的清单

正如我们在前两章中已经讨论过的，除非告诉它负责哪些主机，否则 Ansible 无法做任何事情。这当然是合乎逻辑的——无论自动化工具有多容易使用和设置，你都不希望它简单地控制网络上的每个设备。因此，至少，您必须告诉 Ansible 它将自动化任务的主机是哪些，这在最基本的术语中就是清单。

然而，清单中有很多东西不仅仅是自动化目标的列表。Ansible 清单可以以几种格式提供；它们可以是静态的或动态的，并且它们可以包含定义 Ansible 与每个主机（或主机组）交互的重要变量。因此，它们值得有一个章节来讨论，而在本章中，我们将对清单进行实际探索，以及如何在使用 Ansible 自动化基础设施时充分利用它们。

在本章中，我们将涵盖以下主题：

+   创建清单文件并添加主机

+   生成动态清单文件

+   使用模式进行特殊主机管理

# 技术要求

本章假设您已经按照第一章 *开始使用 Ansible*中详细说明的设置了控制主机，并且您正在使用最新版本——本章的示例是使用 Ansible 2.9 进行测试的。本章还假设您至少有一个额外的主机进行测试，并且最好是基于 Linux 的。尽管本章将给出主机名的具体示例，但您可以自由地用自己的主机名和/或 IP 地址替换它们，如何做到这一点的详细信息将在适当的地方提供。

本章的代码包在此处可用：[`github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%203`](https://github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%203)。

# 创建清单文件并添加主机

每当您在 Ansible 中看到“创建清单”的参考时，通常可以安全地假定它是一个静态清单。Ansible 支持两种类型的清单——静态和动态，我们将在本章后面讨论后者。静态清单本质上是静态的；除非有人去手动编辑它们，否则它们是不会改变的。当您开始测试 Ansible 时，这是一个很好的选择，因为它为您提供了一个非常快速和简单的方法来快速启动和运行。即使在小型封闭环境中，静态清单也是管理环境的好方法，特别是在基础设施的更改不频繁时。

大多数 Ansible 安装将在`/etc/ansible/hosts`中寻找默认的清单文件（尽管这个路径在 Ansible 配置文件中是可配置的，如第二章 *理解 Ansible 的基础知识*中所讨论的）。您可以填充此文件，或为每个 playbook 运行提供自己的清单，通常可以看到清单与 playbooks 一起提供。毕竟，很少有“一刀切”的 playbook，尽管您可以使用组来细分您的清单（稍后会详细介绍），但通常提供一个较小的静态清单文件与特定的 playbook 一起提供也同样容易。正如您在本书的前几章中所看到的，大多数 Ansible 命令在不使用默认值时使用`-i`标志来指定清单文件的位置。假设情况下，这可能看起来像以下示例：

```
$ ansible -i /home/cloud-user/inventory all -m ping
```

您可能会遇到的大多数静态清单文件都是以 INI 格式创建的，尽管重要的是要注意其他格式也是可能的。在 INI 格式的文件之后，您将发现的最常见格式是 YAML 格式 - 您可以在这里找到更多关于您可以使用的清单文件类型的详细信息：[`docs.ansible.com/ansible/latest/user_guide/intro_inventory.html`](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)。

在本章中，我们将为您提供一些 INI 和 YAML 格式的清单文件示例，供您考虑，因为您必须对两者都有所了解。就我个人而言，我已经使用 Ansible 工作了很多年，使用过 INI 格式的文件或动态清单，但他们说知识就是力量，所以了解一下这两种格式也无妨。

让我们从创建一个静态清单文件开始。这个清单文件将与默认清单分开。

在`/etc/ansible/my_inventory`中创建一个清单文件，使用以下格式化的 INI 代码：

```
target1.example.com ansible_host=192.168.81.142 ansible_port=3333  target2.example.com ansible_port=3333 ansible_user=danieloh  target3.example.com ansible_host=192.168.81.143 ansible_port=5555
```

清单主机之间的空行不是必需的 - 它们只是为了使本书中的清单更易读而插入的。这个清单文件非常简单，不包括任何分组；但是，在引用清单时，您仍然可以使用特殊的`all`组来引用所有主机，这个组是隐式定义的，无论您如何格式化和划分您的清单文件。

上述文件中的每一行都包含一个清单主机。第一列包含 Ansible 将使用的清单主机名（并且可以通过我们在第二章中讨论的`inventory_hostname`魔术变量来访问）。之后同一行上的所有参数都是分配给主机的变量。这些可以是用户定义的变量或特殊的 Ansible 变量，就像我们在这里设置的一样。

有许多这样的变量，但前面的例子特别包括以下内容：

+   `ansible_host`：如果无法直接访问清单主机名 - 例如，因为它不在 DNS 中，那么这个变量包含 Ansible 将连接的主机名或 IP 地址。

+   `ansible_port`：默认情况下，Ansible 尝试通过 SSH 的 22 端口进行所有通信 - 如果您在另一个端口上运行 SSH 守护程序，可以使用此变量告诉 Ansible。

+   `ansible_user`：默认情况下，Ansible 将尝试使用您从中运行 Ansible 命令的当前用户帐户连接到远程主机 - 您可以以多种方式覆盖这一点，其中之一就是这个。

因此，前面的三个主机可以总结如下：

+   `target1.example.com`主机应该使用`192.168.81.142`IP 地址连接，端口为`3333`。

+   `target2.example.com`主机也应该连接到端口`3333`，但这次使用`danieloh`用户，而不是运行 Ansible 命令的帐户。

+   `target3.example.com`主机应该使用`192.168.81.143`IP 地址连接，端口为`5555`。

通过这种方式，即使没有进一步的构造，您也可以开始看到静态的 INI 格式的清单的强大之处。

现在，如果您想要创建与前面完全相同的清单，但这次以 YAML 格式进行格式化，您可以指定如下：

```
---
ungrouped:
  hosts:
    target1.example.com:
      ansible_host: 192.168.81.142
      ansible_port: 3333
    target2.example.com:
      ansible_port: 3333
      ansible_user: danieloh
    target3.example.com:
      ansible_host: 192.168.81.143
      ansible_port: 5555
```

您可能会遇到包含参数如`ansible_ssh_port`、`ansible_ssh_host`和`ansible_ssh_user`的清单文件示例 - 这些变量名称（以及类似的其他变量）在 2.0 版本之前的 Ansible 版本中使用。对许多这些变量已经保持了向后兼容性，但在可能的情况下，您应该更新它们，因为这种兼容性可能在将来的某个时候被移除。

现在，如果您在 Ansible 中运行上述清单，使用一个简单的`shell`命令，结果将如下所示：

```
$ ansible -i /etc/ansible/my_inventory.yaml all -m shell -a 'echo hello-yaml' -f 5
target1.example.com | CHANGED | rc=0 >>
hello-yaml
target2.example.com | CHANGED | rc=0 >>
hello-yaml
target3.example.com | CHANGED | rc=0 >>
hello-yaml
```

这涵盖了创建一个简单静态清单文件的基础知识。现在让我们通过在本章的下一部分将主机组添加到清单中来扩展这一点。

# 使用主机组

很少有一个 playbook 适用于整个基础架构，尽管很容易告诉 Ansible 为不同的 playbook 使用备用清单，但这可能会变得非常混乱，非常快速，潜在地在你的网络中散布了数百个小清单文件。你可以想象这会变得多么难以管理，而 Ansible 的目的是使事情更容易管理，而不是相反。这个问题的一个可能简单的解决方案是开始在你的清单中添加组。

假设你有一个简单的三层 Web 架构，每层都有多个主机以实现高可用性和/或负载平衡。这种架构中的三个层可能是以下内容：

+   前端服务器

+   应用服务器

+   数据库服务器

有了这个架构，让我们开始创建一个清单，再次混合使用 YAML 和 INI 格式，以便你在两种格式中都有经验。为了使示例清晰简洁，我们假设你可以使用它们的**完全限定域名**（**FQDNs**）访问所有服务器，因此不会在这些清单文件中添加任何主机变量。当然，没有什么能阻止你这样做，每个示例都是不同的。

首先，让我们使用 INI 格式为三层前端创建清单。我们将称此文件为`hostsgroups-ini`，此文件的内容应该如下所示：

```
loadbalancer.example.com

[frontends]
frt01.example.com
frt02.example.com

[apps]
app01.example.com
app02.example.com

[databases]
dbms01.example.com
dbms02.example.com
```

在前面的清单中，我们创建了三个名为`frontends`、`apps`和`databases`的组。请注意，在 INI 格式的清单中，组名放在方括号内。在每个组名下面是属于每个组的服务器名，因此前面的示例显示了每个组中的两个服务器。请注意顶部的异常值`loadbalancer.example.com` - 这个主机不属于任何组。所有未分组的主机必须放在 INI 格式文件的顶部。

在我们进一步进行之前，值得注意的是，清单也可以包含组的组，这对于通过不同的部门处理某些任务非常有用。前面的清单是独立的，但如果我们的前端服务器是建立在 Ubuntu 上，而应用和数据库服务器是建立在 CentOS 上呢？在处理这些主机的方式上会有一些根本的不同 - 例如，我们可能会在 Ubuntu 上使用`apt`模块来管理软件包，在 CentOS 上使用`yum`模块。

当然，我们可以使用从每个主机收集的事实来处理这种情况，因为这些事实将包含操作系统的详细信息。我们还可以创建清单的新版本，如下所示：

```
loadbalancer.example.com

[frontends]
frt01.example.com
frt02.example.com

[apps]
app01.example.com
app02.example.com

[databases]
dbms01.example.com
dbms02.example.com

[centos:children]
apps
databases

[ubuntu:children]
frontends
```

在组定义中使用`children`关键字（在方括号内），我们可以创建组的组；因此，我们可以进行巧妙的分组，以帮助我们的 playbook 设计，而无需多次指定每个主机。

INI 格式中的这种结构相当易读，但当转换为 YAML 格式时需要一些时间来适应。下面列出的代码显示了前面清单的 YAML 版本 - 就 Ansible 而言，两者是相同的，但你可以决定你更喜欢使用哪种格式：

```
all:
  hosts:
    loadbalancer.example.com:
  children:
    centos:
      children:
        apps:
          hosts:
            app01.example.com:
            app02.example.com:
        databases:
          hosts:
            dbms01.example.com:
            dbms02.example.com:
    ubuntu:
      children:
        frontends:
          hosts:
            frt01.example.com:
            frt02.example.com:
```

你可以看到`children`关键字仍然在 YAML 格式的清单中使用，但现在结构比 INI 格式更加分层。缩进可能更容易让你理解，但请注意主机最终是在相当高层次的缩进下定义的 - 这种格式可能更难扩展，取决于你希望采用的方法。

当你想要使用前面清单中的任何组时，你可以在你的 playbook 或命令行中简单地引用它。例如，在上一节中我们运行的，我们可以使用以下命令：

```
$ ansible -i /etc/ansible/my_inventory.yaml all -m shell -a 'echo hello-yaml' -f 5
```

请注意该行中间的`all`关键字。这是所有库存中都隐含的特殊`all`组，并且在你之前的 YAML 示例中明确提到。如果我们想运行相同的命令，但这次只在之前的 YAML 库存中的`centos`组主机上运行，我们将运行这个命令的变体：

```
$ ansible -i hostgroups-yml centos -m shell -a 'echo hello-yaml' -f 5
app01.example.com | CHANGED | rc=0 >>
hello-yaml
app02.example.com | CHANGED | rc=0 >>
hello-yaml
dbms01.example.com | CHANGED | rc=0 >>
hello-yaml
dbms02.example.com | CHANGED | rc=0 >>
hello-yaml 
```

正如你所看到的，这是一种管理库存并轻松运行命令的强大方式。创建多个组的可能性使生活变得简单和容易，特别是当你想在不同的服务器组上运行不同的任务时。

作为开发库存的一部分，值得注意的是，有一种快速的简写表示法，可以用来创建多个主机。假设你有 100 个应用服务器，所有的名称都是顺序的，如下所示：

```
[apps]
app01.example.com
app02.example.com
...
app99.example.com
app100.example.com
```

这是完全可能的，但手工创建将是乏味和容易出错的，并且会产生一些非常难以阅读和解释的库存。幸运的是，Ansible 提供了一种快速的简写表示法来实现这一点，以下库存片段实际上产生了一个与我们可以手动创建的相同的 100 个应用服务器的库存：

```
[apps]
app[01:100].prod.com
```

也可以使用字母范围以及数字范围——扩展我们的示例以添加一些缓存服务器，你可能会有以下内容：

```
[caches]
cache-[a:e].prod.com  
```

这与手动创建以下内容相同：

```
[caches]
cache-a.prod.com cache-b.prod.com
cache-c.prod.com
cache-d.prod.com
cache-e.prod.com 
```

现在我们已经完成了对各种静态库存格式的探索以及如何创建组（甚至是子组），让我们在下一节中扩展我们之前简要介绍的主机变量。

# 向库存添加主机和组变量

我们已经提到了主机变量——在本章的前面部分，当我们用它们来覆盖连接细节时，比如要连接的用户帐户、要连接的地址和要使用的端口。然而，你可以在 Ansible 和库存变量中做的事情远不止这些，重要的是要注意，它们不仅可以在主机级别定义，还可以在组级别定义，这再次为你提供了一些非常强大的方式来高效地管理你的基础设施。

让我们在之前的三层示例基础上继续建设，并假设我们需要为我们的两个前端服务器中的每一个设置两个变量。这些不是特殊的 Ansible 变量，而是完全由我们自己选择的变量，我们将在稍后运行对这台服务器的 playbook 中使用。假设这些变量如下：

+   `https_port`，定义了前端代理应该监听的端口

+   `lb_vip`，定义了前端服务器前面的负载均衡器的 FQDN

让我们看看这是如何完成的：

1.  我们可以简单地将这些添加到我们库存文件中`frontends`部分的每个主机中，就像我们之前用 Ansible 连接变量做的那样。在这种情况下，我们的 INI 格式的库存的一部分可能是这样的：

```
[frontends]
frt01.example.com https_port=8443 lb_vip=lb.example.com
frt02.example.com https_port=8443 lb_vip=lb.example.com
```

如果我们对这个库存运行一个临时命令，我们可以看到这两个变量的内容：

```
$ ansible -i hostvars1-hostgroups-ini frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
```

这已经按我们的期望工作了，但这种方法效率低下，因为你必须将相同的变量添加到每个主机。

1.  幸运的是，你可以将变量分配给主机组以及单独的主机。如果我们编辑前面的库存以实现这一点，`frontends`部分现在看起来像这样：

```
[frontends]
frt01.example.com
frt02.example.com

[frontends:vars]
https_port=8443
lb_vip=lb.example.com
```

请注意这种方式更易读？然而，如果我们对新组织的库存运行与之前相同的命令，我们会发现结果是一样的：

```
$ ansible -i groupvars1-hostgroups-ini frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
```

1.  有时候你会想要为单个主机使用主机变量，有时候组变量更相关。由你来决定哪个对你的情况更好；然而，请记住主机变量可以组合使用。值得注意的是主机变量会覆盖组变量，所以如果我们需要将连接端口更改为`8444`，我们可以这样做：

```
[frontends]
frt01.example.com https_port=8444
frt02.example.com

[frontends:vars]
https_port=8443
lb_vip=lb.example.com
```

现在，如果我们再次使用新的清单运行我们的临时命令，我们可以看到我们已经覆盖了一个主机上的变量：

```
$ ansible -i hostvars2-hostgroups-ini frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8444"
}
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
```

当然，当只有两个主机时，仅为一个主机执行此操作可能看起来有点无意义，但当你的清单中有数百个主机时，覆盖一个主机的这种方法突然变得非常有价值。

1.  为了完整起见，如果我们要将之前定义的主机变量添加到我们的清单的 YAML 版本中，`frontends`部分将如下所示（其余清单已被删除以节省空间）：

```
        frontends:
          hosts:
            frt01.example.com:
              https_port: 8444
            frt02.example.com:
          vars:
            https_port: 8443
            lb_vip: lb.example.com
```

运行与之前相同的临时命令，你会看到结果与我们的 INI 格式的清单相同：

```
$ ansible -i hostvars2-hostgroups-yml frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8444"
}
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
```

1.  到目前为止，我们已经介绍了几种向清单提供主机变量和组变量的方法；然而，还有一种方法值得特别提及，并且在你的清单变得更大更复杂时会变得有价值。

现在，我们的示例很小很简洁，只包含少数组和变量；然而，当你将其扩展到一个完整的服务器基础设施时，再次使用单个平面清单文件可能会变得难以管理。幸运的是，Ansible 也提供了解决方案。两个特别命名的目录`host_vars`和`group_vars`，如果它们存在于剧本目录中，将自动搜索适当的变量内容。我们可以通过使用这种特殊的目录结构重新创建前面的前端变量示例来测试这一点，而不是将变量放入清单文件中。

让我们首先为此目的创建一个新的目录结构：

```
$ mkdir vartree
$ cd vartree
```

1.  现在，在这个目录下，我们将为变量创建两个更多的目录：

```
$ mkdir host_vars group_vars
```

1.  现在，在`host_vars`目录下，我们将创建一个文件，文件名为需要代理设置的主机名，后面加上`.yml`（即`frt01.example.com.yml`）。这个文件应该包含以下内容：

```
---
https_port: 8444
```

1.  同样，在`group_vars`目录下，创建一个名为要分配变量的组的 YAML 文件（即`frontends.yml`），内容如下：

```
---
https_port: 8443
lb_vip: lb.example.com
```

1.  最后，我们将像以前一样创建我们的清单文件，只是它不包含变量：

```
loadbalancer.example.com

[frontends]
frt01.example.com
frt02.example.com

[apps]
app01.example.com
app02.example.com

[databases]
dbms01.example.com
dbms02.example.com
```

为了清晰起见，你的最终目录结构应该是这样的：

```
$  tree
.
├── group_vars
│   └── frontends.yml
├── host_vars
│   └── frt01.example.com.yml
└── inventory

2 directories, 3 files
```

1.  现在，让我们尝试运行我们熟悉的临时命令，看看会发生什么：

```
$ ansible -i inventory frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8444"
}
```

正如你所看到的，这与以前完全一样，而且在没有进一步的指示的情况下，Ansible 已经遍历了目录结构并摄取了所有的变量文件。

1.  如果你有数百个变量（或需要更精细的方法），你可以用主机和组的名字命名目录来替换 YAML 文件。现在，我们重新创建目录结构，但现在用目录代替：

```
$ tree
.
├── group_vars
│   └── frontends
│       ├── https_port.yml
│       └── lb_vip.yml
├── host_vars
│   └── frt01.example.com
│       └── main.yml
└── inventory
```

注意我们现在有了以`frontends`组和`frt01.example.com`主机命名的目录？在`frontends`目录中，我们将变量分成了两个文件，这对于在组中逻辑地组织变量尤其有用，特别是当你的剧本变得更大更复杂时。

这些文件本身只是我们之前的文件的一种改编：

```
$ cat host_vars/frt01.example.com/main.yml
---
https_port: 8444

$ cat group_vars/frontends/https_port.yml
---
https_port: 8443

$ cat group_vars/frontends/lb_vip.yml
---
lb_vip: lb.example.com
```

即使使用这种更细分的目录结构，运行临时命令的结果仍然是相同的：

```
$ ansible -i inventory frontends -m debug -a "msg=\"Connecting to {{ lb_vip }}, listening on {{ https_port }}\""
frt01.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8444"
}
frt02.example.com | SUCCESS => {
 "msg": "Connecting to lb.example.com, listening on 8443"
}
```

1.  在我们结束本章之前，还有一件事需要注意，即如果您在组级别和子组级别同时定义了相同的变量，则子组级别的变量优先。这并不像听起来那么明显。考虑我们之前的清单，我们在其中使用子组来区分 CentOS 和 Ubuntu 主机——如果我们在`ubuntu`子组和`frontends`组（`ubuntu`组的**子组**）中都添加了同名的变量，结果会是什么？清单将如下所示：

```
loadbalancer.example.com

[frontends]
frt01.example.com
frt02.example.com

[frontends:vars]
testvar=childgroup

[apps]
app01.example.com
app02.example.com

[databases]
dbms01.example.com
dbms02.example.com

[centos:children]
apps
databases

[ubuntu:children]
frontends

[ubuntu:vars]
testvar=group
```

现在，让我们运行一个临时命令，看看`testvar`的实际设置值是多少：

```
$ ansible -i hostgroups-children-vars-ini ubuntu -m debug -a "var=testvar"
frt01.example.com | SUCCESS => {
 "testvar": "childgroup"
}
frt02.example.com | SUCCESS => {
 "testvar": "childgroup"
}
```

需要注意的是，在这个清单中，`frontends`组是`ubuntu`组的子组（因此，组定义是`[ubuntu:children]`），因此在这种情况下，我们在`frontends`组级别设置的变量值会胜出。

到目前为止，您应该已经对如何使用静态清单文件有了相当好的了解。然而，没有查看动态清单的 Ansible 清单功能是完整的，我们将在下一节中做到这一点。

# 生成动态清单文件

在云计算和基础设施即代码的今天，您可能希望自动化的主机每天甚至每小时都会发生变化！保持静态的 Ansible 清单最新可能会成为一项全职工作，在许多大规模的场景中，因此，尝试在持续基础上使用静态清单变得不切实际。

这就是 Ansible 的动态清单支持发挥作用的地方。简而言之，Ansible 可以从几乎任何可执行文件中收集其清单数据（尽管您会发现大多数动态清单都是用 Python 编写的）——唯一的要求是可执行文件以指定的 JSON 格式返回清单数据。如果愿意，您可以自己创建清单脚本，但值得庆幸的是，已经有许多可供您使用的脚本，涵盖了许多潜在的清单来源，包括 Amazon EC2、Microsoft Azure、Red Hat Satellite、LDAP 目录等等。

在撰写书籍时，很难确定要使用哪个动态清单脚本作为示例，因为并不是每个人都有一个可以自由使用来进行测试的 Amazon EC2 帐户（例如）。因此，我们将以 Cobbler 配置系统作为示例，因为这是免费提供的，并且在 CentOS 系统上很容易部署。对于感兴趣的人来说，Cobbler 是一个用于动态配置和构建 Linux 系统的系统，它可以处理包括 DNS、DHCP、PXE 引导等在内的所有方面。因此，如果您要使用它来配置基础架构中的虚拟或物理机器，那么使用它作为清单来源也是有道理的，因为 Cobbler 负责首次构建系统，因此了解所有系统名称。

这个示例将为您演示使用动态清单的基本原理，然后您可以将其应用到其他系统的动态清单脚本中。让我们开始这个过程，首先安装 Cobbler——这个过程在 CentOS 7.8 上进行了测试：

1.  您的第一个任务是使用`yum`安装相关的 Cobbler 软件包。请注意，在撰写本文时，CentOS 7 提供的 SELinux 策略不支持 Cobbler 的功能，并阻止了一些方面的工作。尽管这不是您在生产环境中应该做的事情，但让这个演示快速运行的最简单方法是简单地禁用 SELinux：

```
$ yum install -y cobbler cobbler-web
$ setenforce 0
```

1.  接下来，请确保`cobblerd`服务已配置为在环回地址上监听，方法是检查`/etc/cobbler/settings`中的设置——文件的相关片段如下所示：

```
# default, localhost server: 127.0.0.1  
```

这不是一个公共监听地址，请*不要使用*`0.0.0.0`。您也可以将其设置为 Cobbler 服务器的 IP 地址。

1.  完成这一步后，您可以使用`systemctl`启动`cobblerd`服务。

```
$ systemctl start cobblerd.service
$ systemctl enable cobblerd.service
$ systemctl status cobblerd.service
```

1.  Cobbler 服务已经启动运行，现在我们将逐步介绍向 Cobbler 添加发行版的过程，以创建一些主机。这个过程非常简单，但您需要添加一个内核文件和一个初始 RAM 磁盘文件。获取这些文件的最简单来源是您的`/boot`目录，假设您已在 CentOS 7 上安装了 Cobbler。在用于此演示的测试系统上使用了以下命令，但是，您必须将`vmlinuz`和`initramfs`文件名中的版本号替换为您系统`/boot`目录中的适当版本号：

```
$ cobbler distro add --name=CentOS --kernel=/boot/vmlinuz-3.10.0-957.el7.x86_64 --initrd=/boot/initramfs-3.10.0-957.el7.x86_64.img

$ cobbler profile add --name=webservers --distro=CentOS
```

这个定义非常基础，可能无法生成可用的服务器镜像；但是，对于我们的简单演示来说，它足够了，因为我们可以基于这个假设的基于 CentOS 的镜像添加一些系统。请注意，我们正在创建的配置文件名`webservers`将在我们的动态清单中成为我们的清单组名。

1.  现在让我们将这些系统添加到 Cobbler 中。以下两个命令将向我们的 Cobbler 系统添加两个名为`frontend01`和`frontend02`的主机，使用我们之前创建的`webservers`配置文件：

```
$ cobbler system add --name=frontend01 --profile=webservers --dns-name=frontend01.example.com --interface=eth0

$ cobbler system add --name=frontend02 --profile=webservers --dns-name=frontend02.example.com --interface=eth0
```

请注意，为了使 Ansible 工作，它必须能够到达`--dns-name`参数中指定的这些 FQDN。为了实现这一点，我还在 Cobbler 系统的`/etc/hosts`中添加了这两台机器的条目，以确保我们以后可以到达它们。这些条目可以指向您选择的任何两个系统，因为这只是一个测试。

此时，您已成功安装了 Cobbler，创建了一个配置文件，并向该配置文件添加了两个假设系统。我们过程的下一阶段是下载并配置 Ansible 动态清单脚本，以便与这些条目一起使用。为了实现这一点，让我们开始执行以下给出的过程：

1.  从 GitHub Ansible 存储库下载 Cobbler 动态清单文件以及相关的配置文件模板。请注意，大多数由 Ansible 提供的动态清单脚本也有一个模板化的配置文件，其中包含您可能需要设置的参数，以使动态清单脚本工作。对于我们的简单示例，我们将把这些文件下载到我们当前的工作目录中：

```
$ wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/cobbler.py
$ wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/cobbler.ini
$ chmod +x cobbler.py
```

重要的是要记住，要使您下载的任何动态清单脚本可执行，就像之前展示的那样；如果您不这样做，那么即使其他一切都设置得完美，Ansible 也无法运行该脚本。

1.  编辑`cobbler.ini`文件，并确保它指向本地主机，因为在本例中，我们将在同一系统上运行 Ansible 和 Cobbler。在现实生活中，您会将其指向 Cobbler 系统的远程 URL。以下是配置文件的一部分，以便让您了解如何配置：

```
[cobbler]

# Specify IP address or Hostname of the cobbler server. The default variable is here:
host = http://127.0.0.1/cobbler_api

# (Optional) With caching, you will have responses of API call with the cobbler server quicker
cache_path = /tmp
cache_max_age = 900
```

1.  现在，您可以按照您习惯的方式运行 Ansible 的临时命令——这次唯一的区别是，您将指定动态清单脚本的文件名，而不是静态清单文件的名称。假设您已经在 Cobbler 中输入了两个地址的主机，您的输出应该看起来像这样：

```
$  ansible -i cobbler.py webservers -m ping
frontend01.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
frontend02.example.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
} 
```

就是这样！您刚刚在 Ansible 中实现了您的第一个动态清单。当然，我们知道许多读者不会使用 Cobbler，一些其他动态清单插件更复杂。例如，Amazon EC2 动态清单脚本需要您的 Amazon Web Services 的身份验证详细信息（或适当的 IAM 帐户）以及 Python `boto`和`boto3`库的安装。您怎么知道要做所有这些？幸运的是，所有这些都在动态清单脚本或配置文件的头部有记录，所以我能给出的最基本的建议是：每当您下载新的动态清单脚本时，请务必在您喜欢的编辑器中查看文件本身，因为它们的要求很可能已经为您记录了。

在本书的这一节结束之前，让我们看一下使用多个清单来源的其他一些方便提示，从下一节开始。

# 在清单目录中使用多个清单来源

到目前为止，在本书中，我们一直在使用我们的 Ansible 命令中的`-i`开关来指定我们的清单文件（静态或动态）。可能不明显的是，您可以多次指定`-i`开关，因此同时使用多个清单。这使您能够执行跨静态和动态清单的主机的任务，例如运行一个 playbook（或临时命令）。Ansible 将会计算出需要做什么——静态清单不应标记为可执行，因此不会被处理为这样，而动态清单将会被处理。这个小巧但聪明的技巧使您能够轻松地结合多个清单来源。让我们在下一节中继续看一下静态清单组与动态清单组的使用，这是多清单功能的扩展。

# 在动态组中使用静态组

当然，混合清单的可能性带来了一个有趣的问题——如果您同时定义动态清单和静态清单中的组，会发生什么？答案是 Ansible 会将两者结合起来，这带来了一个有趣的可能性。正如您所看到的，我们的 Cobbler 清单脚本从我们称为`webservers`的 Cobbler 配置文件中产生了一个名为`webservers`的 Ansible 组。这对于大多数动态清单提供者来说很常见；大多数清单来源（例如 Cobbler 和 Amazon EC2）都不是 Ansible 感知的，因此不提供 Ansible 可以直接使用的组。因此，大多数动态清单脚本将使用清单来源的某些信息来产生分组，Cobbler 机器配置文件就是一个例子。

让我们通过混合静态清单来扩展前一节中的 Cobbler 示例。假设我们想要将我们的`webservers`机器作为名为`centos`的组的子组，以便我们将来可以将所有 CentOS 机器分组在一起。我们知道我们只有一个名为`webservers`的 Cobbler 配置文件，理想情况下，我们不想开始干扰 Cobbler 设置，只是为了做一些与 Ansible 相关的事情。

解决这个问题的方法是创建一个具有两个组定义的静态清单文件。第一个必须与您从动态清单中期望的组的名称相同，只是您应该将其留空。当 Ansible 组合静态和动态清单内容时，它将重叠这两个组，因此将 Cobbler 的主机添加到这些`webservers`组中。

第二个组定义应该说明`webservers`是`centos`组的子组。生成的文件应该看起来像这样：

```
[webservers]

[centos:children]
webservers
```

现在让我们在 Ansible 中运行一个简单的临时`ping`命令，以查看它如何评估两个清单。请注意，我们将指定`centos`组来运行`ping`，而不是`webservers`组。我们知道 Cobbler 没有`centos`组，因为我们从未创建过，我们知道当您组合两个清单时，此组中的任何主机必须通过`webservers`组来，因为我们的静态清单中没有主机。结果将看起来像这样：

```
$ ansible -i static-groups-mix-ini -i cobbler.py centos -m ping
frontend01.example.com | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
frontend02.example.com | SUCCESS => {
 "ansible_facts": {
 "discovered_interpreter_python": "/usr/bin/python"
 },
 "changed": false,
 "ping": "pong"
}
```

从前面的输出中可以看出，我们引用了两个不同的清单，一个是静态的，另一个是动态的。我们已经组合了组，将仅存在于一个清单源中的主机与仅存在于另一个清单源中的组合在一起。正如您所看到的，这是一个非常简单的例子，很容易将其扩展为组合静态和动态主机的列表，或者向来自动态清单的主机添加自定义变量。

这是 Ansible 的一个鲜为人知的技巧，但在清单扩展和增长时可以非常强大。当我们通过本章工作时，您会注意到我们非常精确地指定了我们的清单主机，要么是单独的，要么是通过组；例如，我们明确告诉`ansible`对`webservers`组中的所有主机运行临时命令。在下一节中，我们将继续探讨 Ansible 如何管理使用模式指定的一组主机。

# 使用模式进行特殊主机管理

我们已经确定，您经常会想要针对清单的一个子部分运行一个临时命令或一个 playbook。到目前为止，我们一直在做得很精确，但现在让我们通过查看 Ansible 如何使用模式来确定应该针对哪些主机运行命令（或 playbook）来扩展这一点。

作为起点，让我们再次考虑本章早些时候定义的清单，以便探索主机组和子组。为了方便起见，清单内容再次提供如下：

```
loadbalancer.example.com

[frontends]
frt01.example.com
frt02.example.com

[apps]
app01.example.com
app02.example.com

[databases]
dbms01.example.com
dbms02.example.com

[centos:children]
apps
databases

[ubuntu:children]
frontends
```

为了演示通过模式进行主机/组选择，我们将使用`ansible`命令的`--list-hosts`开关来查看 Ansible 将对哪些主机进行操作。您可以扩展示例以使用`ping`模块，但出于空间和输出简洁可读的考虑，我们将在这里使用`--list-hosts`：

1.  我们已经提到了特殊的`all`组来指定清单中的所有主机：

```
$ ansible -i hostgroups-children-ini all --list-hosts
 hosts (7):
 loadbalancer.example.com
 frt01.example.com
 frt02.example.com
 app01.example.com
 app02.example.com
 dbms01.example.com
 dbms02.example.com
```

星号字符具有与`all`相同的效果，但需要在 shell 中用单引号引起来，以便 shell 正确解释命令：

```
$ ansible -i hostgroups-children-ini '*' --list-hosts
 hosts (7):
 loadbalancer.example.com
 frt01.example.com
 frt02.example.com
 app01.example.com
 app02.example.com
 dbms01.example.com
 dbms02.example.com
```

1.  使用`:`来指定逻辑`OR`，意思是“应用于这个组或那个组中的主机”，就像这个例子中一样：

```
$ ansible -i hostgroups-children-ini frontends:apps --list-hosts
 hosts (4):
 frt01.example.com
 frt02.example.com
 app01.example.com
 app02.example.com
```

1.  使用`!`来排除特定组——您可以将其与其他字符（例如`:`）结合使用，以显示（例如）除`apps`组中的所有主机之外的所有主机。同样，`!`是 shell 中的特殊字符，因此您必须在单引号中引用模式字符串，以使其正常工作，就像这个例子中一样：

```
$ ansible -i hostgroups-children-ini 'all:!apps' --list-hosts
 hosts (5):
 loadbalancer.example.com
 frt01.example.com
 frt02.example.com
 dbms01.example.com
 dbms02.example.com
```

1.  使用`:&`来指定两个组之间的逻辑`AND`，例如，如果我们想要在`centos`组和`apps`组中的所有主机（再次，您必须在 shell 中使用单引号）：

```
$ ansible -i hostgroups-children-ini 'centos:&apps' --list-hosts
  hosts (2):
    app01.example.com
    app02.example.com
```

1.  使用`*`通配符的方式与在 shell 中使用的方式类似，就像这个例子中一样：

```
$ ansible -i hostgroups-children-ini 'db*.example.com' --list-hosts
 hosts (2):
 dbms02.example.com
 dbms01.example.com
```

另一种限制命令运行的主机的方法是使用 Ansible 的`--limit`开关。这与前面的语法和模式表示完全相同，但它的优势在于您可以在`ansible-playbook`命令中使用它，而在命令行上指定主机模式仅支持`ansible`命令本身。因此，例如，您可以运行以下命令：

```
$ ansible-playbook -i hostgroups-children-ini site.yml --limit frontends:apps

PLAY [A simple playbook for demonstrating inventory patterns] ******************

TASK [Gathering Facts] *********************************************************
ok: [frt02.example.com]
ok: [app01.example.com]
ok: [frt01.example.com]
ok: [app02.example.com]

TASK [Ping each host] **********************************************************
ok: [app01.example.com]
ok: [app02.example.com]
ok: [frt02.example.com]
ok: [frt01.example.com]

PLAY RECAP *********************************************************************
app01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt01.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
frt02.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

模式是处理清单的非常有用和重要的部分，您无疑会发现它们非常有价值。这结束了我们关于 Ansible 清单的章节；但是，希望这为您提供了一切您需要自信地使用 Ansible 清单。

# 摘要

创建和管理 Ansible 清单是您使用 Ansible 的工作的重要部分，因此我们在本书的早期阶段就介绍了这个基本概念。它们至关重要，因为没有它们，Ansible 将不知道要针对哪些主机运行自动化任务，但它们提供的远不止这些。它们为配置管理系统提供了一个集成点，它们为存储主机特定（或组特定）变量提供了一个明智的来源，并且它们为您提供了运行此 playbook 的灵活方式。

在本章中，您学习了如何创建简单的静态清单文件并向其中添加主机。然后，我们通过学习如何添加主机组并为主机分配变量来扩展了这一点。我们还研究了在单个平面清单文件变得太难处理时如何组织您的清单和变量。然后，我们学习了如何利用动态清单文件，最后通过查看有用的技巧和诀窍，如组合清单来源和使用模式来指定主机，使您处理清单更加容易，同时更加强大。

在下一章中，我们将学习如何开发 playbooks 和 roles 来使用 Ansible 配置，部署和管理远程机器。

# 问题

1.  如何将`frontends`组变量添加到您的清单中？

A) `[frontends::]`

B) `[frontends::values]`

C) `[frontends:host:vars]`

D) `[frontends::variables]`

E) `[frontends:vars]`

1.  什么使您能够自动执行 Linux 任务，如提供 DNS，管理 DHCP，更新软件包和配置管理？

A) 播放书

B) Yum

C) 修鞋匠

D) Bash

E) 角色

1.  Ansible 允许您使用命令行上的`-i`选项来指定清单文件位置。

A) 真

B) 错误

# 进一步阅读

+   Ansible 的所有常见动态清单都在 GitHub 存储库中：[`github.com/ansible/ansible/tree/devel/contrib/inventory`](https://github.com/ansible/ansible/tree/devel/contrib/inventory)。
