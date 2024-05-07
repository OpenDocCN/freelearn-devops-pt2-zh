# 前言

随着 OpenStack 开始被视为更主流的云平台，构建后对其进行操作的挑战变得更加突出。虽然所有云任务都可以通过 API 或 CLI 工具逐个执行，但这并不是处理更大规模云部署的最佳方式。现在明显需要更多自动化的方法来管理 OpenStack。大多数组织都在寻求改善业务敏捷性的方法，并意识到仅仅拥有一个云是不够的。只有通过某种形式的自动化才能改善应用部署、减少基础设施停机时间和消除日常手动任务。OpenStack 和 Ansible 将帮助任何组织弥合这一差距。OpenStack 提供的许多基础设施即服务功能，再加上 Ansible 这种易于使用的配置管理工具，可以确保更完整的云实施。

无论您是 OpenStack 的新手还是经验丰富的云管理员，本书都将帮助您在设置好 OpenStack 云后进行管理。本书充满了真实的 OpenStack 管理任务，我们将首先逐步介绍这些工作示例，然后过渡到介绍如何使用最流行的开源自动化工具之一 Ansible 来自动化这些任务的说明。

Ansible 已经成为开源编排和自动化领域的市场领导者。它也是使用 Python 构建的，与 OpenStack 类似，这使得二者易于结合。利用现有和/或新的 OpenStack 模块的能力将使您能够快速创建您的 playbook。

我们将从简要介绍 OpenStack 和 Ansible 开始，重点介绍一些最佳实践。接下来，每个后续章节的开头都将让您更加熟悉处理云操作员管理任务，如创建多个用户/租户、管理容器、自定义云配额、创建实例快照、设置主动-主动区域、运行云健康检查等。最后，每个章节都将以逐步教程结束，介绍如何使用 Ansible 自动化这些任务。作为额外的奖励，完全功能的 Ansible 代码将在 GitHub 上发布，供您在审阅章节时参考和/或以供以后审阅时参考。

将本书视为一次 2 合 1 的学习体验，深入了解基于 OpenStack 的云管理知识以及了解 Ansible 的工作原理。作为读者，您将被鼓励亲自动手尝试这些任务。

# 本书涵盖内容

第一章 *OpenStack 简介*，提供了 OpenStack 及构成该云平台的项目的高层概述。本介绍将为读者介绍 OpenStack 组件、概念和术语。

第二章 *Ansible 简介*，详细介绍了 Ansible 2.0，其特性和建立坚实起步基础的最佳实践。此外，它还将介绍为什么利用 Ansible 来自动化 OpenStack 任务是最简单的选择。

第三章 *创建多个用户/租户*，指导读者手动在 OpenStack 中创建用户和租户的过程，以及在使用 Ansible 自动化此过程时需要考虑的创建。

第四章 *自定义云配额*，让您了解配额是什么，以及它们如何用于限制您的云资源。它向读者展示了如何在 OpenStack 中手动创建配额。之后，它解释了如何使用 Ansible 自动化此过程，以便一次处理多个租户的任务。

第五章, *快照您的云*，教您如何在 OpenStack 内手动创建云实例的快照，以及如何使用 Ansible 自动化此过程。它探讨了一次性对一个租户内的所有实例进行快照的强大功能。

第六章, *迁移实例*，介绍了在传统的 OpenStack 方法中迁移选择实例到计算节点的概念。然后，它演示了自动化此任务所需的步骤，同时将实例分组，并展示了 Ansible 在处理此类任务时可以提供的其他选项。

第七章, *管理云上的容器*，带领读者了解如何自动化构建和部署在 OpenStack 云上运行的容器的一些策略。现在有几种方法可用，但关键是自动化该过程，使其成为可重复使用的功能。对于每种方法，本章展示了如何成功地使用 OpenStack 完成这些构建块。

第八章, *设置主动-主动区域*，详细审查了设置主动-主动 OpenStack 云区域的几个用例。有了这些知识，您将学会如何自动化部署到您的云。

第九章, *盘点您的云*，探讨了读者如何使用一个 Ansible playbook 动态盘点所有 OpenStack 云用户资源。它指导他们收集必要的指标以及如何将这些信息存储以供以后参考。这对于云管理员/操作员来说是一个非常强大的工具。

第十章, *使用 Nagios 检查您的云健康状况*，演示了如何手动检查云的健康状况以及如何利用 Ansible 设置 Nagios 和必要的检查来监视您的云的一些有用提示和技巧。Nagios 是领先的开源监控平台之一，并且与 OpenStack 和 Ansible 非常配合。

# 您需要为本书做好准备

要真正从本书中受益，最好部署或访问使用 openstack-ansible（OSA）构建的 OpenStack 云，该云运行 Newton 版本或更高版本。OSA 部署方法提供了一个环境，可以安装 OpenStack 和 Ansible。

如果您计划部署其他任何 OpenStack 发行版，您仍然只需要运行 OpenStack Newton 版本或更高版本。此外，您需要在相同节点上或您的工作站上安装 Ansible 版本 2.1 或更高版本。

另外，如果您计划添加或编辑 GitHub 存储库中找到的任何 Ansible playbooks/roles，拥有良好的文本编辑器，如 TextWrangler、Notepad++或 Vim，将非常有用。

# 这本书是为谁写的

如果您是基于 OpenStack 的云操作员和/或基础架构管理员，已经具有基本的 OpenStack 知识，并且有兴趣自动化管理功能，那么这本书正是您在寻找的。通过学习如何自动化简单和高级的 OpenStack 管理任务，您将把您的基本 OpenStack 知识提升到一个新的水平。拥有一个运行良好的 OpenStack 环境是有帮助的，但绝对不是必需的。

# 惯例

本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是这些样式的一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："我们可以从我们创建的名为`create-users-env`的角色开始。"

代码块设置如下：

```
- name: User password assignment 
 debug: msg="User {{ item.0 }} was added to {{ item.2 }} project, with the assigned password of {{ item.1 }}" 
 with_together: 
  - userid 
  - passwdss.stdout_lines 
  - tenantid 

```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目会以粗体显示：

```
- name: User password assignment 
 debug: msg="User {{ item.0 }} was added to {{ item.2 }} project, with the assigned password of {{ item.1 }}" 
 with_together: 
  - userid 
  **- passwdss.stdout_lines** 
  - tenantid 

```

任何命令行输入或输出都是这样写的：

```
**$ source openrc**
**$ openstack user create --password-prompt <username>**

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会在文本中以这种方式出现：“通过**Horizon**仪表板下的****Images****选项卡查看它们。”

### 注意

警告或重要说明会以这种方式出现在框中。

### 提示

提示和技巧会以这种方式出现。
