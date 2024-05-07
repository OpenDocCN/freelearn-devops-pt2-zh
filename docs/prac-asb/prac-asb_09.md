# 第七章：编码最佳实践

Ansible 可以帮助您自动化几乎所有日常 IT 任务，从单调的任务，如应用补丁或部署配置文件，到部署全新的基础设施作为代码。随着越来越多的人意识到其强大和简单，Ansible 的使用和参与每年都在增长。您会在互联网上找到许多示例 Ansible playbook、角色、博客文章等，再加上本书这样的资源，您将能够熟练地编写自己的 Ansible playbook。

然而，您如何知道在 Ansible 中编写自动化代码的最佳方法是什么？您如何判断在互联网上找到的示例是否实际上是一种好的做事方式？在本章中，我们将带您了解 Ansible 最佳实践的实际指南，向您展示目前被认为是关于目录结构和 playbook 布局的良好实践，如何有效地使用清单（特别是在云上），以及如何最好地区分您的环境。通过本章的学习，您应该能够自信地编写从小型单任务 playbook 到复杂环境的大规模 playbook。

在本章中，我们将涵盖以下主题：

+   首选的目录布局

+   云清单的最佳方法

+   区分不同的环境类型

+   定义组和主机变量的正确方法

+   使用顶级 playbook

+   利用版本控制工具

+   设置操作系统和分发差异

+   Ansible 版本之间的移植

# 技术要求

本章假设您已经按照第一章 *开始使用 Ansible*中的方式设置了 Ansible 的控制主机，并且您正在使用最新版本；本章的示例是在 Ansible 2.9 上测试的。本章还假设您至少有一个额外的主机进行测试；理想情况下，这应该是基于 Linux 的。尽管本章将给出主机名的具体示例，但欢迎您用自己的主机名和/或 IP 地址替换它们，如何做到这一点的详细信息将在适当的地方提供。

