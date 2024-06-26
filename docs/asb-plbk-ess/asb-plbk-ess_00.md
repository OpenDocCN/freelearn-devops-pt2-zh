# 前言

随着云计算、敏捷开发方法学的演进和近年来数据的爆炸性增长，管理大规模基础设施的需求日益增长。DevOps 工具和实践已经成为自动化此类可扩展、动态和复杂基础设施的每个阶段的必要条件。配置管理工具是这种 DevOps 工具集的核心。

Ansible 是一个简单、高效、快速的配置管理、编排和应用部署工具，集所有功能于一身。本书将帮助您熟悉编写 Playbooks，即 Ansible 的自动化语言。本书采用实践方法向您展示如何创建灵活、动态、可重用和数据驱动的角色。然后，它将带您了解 Ansible 的高级功能，如节点发现、集群化、使用保险库保护数据以及管理环境，最后向您展示如何使用 Ansible 来编排多层次基础架构堆栈。

# 本书内容概述

第一章，*为基础设施绘制蓝图*，将向您介绍 Playbooks、YAML 等内容。您还将了解 Playbook 的组成部分。

第二章, *使用 Ansible 角色进行模块化*，将演示如何使用 Ansible 角色创建可重用的模块化自动化代码，这些角色是自动化的单位。

第三章, *分离代码和数据 – 变量、事实和模板*，涵盖了使用模板和变量创建灵活、可定制、数据驱动的角色。您还将学习自动发现变量，即事实。

第四章，*引入您的代码 – 自定义命令和脚本*，涵盖了如何引入您现有的脚本，并使用 Ansible 调用 Shell 命令。

第五章，*控制执行流程 – 条件语句*，讨论了 Ansible 提供的控制结构，以改变执行方向。

第六章，*迭代控制结构 – 循环*，演示了如何使用强大的 with 语句来遍历数组、哈希等内容。

第七章，*节点发现和集群化*，讨论了拓扑信息的发现，并使用魔术变量和事实缓存创建动态配置。

第八章，*使用 Vault 加密数据*，讨论了使用 Ansible-vault 在版本控制系统中存储和共享的安全变量。

第九章，“管理环境”，涵盖了使用 Ansible 创建和管理隔离环境以及将自动化代码映射到软件开发工作流程中。

第十章，“使用 Ansible 编排基础设施”，涵盖了 Ansible 的编排功能，如滚动更新、预任务和后任务、标签、在 playbook 中构建测试等。

# 阅读本书所需材料

本书假设您已经安装了 Ansible，并且对 Linux/Unix 环境和系统操作有很好的了解，并且熟悉使用命令行接口。

# 本书的目标读者

本书的目标读者是系统或自动化工程师，具有数年管理基础设施各个部分经验，包括操作系统、应用配置和部署。本书也针对任何打算以最短的学习曲线有效自动化管理系统和应用配置的人群。

假设读者对 Ansible 有概念性的了解，已经安装过，并熟悉基本操作，比如创建清单文件和使用 Ansible 运行临时命令。

# 约定

在本书中，您会发现一些不同类型信息的文本样式。以下是这些样式的一些示例，以及其含义的解释。

文本中的代码单词显示如下：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```
---
# site.yml : This is a sitewide playbook
- include: www.yml
```

任何命令行输入或输出会写成如下形式：

```
$ ansible-playbook simple_playbook.yml -i customhosts

```

**新术语**和**重要词汇**都显示为加粗。例如，屏幕上看到的文本、菜单或对话框中的单词会像这样出现在文本中：“结果变量哈希应包含**defaults**中的项目以及**vars**中的覆盖值”。

### 注意

警告或重要提示会显示为这样的框。

### 小贴士

贴士和技巧将以这种方式出现。

# 读者反馈

我们非常欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们开发您能够充分利用的标题非常重要。

要给我们发送一般性反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在标题中提及书名。

如果您对某个专题非常了解，并且有兴趣写作或为书籍做出贡献，请查看我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事项可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)购买的所有 Packt 图书中下载示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接将文件通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误还是会发生。如果您在我们的任何一本书中发现错误——可能是文字或代码方面的错误——我们将不胜感激地接受您的报告。通过这样做，您可以帮助其他读者避免困惑，并帮助我们改进后续版本的本书。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，然后输入您的勘误详细信息。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站上，或者被添加到该书籍的任何现有勘误列表中，位于该标题的勘误部分下面。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的书籍来查看任何现有的勘误。

## 盗版

互联网上的版权盗版是各种媒体上持续存在的问题。在 Packt，我们非常重视我们的版权和许可的保护。如果您在互联网上发现我们任何形式的作品的非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过链接<copyright@packtpub.com>与我们联系，报告涉嫌盗版材料。

我们感谢您帮助保护我们的作者，以及我们为您提供有价值内容的能力。

## 问题

如果您在阅读本书的任何方面遇到问题，请通过链接<questions@packtpub.com>与我们联系，我们将尽力解决。