本章中使用的代码包可以在[`github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%207`](https://github.com/PacktPublishing/Ansible-2-Cookbook/tree/master/Chapter%207)找到。

# 首选的目录布局

正如我们在本书中探讨了 Ansible 一样，我们多次表明，随着 playbook 的规模和规模的增长，您越有可能希望将其分成多个文件和目录。这方面的一个很好的例子是角色，在第四章 *Playbooks and Roles*中，我们定义了角色，不仅使我们能够重用常见的自动化代码，还使我们能够将潜在的庞大的单个 playbook 分成更小、逻辑上组织得更好、更易管理的部分。我们还在第三章 *Defining Your Inventory*中，探讨了定义清单文件的过程，以及如何将其分成多个文件和目录。然而，我们还没有探讨如何将所有这些放在一起。所有这些都在官方 Ansible 文档中有记录，网址是[`docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#content-organization`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#content-organization)。

然而，在本章中，让我们从一个实际的例子开始，向您展示一个设置简单基于角色的 playbook 的目录结构的好方法，其中有两个不同的清单——一个用于开发环境，一个用于生产环境（在任何真实的用例中，您都希望将它们分开，尽管理想情况下，您应该能够在两者上执行相同的操作以保持一致性和测试目的）。

让我们开始构建目录结构：

1.  使用以下命令为您的开发清单创建目录树：

```
$ mkdir -p inventories/development/group_vars
$ mkdir -p inventories/development/host_vars
```

1.  接下来，我们将为我们的开发清单定义一个 INI 格式的清单文件——在我们的示例中，我们将保持非常简单，只有两台服务器。要创建的文件是`inventories/development/hosts`：

```
[app]
app01.dev.example.com
app02.dev.example.com
```

1.  为了进一步说明，我们将为我们的 app 组添加一个组变量。如第三章中所讨论的，创建一个名为`app.yml`的文件，放在我们在上一步中创建的`group_vars`目录中：

```
---
http_port: 8080
```

1.  接下来，使用相同的方法创建一个`production`目录结构：

```
$ mkdir -p inventories/production/group_vars
$ mkdir -p inventories/production/host_vars
```

1.  在新创建的`production`目录中创建名为`hosts`的清单文件，并包含以下内容：

```
[app]
app01.prod.example.com
app02.prod.example.com
```

1.  现在，我们将为我们的生产清单的`http_port`组变量定义一个不同的值。将以下内容添加到`inventories/production/group_vars/app.yml`中：

```
---
http_port: 80
```

这完成了我们的清单定义。接下来，我们将添加任何我们可能发现对我们的 playbook 有用的自定义模块或插件。假设我们想要使用我们在第五章中创建的`remote_filecopy.py`模块。就像我们在本章中讨论的那样，我们首先为这个模块创建目录：

```
$ mkdir library
```

然后，将`remote_filecopy.py`模块添加到此库中。我们不会在这里重新列出代码以节省空间，但您可以从第五章中名为*开发自定义模块*的部分复制它，或者利用本书在 GitHub 上附带的示例代码。

插件也可以做同样的事情；如果我们还想使用我们在第六章中创建的`filter`插件，我们将创建一个适当命名的目录：

```
$ mkdir filter_plugins
```

然后，将`filter`插件代码复制到此目录中。

最后，我们将创建一个角色来在我们的新 playbook 结构中使用。当然，您会有很多角色，但我们将创建一个作为示例，然后您可以为每个角色重复这个过程。我们将称我们的角色为`installapp`，并使用`ansible-galaxy`命令（在第四章中介绍）为我们创建目录结构：

```
$ mkdir roles
$ ansible-galaxy role init --init-path roles/ installapp
- Role installapp was created successfully
```

然后，在我们的`roles/installapp/tasks/main.yml`文件中，我们将添加以下内容：

```
---
- name: Display http_port variable contents
  debug:
    var: http_port

- name: Create /tmp/foo
  file:
    path: /tmp/foo
    state: file

- name: Use custom module to copy /tmp/foo
  remote_filecopy:
    source: /tmp/foo
    dest: /tmp/bar

- name: Define a fact about automation
  set_fact:
    about_automation: "Puppet is an excellent automation tool"

- name: Tell us about automation with a custom filter applied
  debug:
    msg: "{{ about_automation | improve_automation }}"
```

在上述代码中，我们重用了本书前几章的许多示例。您还可以像之前讨论的那样为角色定义处理程序、变量、默认值等，但对于我们的示例来说，这就足够了。

创建我们最佳实践目录结构的最后阶段是添加一个顶层 playbook 来运行。按照惯例，这将被称为`site.yml`，并且它将具有以下简单内容（请注意，我们构建的目录结构处理了许多事情，使得顶层 playbook 非常简单）：

```
---
- name: Play using best practise directory structure
  hosts: all

  roles:
    - installapp
```

为了清晰起见，您的最终目录结构应如下所示：

```
.
├── filter_plugins
│   ├── custom_filter.py
│   └── custom_filter.pyc
├── inventories
│   ├── development
│   │   ├── group_vars
│   │   │   └── app.yml
│   │   ├── hosts
│   │   └── host_vars
│   └── production
│   ├── group_vars
│   │   └── app.yml
│   ├── hosts
│   └── host_vars
├── library
│   └── remote_filecopy.py
├── roles
│   └── installapp
│   ├── defaults
│   │   └── main.yml
│   ├── files
│   ├── handlers
│   │   └── main.yml
│   ├── meta
│   │   └── main.yml
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   ├── templates
│   ├── tests
│   │   ├── inventory
│   │   └── test.yml
│   └── vars
│   └── main.yml
└── site.yml
```

现在，我们可以以正常方式运行我们的 playbook。例如，要在开发清单上运行它，请执行以下操作：

```
$ ansible-playbook -i inventories/development/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app02.dev.example.com]
ok: [app01.dev.example.com]

TASK [installapp : Display http_port variable contents] ************************
ok: [app01.dev.example.com] => {
 "http_port": 8080
}
ok: [app02.dev.example.com] => {
 "http_port": 8080
}

TASK [installapp : Create /tmp/foo] ********************************************
changed: [app02.dev.example.com]
changed: [app01.dev.example.com]

TASK [installapp : Use custom module to copy /tmp/foo] *************************
changed: [app02.dev.example.com]
changed: [app01.dev.example.com]

TASK [installapp : Define a fact about automation] *****************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

TASK [installapp : Tell us about automation with a custom filter applied] ******
ok: [app01.dev.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
ok: [app02.dev.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

同样，对生产清单运行以下命令：

```
$ ansible-playbook -i inventories/production/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app02.prod.example.com]
ok: [app01.prod.example.com]

TASK [installapp : Display http_port variable contents] ************************
ok: [app01.prod.example.com] => {
 "http_port": 80
}
ok: [app02.prod.example.com] => {
 "http_port": 80
}

TASK [installapp : Create /tmp/foo] ********************************************
changed: [app01.prod.example.com]
changed: [app02.prod.example.com]

TASK [installapp : Use custom module to copy /tmp/foo] *************************
changed: [app02.prod.example.com]
changed: [app01.prod.example.com]

TASK [installapp : Define a fact about automation] *****************************
ok: [app01.prod.example.com]
ok: [app02.prod.example.com]

TASK [installapp : Tell us about automation with a custom filter applied] ******
ok: [app01.prod.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}
ok: [app02.prod.example.com] => {
 "msg": "Ansible is an excellent automation tool"
}

PLAY RECAP *********************************************************************
app01.prod.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.prod.example.com : ok=6 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

注意适当的主机和相关变量是如何被每个清单捕捉到的，以及我们的目录结构是多么整洁和有条理。这是你布置 playbooks 的理想方式，将确保它们可以按需扩展到任何你需要的规模，而不会变得笨重和难以管理或排查故障。在本章的下一节中，我们将探讨处理云清单的最佳方法。

# 云清单的最佳方法

在第三章《定义清单》中，我们看了一个简单的例子，介绍了如何使用动态清单，并通过使用 Cobbler provisioning 系统的实际示例为你提供了指导。然而，当涉及到使用云清单（它们只是动态清单的一种形式，但专门针对云）时，一开始可能会感到有些困惑，你可能会发现很难让它们运行起来。如果你按照本节概述的高级程序，这将成为一个简单而直接的任务。

由于这是一本实践性的书，我们将选择一个示例进行讨论。遗憾的是，我们没有空间为所有云提供商提供实际示例，但如果你按照我们将为亚马逊 EC2 概述的高级流程，并将其应用到你所需的云提供商（例如，Microsoft Azure 或 Google Cloud Platform），你会发现上手和运行的过程实际上非常简单。

然而，在开始之前需要注意的一点是，在包括 2.8.x 版本在内的 Ansible 版本中，动态清单脚本是 Ansible 源代码的一部分，并且可以从我们在本书中之前检查和克隆的主要 Ansible 存储库中获取。随着 Ansible 不断增长和扩展的性质，已经有必要在 2.9.x 版本（以及以后的版本）中将动态清单脚本分离到一个称为 Ansible 集合的新分发机制中，这将成为 2.10 版本的主流（在撰写本文时尚未发布）。你可以在[`www.ansible.com/blog/getting-started-with-ansible-collections`](https://www.ansible.com/blog/getting-started-with-ansible-collections)了解更多关于 Ansible 集合及其内容。

然而，随着 Ansible 2.10 版本的发布，你下载和使用动态清单脚本的方式可能会发生根本性的变化，然而，遗憾的是，在撰写本文时，关于这将是什么样子，目前还没有透露太多。因此，我们将指导你下载当前 2.9 版本所需的动态清单提供商脚本，并建议你在 2.10 版本发布时查阅 Ansible 文档，以获取相关脚本的下载位置。一旦你下载了它们，我相信你将能够按照本章概述的方式继续使用它们。

如果你正在使用 Ansible 2.9 版本，你可以在 GitHub 的 stable-2.9 分支上找到并下载所有最新的动态清单脚本，网址为[`github.com/ansible/ansible/tree/stable-2.9/contrib/inventory`](https://github.com/ansible/ansible/tree/stable-2.9/contrib/inventory)。

尽管官方的 Ansible 文档已经更新，但互联网上的大多数指南仍然引用这些脚本的旧 GitHub 位置，你会发现它们已经不再起作用。在使用动态清单时，请记住这一点！现在让我们继续讨论使用云提供商的动态清单脚本的过程；我们将以亚马逊 EC2 动态清单脚本作为工作示例，但我们在这里应用的原则同样适用于任何其他云清单脚本：

1.  在确定我们要使用 Amazon EC2 之后，我们的第一个任务是获取动态清单脚本及其相关的配置文件。由于云技术发展迅速，最安全的做法可能是直接从 GitHub 上的官方 Ansible 项目下载这些文件的最新版本。以下三个命令将下载动态清单脚本并使其可执行，以及下载模板配置文件：

```
$ wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.py
$ chmod +x ec2.py
$ wget https://raw.githubusercontent.com/ansible/ansible/stable-2.9/contrib/inventory/ec2.ini
```

1.  成功下载文件后，让我们来看看它们的内容。不幸的是，Ansible 动态清单没有与我们在模块和插件中看到的那样整洁的文档系统。然而，对我们来说幸运的是，这些动态清单脚本的作者在这些文件的顶部放置了许多有用的注释，以帮助我们入门。让我们来看看`ec2.py`的内容：

```
#!/usr/bin/env python

'''
EC2 external inventory script
=================================

Generates inventory that Ansible can understand by making API request to
AWS EC2 using the Boto library.

NOTE: This script assumes Ansible is being executed where the environment
variables needed for Boto have already been set:
    export AWS_ACCESS_KEY_ID='AK123'
    export AWS_SECRET_ACCESS_KEY='abc123'

Optional region environment variable if region is 'auto'

This script also assumes that there is an ec2.ini file alongside it. To specify
 a
different path to ec2.ini, define the EC2_INI_PATH environment variable:

    export EC2_INI_PATH=/path/to/my_ec2.ini
```

有很多文档需要阅读，但其中一些最相关的信息包含在那些开头几行中。首先，我们需要确保`Boto`库已安装。其次，我们需要为`Boto`设置 AWS 访问参数。本文档的作者已经给了我们最快的入门方式（确实，他们的工作不是复制`Boto`文档）。

但是，如果您参考`Boto`的官方文档，您会发现有很多配置 AWS 凭据的方法——设置环境变量只是其中之一。您可以在[`boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html`](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html)上阅读有关配置`Boto`身份验证的更多信息。

1.  在继续安装`Boto`之前，让我们来看看示例`ec2.ini`文件：

```
# Ansible EC2 external inventory script settings
#

[ec2]

# to talk to a private eucalyptus instance uncomment these lines
# and edit edit eucalyptus_host to be the host name of your cloud controller
#eucalyptus = True
#eucalyptus_host = clc.cloud.domain.org

# AWS regions to make calls to. Set this to 'all' to make request to all regions
# in AWS and merge the results together. Alternatively, set this to a comma
# separated list of regions. E.g. 'us-east-1,us-west-1,us-west-2' and do not
# provide the 'regions_exclude' option. If this is set to 'auto', AWS_REGION or
# AWS_DEFAULT_REGION environment variable will be read to determine the region.
regions = all
regions_exclude = us-gov-west-1, cn-north-1
```

同样，您可以在此文件中看到大量经过良好记录的选项，并且如果您滚动到底部，甚至会发现您可以在此文件中指定您的凭据，作为先前讨论的方法的替代。然而，默认设置对于您只是想开始使用的情况已经足够了。

1.  让我们现在确保`Boto`库已安装；确切的安装方法将取决于您选择的操作系统和 Python 版本。您可能可以通过软件包安装它；在 CentOS 7 上，您可以按照以下步骤执行此操作：

```
$ sudo yum -y install python-boto python-boto3
```

或者，您可以使用`pip`来实现这一目的。例如，要将其安装为 Python 3 环境的一部分，您可以运行以下命令：

```
$ sudo pip3 install boto3
```

1.  安装了`Boto`之后，让我们继续使用前面文档中建议给我们的环境变量来设置我们的 AWS 凭据：

```
$ export AWS_ACCESS_KEY_ID='<YOUR_DATA>'
$ export AWS_SECRET_ACCESS_KEY='<YOUR_DATA>'
```

1.  完成这些步骤后，您现在可以像往常一样使用动态清单脚本——只需使用`-i`参数引用可执行的清单脚本，就像您在静态清单中所做的那样。例如，如果您想对在 Amazon EC2 上运行的所有主机运行 Ansible `ping`模块作为临时命令，您需要运行以下命令。确保用`-u`开关指定的用户帐户替换连接到 EC2 实例的用户帐户。还要引用您的私有 SSH 密钥文件：

```
$ ansible -i ec2.py -u ec2-user --private-key /home/james/my-ec2-id_rsa -m ping all
```

就是这样——如果您以同样的系统方法处理所有动态清单脚本，那么您将毫无问题地使它们运行起来。只需记住，文档通常嵌入在脚本文件和其附带的配置文件中，请确保在尝试使用脚本之前阅读两者。

需要注意的一点是，许多动态清单脚本，包括`ec2.py`，会缓存其对云提供商的 API 调用结果，以加快重复运行的速度并避免过多的 API 调用。然而，在快速发展的开发环境中，你可能会发现云基础设施的更改没有被及时捕捉到。对于大多数脚本，有两种解决方法——大多数特性缓存配置参数在其配置文件中，比如`ec2.ini`中的`cache_path`和`cache_max_age`参数。如果你不想为每次运行都设置这些参数，你也可以通过直接调用动态清单脚本并使用特殊开关来手动刷新缓存，例如在`ec2.py`中：

```
$ ./ec2.py --refresh-cache
```

这就结束了我们对云清单脚本的实际介绍。正如我们讨论过的，只要你查阅文档（包括互联网上的文档和每个动态清单脚本中嵌入的文档），并遵循我们描述的简单方法，你应该不会遇到问题，并且应该能够在几分钟内开始使用动态清单。在下一节中，我们将回到静态清单，并探讨区分各种技术环境的最佳方法。

# 区分不同的环境类型

在几乎每个企业中，你都需要按类型划分你的技术环境。例如，你几乎肯定会有一个开发环境，在这里进行所有的测试和开发工作，并且有一个生产环境，在这里运行所有稳定的测试代码。这些环境（在最理想的情况下）应该使用相同的 Ansible playbooks——毕竟，逻辑是，如果你能够在开发环境成功部署和测试一个应用程序，那么你应该能够以同样的方式在生产环境中部署它，并且它能够正常运行。然而，这两个环境之间总是存在差异，不仅仅是在主机名上，有时还包括参数、负载均衡器名称、端口号等等——这个列表似乎是无穷无尽的。

在本章的*首选目录布局*部分，我们介绍了使用两个单独的清单目录树来区分开发和生产环境的方法。当涉及到区分这些环境时，你应该按照这种方式进行；因此，显然，我们不会重复这些例子，但重要的是要注意，当处理多个环境时，你的目标应该是：

+   尽量重用相同的 playbooks 来运行相同代码的所有环境。例如，如果你在开发环境部署了一个 web 应用程序，你应该有信心你的 playbooks 也能在生产环境（以及你的**质量保证**（**QA**）环境，以及其他可能需要部署的环境）中部署相同的应用程序。

+   这意味着你不仅在测试应用程序部署和代码，还在测试 Ansible 的 playbooks 和 roles 作为整个测试过程的一部分。

+   每个环境的清单应该保存在单独的目录树中（就像本章的*首选目录布局*部分所示），但所有的 roles、playbooks、插件和模块（如果有的话）都应该在相同的目录结构中（这对于两个环境来说应该是一样的）。

+   不同的环境通常需要不同的身份验证凭据；你应该将这些凭据分开保存，不仅是为了安全，还为了确保 playbooks 不会意外地在错误的环境中运行。

+   你的 playbooks 应该在你的版本控制系统中，就像你的代码一样。这样可以让你随着时间跟踪变化，并确保每个人都在使用相同的自动化代码副本。

如果您注意这些简单的指针，您会发现您的自动化工作流程成为您业务的真正资产，并确保在所有部署中可靠性和一致性。相反，不遵循这些指针会使您面临在开发中运行正常但在生产中运行失败的可怕情况，这经常困扰着技术行业。现在，让我们在下一节中继续讨论，看看在处理主机和组变量时的最佳实践，正如我们在*首选目录布局*部分中所看到的，您需要应用这些实践，特别是在处理多个环境时。

# 定义组和主机变量的正确方法

在处理组和主机变量时，您可以使用我们在*首选目录布局*部分中使用的基于目录的方法进行拆分。但是，有一些额外的指针可以帮助您管理这一点。首先，您应该始终注意变量的优先级。变量优先级顺序的详细列表可以在[`docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)找到。但是，处理多个环境的关键要点如下：

+   主机变量始终比组变量的优先级高；因此，您可以使用主机变量覆盖任何组变量。如果您以受控的方式利用此行为，这种行为是有用的，但如果您不了解它，可能会产生意想不到的结果。

+   有一个名为`all`的特殊组变量定义，适用于所有清单组。这比特定定义的组变量的优先级低。

+   如果您在两个组中定义相同的变量会发生什么？如果发生这种情况，两个组具有相同的优先级，那么谁会获胜？为了演示这一点（以及我们之前的例子），我们将为您创建一个简单的实际示例。

要开始，让我们为我们的清单创建一个目录结构。为了尽可能简洁，我们只会创建一个开发环境的例子。但是，您可以通过在本章的*首选目录布局*部分构建更完整的示例来扩展这些概念：

1.  使用以下命令创建一个清单目录结构：

```
$ mkdir -p inventories/development/group_vars
$ mkdir -p inventories/development/host_vars
```

1.  在`inventories/development/hosts`文件中创建一个包含两个主机的单个组的简单清单文件；内容应如下所示：

```
[app]
app01.dev.example.com
app02.dev.example.com
```

1.  现在，让我们为清单中的所有组创建一个特殊的组变量文件；这个文件将被称为`inventories/development/group_vars/all.yml`，应包含以下内容：

```
---
http_port: 8080
```

1.  最后，让我们创建一个名为`site.yml`的简单 playbook，以查询和打印我们刚刚创建的变量的值：

```
---
- name: Play using best practise directory structure
  hosts: all

  tasks:
    - name: Display the value of our inventory variable
      debug:
        var: http_port
```

1.  现在，如果我们运行这个 playbook，我们会看到变量（我们只在一个地方定义）取得了我们期望的值：

```
$ ansible-playbook -i inventories/development/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

TASK [Display the value of our inventory variable] *****************************
ok: [app01.dev.example.com] => {
 "http_port": 8080
}
ok: [app02.dev.example.com] => {
 "http_port": 8080
}

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

1.  到目前为止，一切顺利！现在，让我们向我们的清单目录结构添加一个新文件，`all.yml`文件保持不变。我们还将创建一个位于`inventories/development/group_vars/app.yml`的新文件，其中将包含以下内容：

```
---
http_port: 8081
```

1.  我们现在在一个名为`all`的特殊组和`app`组中定义了相同的变量（我们的开发清单中的两个服务器都属于这个组）。那么，如果我们现在运行我们的 playbook 会发生什么？输出应如下所示：

```
$ ansible-playbook -i inventories/development/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app02.dev.example.com]
ok: [app01.dev.example.com]

TASK [Display the value of our inventory variable] *****************************
ok: [app01.dev.example.com] => {
 "http_port": 8081
}
ok: [app02.dev.example.com] => {
 "http_port": 8081
}

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

1.  如预期的那样，在特定组中的变量定义获胜，这符合 Ansible 文档中记录的优先顺序。现在，让我们看看如果我们在两个特定命名的组中定义相同的变量会发生什么。为了完成这个示例，我们将创建一个子组，称为`centos`，以及另一个可能包含按照新的构建标准构建的主机的组，称为`newcentos`，这两个应用服务器都将是其成员。这意味着修改`inventories/development/hosts`，使其看起来如下：

```
[app]
app01.dev.example.com
app02.dev.example.com

[centos:children]
app

[newcentos:children]
app
```

1.  现在，让我们通过创建一个名为`inventories/development/group_vars/centos.yml`的文件来重新定义`centos`组的`http_port`变量，其中包含以下内容：

```
---
http_port: 8082
```

1.  为了增加混乱，让我们也在`inventories/development/group_vars/newcentos.yml`中为`newcentos`组定义这个变量，其中包含以下内容：

```
---
http_port: 8083
```

1.  我们现在在组级别定义了相同的变量四次！让我们重新运行我们的 playbook，看看哪个值会通过：

```
$ ansible-playbook -i inventories/development/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

TASK [Display the value of our inventory variable] *****************************
ok: [app01.dev.example.com] => {
 "http_port": 8083
}
ok: [app02.dev.example.com] => {
 "http_port": 8083
}

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

我们在`newcentos.yml`中输入的值赢了-但为什么？Ansible 文档规定，在清单中（唯一可以这样做的地方）在组级别定义相同的变量时，最后加载的组中的变量获胜。组按字母顺序处理，`newcentos`是字母表中最后一个字母开头的组-因此，它的`http_port`值是获胜的值。

1.  为了完整起见，我们可以通过不触及`group_vars`目录，但添加一个名为`inventories/development/host_vars/app01.dev.example.com.yml`的文件来覆盖所有这些，其中包含以下内容：

```
---
http_port: 9090
```

1.  现在，如果我们最后再次运行我们的 playbook，我们会看到我们在主机级别定义的值完全覆盖了我们为`app01.dev.example.com`设置的任何值。`app02.dev.example.com`不受影响，因为我们没有为它定义主机变量，所以优先级的下一个最高级别是`newcentos`组的组变量：

```
$ ansible-playbook -i inventories/development/hosts site.yml

PLAY [Play using best practise directory structure] ****************************

TASK [Gathering Facts] *********************************************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

TASK [Display the value of our inventory variable] *****************************
ok: [app01.dev.example.com] => {
 "http_port": 9090
}
ok: [app02.dev.example.com] => {
 "http_port": 8083
}

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=2 changed=0 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

有了这些知识，你现在可以做出关于如何在清单中结构化你的变量以确保在主机和组级别都能达到期望结果的高级决策。了解变量优先级顺序是很重要的，因为这些示例已经证明了这一点，但遵循文档中的顺序也将使你能够创建功能强大、灵活的 playbook 清单，可以在多个环境中很好地工作。现在，你可能已经注意到，在本章中，我们在我们的目录结构中使用了一个名为`site.yml`的顶层 playbook。我们将在下一节更详细地讨论这个 playbook。

# 使用顶层 playbooks

到目前为止的所有示例中，我们都是使用 Ansible 推荐的最佳实践目录结构构建的，并且不断地引用顶层 playbook，通常称为`site.yml`。这个 playbook 的理念，实际上，是在我们所有的目录结构中都有一个共同的名字，这样它就可以在整个服务器环境中使用-也就是说，你的**site**。

当然，这并不是说你必须在基础设施的每台服务器或每个功能上使用相同的 playbook 集合；相反，这意味着只有你才能做出最适合你环境的最佳决定。然而，Ansible 自动化的整个目标是所创建的解决方案简单易于运行和操作。想象一下，将一个包含 100 个不同 playbook 的 playbook 目录结构交给一个新的系统管理员-他们怎么知道应该在什么情况下运行哪些 playbook？培训某人使用 playbook 的任务将是巨大的，只会将复杂性从一个领域转移到另一个领域。

在另一端，您可以利用`when`子句与事实和清单分组，以便您的 playbook 确切地知道在每种可能的情况下在每台服务器上运行什么。当然，这是不太可能发生的，事实是您的自动化解决方案最终会处于中间位置。

最重要的是，当收到新的 playbook 目录结构时，新操作员至少知道运行 playbook 和理解代码的起点在哪里。如果他们遇到的顶级 playbook 总是`site.yml`，那么至少每个人都知道从哪里开始。通过巧妙地使用角色和`import_*`和`include_*`语句，您可以将 playbook 分割成可重用代码的逻辑部分，正如我们之前讨论的那样，所有这些都来自一个 playbook 文件。

现在您已经了解了顶级 playbook 的重要性，让我们在下一节中看看如何利用版本控制工具来确保在集中和维护自动化代码时遵循良好的实践。

# 利用版本控制工具

正如我们在本章前面讨论的那样，对于您的 Ansible 自动化代码，版本控制和测试不仅仅是您的代码，还包括清单（或动态清单脚本）、任何自定义模块、插件、角色和 playbook 代码都至关重要。这是因为 Ansible 自动化的最终目标很可能是使用 playbook（或一组 playbook）部署整个环境。这甚至可能涉及部署基础设施作为代码，特别是如果您要部署到云环境中。

对 Ansible 代码的任何更改可能意味着对您的环境的重大更改，甚至可能意味着重要的生产服务是否正常工作。因此，非常重要的是您保持 Ansible 代码的版本历史，并且每个人都使用相同的版本。您可以自由选择最适合您的版本控制系统；大多数公司环境已经有某种版本控制系统。但是，如果您以前没有使用过版本控制系统，我们建议您在 GitHub 或 GitLab 等地方注册免费帐户，这两者都提供免费的版本控制存储库，以及更高级的付费计划。

关于 Git 的版本控制的完整讨论超出了本书的范围；事实上，整本书都致力于这个主题。但是，我们将带您了解最简单的用例。在以下示例中，假定您正在使用 GitHub 上的免费帐户，但如果您使用不同的提供商，只需更改 URL 以匹配您的版本控制存储库主机给您的 URL。

除此之外，您还需要在 Linux 主机上安装命令行 Git 工具。在 CentOS 上，您可以按照以下方式安装这些工具：

```
$ sudo yum install git
```

在 Ubuntu 上，这个过程同样简单：

```
$ sudo apt-get update
$ sudo apt-get install git
```

工具安装完成并且您的帐户设置好之后，您的下一个任务是将 Git 存储库克隆到您的计算机。如果您想开始使用自己的存储库进行工作，您需要与提供商一起设置这一点——GitHub 和 GitLab 都提供了出色的文档，您应该按照这些文档设置您的第一个存储库。

一旦设置和初始化，您可以克隆一个副本到您的本地计算机以对代码进行更改。这个本地副本称为工作副本，您可以按照以下步骤进行克隆和更改的过程（请注意，这些纯属假设性的例子，只是为了让您了解需要运行的命令；您应该根据自己的用例进行调整）：

1.  使用以下命令将您的`git`存储库克隆到本地计算机以创建一个工作副本：

```
$ git clone https://github.com/<YOUR_GIT_ACCOUNT>/<GIT_REPO>.git
Cloning into '<GIT_REPO>'...
remote: Enumerating objects: 7, done.
remote: Total 7 (delta 0), reused 0 (delta 0), pack-reused 7
Unpacking objects: 100% (7/7), done. 
```

1.  切换到您克隆的代码目录（工作副本）并进行任何需要的代码更改：

```
$ cd <GIT_REPO>
$ vim myplaybook.yml
```

1.  确保测试您的代码，并且当您对其满意时，添加已准备提交新版本的更改文件，使用以下命令：

```
$ git add myplaybook.yml
```

1.  接下来要做的是提交您所做的更改。提交基本上是存储库中的新代码版本，因此应该附有有意义的`commit`消息（在`-m`开关后面用引号指定），如下所示：

```
$ git commit -m 'Added new spongle-widget deployment to myplaybook.yml'
[master ed14138] Added new spongle-widget deployment to myplaybook.yml
 Committer: Daniel Oh <doh@danieloh.redhat.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 1 insertion(+), 1 deletion(-) 
```

1.  现在，所有这些更改都仅存在于您本地计算机上的工作副本中。这本身就很好，但如果代码可以供所有需要在版本控制系统上查看它的人使用，那将更好。要将更新的提交推送回（例如）GitHub，运行以下命令：

```
$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 297 bytes | 297.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/<YOUR_GIT_ACCOUNT>/<GIT_REPO>.git
   0d00263..ed14138 master -> master 
```

就是这样！

1.  现在，其他合作者可以克隆您的代码，就像我们在*步骤 1*中所做的那样。或者，如果他们已经有您的存储库的工作副本，他们可以使用以下命令更新他们的工作副本（如果您想要更新您的工作副本以查看其他人所做的更改，也可以这样做）：

```
$ git pull
```

Git 还有一些非常高级的主题和用例超出了本书的范围。但是，您会发现大约 80%的时间，前面的命令就是您需要的所有 Git 命令行知识。还有许多图形界面的 Git 前端，以及与 Git 存储库集成的代码编辑器和**集成开发环境**（**IDEs**），可以帮助您更好地利用它们。完成这些后，让我们看看如何确保您可以在多个主机上使用相同的 playbook（或 role），即使它们可能具有不同的操作系统和版本。

# 设置 OS 和分发差异

如前所述，我们的目标是尽可能广泛地使用相同的自动化代码。然而，尽管我们努力标准化我们的技术环境，变体总是会出现。例如，不可能同时对所有服务器进行主要升级，因此当出现主要的新操作系统版本时，例如**Red Hat Enterprise Linux**（**RHEL**）8 或 Ubuntu Server 20.04，一些机器将保持在旧版本上，而其他机器将进行升级。同样，一个环境可能标准化为 Ubuntu，但随后引入了一个只在 CentOS 上获得认证的应用程序。简而言之，尽管标准化很重要，但变体总是会出现。

在编写 Ansible playbook 时，特别是 role 时，您的目标应该是使它们尽可能广泛地适用于您的环境。其中一个经典例子是软件包管理——假设您正在编写一个安装 Apache 2 Web 服务器的 role。如果您必须使用此 role 支持 Ubuntu 和 CentOS，不仅要处理不同的软件包管理器（`yum`和`apt`），还要处理不同的软件包名称（`httpd`和`apache2`）。

在第四章中，*Playbooks and Roles*，我们看了如何使用`when`子句将条件应用于任务，以及 Ansible 收集的事实，如`ansible_distribution`。然而，还有另一种在特定主机上运行任务的方法，我们还没有看过。在同一章中，我们还看了如何在一个 playbook 中定义多个 play 的概念——有一个特殊的模块可以根据 Ansible 事实为我们创建清单组，我们可以利用这一点以及多个 play 来创建一个 playbook，根据主机的类型在每个主机上运行适当的任务。最好通过一个实际的例子来解释这一点，所以让我们开始吧。

假设我们在此示例中使用以下简单的清单文件，其中有两个主机在一个名为`app`的单个组中：

```
[app]
app01.dev.example.com
app02.dev.example.com
```

现在让我们构建一个简单的 playbook，演示如何使用 Ansible 事实对不同的 play 进行分组，以便操作系统分发确定 playbook 中运行哪个 play。按照以下步骤创建此 playbook 并观察其运行：

1.  首先创建一个新的 playbook——我们将其称为`osvariants.yml`——包含以下`Play`定义。它还将包含一个单独的任务，如下所示：

```
---
- name: Play to demonstrate group_by module
  hosts: all

  tasks:
    - name: Create inventory groups based on host facts
      group_by:
        key: os_{{ ansible_facts['distribution'] }}
```

到目前为止，playbook 结构对您来说应该已经非常熟悉了。但是，使用`group_by`模块是新的。它根据我们指定的键动态创建新的清单组——在本例中，我们根据从`Gathering Facts`阶段获取的 OS 发行版事实创建组。原始清单组结构保留不变，但所有主机也根据其事实添加到新创建的组中。

因此，我们简单清单中的两台服务器仍然在`app`组中，但如果它们基于 Ubuntu，它们将被添加到一个名为`os_Ubuntu`的新创建的清单组中。同样，如果它们基于 CentOS，它们将被添加到一个名为`os_CentOS`的组中。

1.  有了这些信息，我们可以继续根据新创建的组创建额外的 play。让我们将以下`Play`定义添加到同一个 playbook 文件中，以在 CentOS 上安装 Apache：

```
- name: Play to install Apache on CentOS
  hosts: os_CentOS
  become: true

  tasks:
    - name: Install Apache on CentOS
      yum:
        name: httpd
        state: present
```

这是一个完全正常的`Play`定义，它使用`yum`模块来安装`httpd`包（在 CentOS 上需要）。唯一与我们之前工作不同的是 play 顶部的`hosts`定义。这使用了第一个 play 中由`group_by`模块创建的新创建的清单组。

1.  同样，我们可以添加第三个`Play`定义，这次是使用`apt`模块在 Ubuntu 上安装`apache2`包：

```
- name: Play to install Apache on Ubuntu
  hosts: os_Ubuntu
  become: true

  tasks:
    - name: Install Apache on Ubuntu
      apt:
        name: apache2
        state: present
```

1.  如果我们的环境是基于 CentOS 服务器并运行此 playbook，则结果如下：

```
$ ansible-playbook -i hosts osvariants.yml

PLAY [Play to demonstrate group_by module] *************************************

TASK [Gathering Facts] *********************************************************
ok: [app02.dev.example.com]
ok: [app01.dev.example.com]

TASK [Create inventory groups based on host facts] *****************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

PLAY [Play to install Apache on CentOS] ****************************************

TASK [Gathering Facts] *********************************************************
ok: [app01.dev.example.com]
ok: [app02.dev.example.com]

TASK [Install Apache on CentOS] ************************************************
changed: [app02.dev.example.com]
changed: [app01.dev.example.com]
[WARNING]: Could not match supplied host pattern, ignoring: os_Ubuntu

PLAY [Play to install Apache on Ubuntu] ****************************************
skipping: no hosts matched

PLAY RECAP *********************************************************************
app01.dev.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
app02.dev.example.com : ok=4 changed=2 unreachable=0 failed=0 skipped=0 rescued=0 ignored=0
```

请注意安装 CentOS 上的 Apache 的任务是如何运行的。它是这样运行的，因为`group_by`模块创建了一个名为`os_CentOS`的组，而我们的第二个 play 仅在名为`os_CentOS`的组中运行。由于清单中没有运行 Ubuntu 的服务器，因此`os_Ubuntu`组从未被创建，因此第三个 play 不会运行。我们会收到有关没有与`os_Ubuntu`匹配的主机模式的警告，但 playbook 不会失败——它只是跳过了这个 play。

我们提供了这个例子，以展示另一种管理自动化编码中不可避免的 OS 类型差异的方式。归根结底，选择最适合您的编码风格取决于您。您可以使用`group_by`模块，如此处所述，或者将任务编写成块，并向块添加`when`子句，以便仅在满足某个基于事实的条件时运行（例如，OS 发行版是 CentOS）——或者甚至两者的组合。选择最终取决于您，这些不同的示例旨在为您提供多种选择，以便您在其中选择最佳解决方案。

最后，让我们通过查看在 Ansible 版本之间移植自动化代码来结束本章。

# 在 Ansible 版本之间移植

Ansible 是一个快速发展的项目，随着发布和新功能的添加，新模块（和模块增强）被发布，软件中不可避免的错误也得到修复。毫无疑问，您最终会写出针对 Ansible 的一个版本的代码，然后在某个时候需要再次在更新的版本上运行它。举例来说，当我们开始写这本书时，当前的 Ansible 版本是 2.7。当我们编辑这本书准备出版时，版本 2.9.6 是当前的稳定版本。

通常情况下，当你升级时，你会发现你的早期版本的代码“差不多能用”，但这并不总是确定的。有时模块会被弃用（尽管通常不会没有警告地弃用），功能也会发生变化。预计 Ansible 2.10 发布时会有一些重大变更。因此，问题是——当你更新你的 Ansible 安装时，如何确保你的剧本、角色、模块和插件仍然能够正常工作？

回答的第一部分是确定你从哪个版本的 Ansible 开始。例如，假设你正在准备升级到 Ansible 2.10。如果你查询已安装的 Ansible 版本，看到类似以下的内容，那么你就知道你是从 Ansible 2.9 版本开始的：

```
$ ansible --version
ansible 2.9.6
 config file = /etc/ansible/ansible.cfg
 configured module search path = [u'/home/james/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/site-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.5 (default, Aug 7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
```

因此，你首先需要查看 Ansible 2.10 版本的迁移指南；通常每个主要版本（比如 2.8、2.9 等）都会有一个迁移指南。2.10 版本的指南可以在 [`docs.ansible.com/ansible/devel/porting_guides/porting_guide_2.10.html`](https://docs.ansible.com/ansible/devel/porting_guides/porting_guide_2.10.html) 找到。

如果我们查看这份文档，我们会发现即将有一些变更——这些变更对你是否重要取决于你正在运行的代码。例如，如果我们查看指南中的 *Modules Removed* 部分，我们会发现 `letsencrypt` 模块已被移除，并建议你使用 `acme_certificate` 模块代替。如果你在 Ansible 中使用 `letsencrypt` 模块生成免费的 SSL 证书，那么你肯定需要更新你的剧本和角色以适应这一变更。

正如你在前面的链接中看到的，Ansible 2.9 和 2.10 版本之间有大量的变更。因此，重要的是要注意，迁移指南是从升级前一个主要版本的角度编写的。也就是说，如果你查询你的 Ansible 版本并返回以下内容，那么你是从 Ansible 2.8 迁移过来的：

```
$ ansible --version
ansible 2.8.4
 config file = /etc/ansible/ansible.cfg
 configured module search path = [u'/home/james/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
 ansible python module location = /usr/lib/python2.7/site-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.5 (default, Aug 7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
```

如果你直接升级到 Ansible 2.10，那么你需要查看 2.9（涵盖了从 2.8 到 2.9 的代码变更）和 2.10（涵盖了从 2.9 到 2.10 的升级所需的变更）的迁移指南。所有迁移指南的索引可以在官方 Ansible 网站上找到，网址是 [`docs.ansible.com/ansible/devel/porting_guides/porting_guides.html`](https://docs.ansible.com/ansible/devel/porting_guides/porting_guides.html)。

另一个获取信息的好途径，尤其是更精细的信息，是变更日志。这些日志会在每个次要版本发布时发布和更新，目前可以在官方 Ansible GitHub 仓库的`stable`分支上找到，用于你想查询的版本。例如，如果你想查看 Ansible 2.9 的所有变更日志，你需要前往 [`github.com/ansible/ansible/blob/stable-2.9/changelogs/CHANGELOG-v2.9.rst`](https://github.com/ansible/ansible/blob/stable-2.9/changelogs/CHANGELOG-v2.9.rst)。

将代码从 Ansible 版本迁移到另一个版本（如果你愿意这么称呼的话）的诀窍就是阅读 Ansible 项目团队发布的优秀文档。大量的工作投入到了创建这些文档中，因此建议你充分利用。这就结束了我们对使用 Ansible 的最佳实践的介绍。希望你觉得这一章很有价值。

# 总结

Ansible 自动化项目通常从小规模开始，但随着人们意识到 Ansible 的强大和简单，代码和清单往往呈指数增长（至少在我的经验中是这样）。在推动更大规模自动化的过程中，重要的是 Ansible 自动化代码和基础设施本身不会成为另一个头疼事。通过在早期嵌入一些良好的实践并在整个使用 Ansible 进行自动化的过程中始终如一地应用它们，您会发现管理 Ansible 自动化是简单易行的，并且对您的技术基础设施是真正有益的。

在本章中，您了解了应该为 playbook 采用的目录布局的最佳实践，以及在使用云清单时应采用的步骤。然后，您学习了通过 OS 类型区分环境的新方法，以及有关变量优先级以及在处理主机和组变量时如何利用它的更多信息。然后，您探索了顶级 playbook 的重要性，然后看了如何利用版本控制工具来管理您的自动化代码。最后，您探讨了创建单个 playbook 的新技术，该 playbook 将管理不同 OS 版本和发行版的服务器，最后看了将代码移植到新的 Ansible 版本的重要主题。

在下一章中，我们将探讨您可以使用 Ansible 来处理在自动化过程中可能出现的一些特殊情况的一些更高级的方法。

# 问题

1.  什么是一种安全且简单的方式来持续管理（即修改、修复和创建）代码更改并与他人共享？

A）Playbook 修订

B）任务历史

C）临时创建

D）使用 Git 存储库

E）日志管理

1.  Ansible Galaxy 支持从中央、社区支持的存储库与其他用户共享角色。 

A）真

B）假

1.  真或假- Ansible 模块保证在将来的所有版本中都可用。

A）真

B）假

# 进一步阅读

通过创建分支和标签来管理多个存储库、版本或任务，以有效地控制多个版本。有关更多详细信息，请参考以下链接：

+   如何使用 Git 标记：[`git-scm.com/book/en/v2/Git-Basics-Tagging`](https://git-scm.com/book/en/v2/Git-Basics-Tagging)

+   如何使用 Git 分支：[`git-scm.com/docs/git-branch`](https://git-scm.com/docs/git-branch)
