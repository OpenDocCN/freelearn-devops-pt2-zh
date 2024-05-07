# 前言

IT 正在经历一次巨大的范式转变。从以正常运行时间作为 IT 成功的衡量标准的时代，我们正在转向不可变基础设施的理念，根据需求，我们可以自动地随时启动和销毁服务器。Ansible 在这种转变中扮演着主导角色。它已经成为各大公司和小公司选择的工具，用于单个服务器到整个集群的任务。

本书介绍了安全自动化。我们运用对 Ansible 的知识到不同的场景和工作负载中，这些场景和工作负载围绕着安全展开，因此得名。当无聊和单调的任务被自动化时，做这些任务的人可以集中精力解决他们所面对的安全问题。这为我们学习安全（培训）的方式、我们可以存储、处理和分析日志数据的数量（DFIR）、我们如何可以在没有任何中断的情况下保持应用安全更新（安全运营），以及更多内容提供了全新的观点。

在本书中，我们将分享使用 Ansible 可以实现的各种自动化类型的经验。你可能对其中一些内容很熟悉，或者对你来说它们可能是全新的。无论如何，我们希望你不要试图规定应该如何使用 Ansible，而是希望你阅读并理解如何使用每个 playbook/工作流程，使你的安全工作更快、更好、更可靠，或者只是为自己或他人创建复杂的基础架构场景而感到开心。

如果没有 Red Hat Ansible 的同事以及其他无数的博客和项目提供的优秀文档，本书将不可能问世，他们已经创建了安全、具有弹性的 playbooks，我们都可以从中学习并使用。

本书分为三个主要部分：

+   构建有用的 playbook 所需的基本 Ansible

+   安全自动化技术和方法

+   扩展和编程 Ansible 以获得更多的安全性

我们的目标是让你快速更新你对 Ansible 的知识，然后让你变得对它更加高效，最后，你将看到如何通过扩展 Ansible 或创建你自己的安全模块来做更多事情

# 这本书涵盖的内容

第一章，*介绍 Ansible Playbooks 和 Roles*，介绍了您在 Ansible 中可能已经熟悉的术语。它们通过示例 playbooks 和运行这些 playbooks 所需的 Ansible 命令进行了解释。如果你觉得你的 Ansible 概念和技能有点生疏，可以从这里开始。

第二章，*Ansible Tower、Jenkins 和其他自动化工具*，全都是关于自动化的自动化。我们涵盖了与 Ansible 一起常用的调度自动化工具的使用，例如 Ansible Tower、Jenkins 和 Rundeck。如果您开始使用这些工具，那么记住何时安排和执行 playbooks 以及获取有关输出的通知等单调乏味的任务可以委托给这些工具，而不是留在您的头脑中。如果您还没有使用过这些工具，您应该阅读本章。

第三章，*使用加密自动备份设置加固的 WordPress*，涵盖了各种安全自动化技术和方法的探索。与任何技术或方法一样，我们所说的一些内容可能并不适用于您的用例。然而，通过采取一种主观的方法，我们向您展示了一种我们认为在很大程度上运行良好的方法。WordPress 是目前最流行的网站创建软件。通过使用 playbooks（并在 IT 自动化工具中运行），我们开始讨论一个 IT/ops 需求，即保持正在运行的服务器安全，并确保我们可以从故障中恢复。如果您负责管理网站（即使只是您自己的网站），这一章应该是有用的。如果您不使用 WordPress，在本章中有足够的内容让您思考如何将本章应用于您的用例。

第四章，*日志监视和无服务器自动防御（AWS 中的 Elastic Stack）*，涵盖了日志监视和安全自动化，就像花生酱和果冻一样。在本章中，我们使用 Ansible 在 AWS 中的服务器上设置日志监视服务器基础架构。基于攻击通知，我们使用 AWS Lambda、Dynamo DB 和 AWS Cloudwatch 等 AWS 服务创建一个几乎实时的动态防火墙服务。

第五章，*使用 OWASP ZAP 自动化 Web 应用程序安全测试*，涵盖了测试网站安全性的最常见安全工作流程之一，即使用最流行的开源工具之一 OWASP ZAP。一旦我们弄清了基本工作流程，我们就会通过 Ansible 和 Jenkins 对您的网站进行持续扫描，使其超级强大。阅读本章，了解如何使用 Ansible 在处理持续安全扫描时与 Docker 容器一起工作。这是一个确保双赢的策略！

第六章，*使用 Nessus 进行漏洞扫描*，解释了使用 Nessus 与 Ansible 进行漏洞扫描的方法。本章涵盖了进行基本网络扫描、进行安全补丁审核和枚举漏洞的方法。

第七章，*应用程序和网络的安全加固*，展示了 Ansible 已经使我们能够以声明方式表达我们的安全思想。通过利用系统状态应该是什么的想法，我们可以基于标准（如 CIS 和 NIST）以及美国国防部的 STIGs 提供的指南创建安全加固剧本。熟悉使用现有安全文档来加固应用程序和服务器的方法，但最重要的是，以可重复自描述的方式进行，这是在版本控制下进行的。如果你和我们一样，多年来一直手动执行所有这些任务，你会很感激这对安全自动化的改变。

第八章，*Docker 容器的持续安全扫描*，介绍了如何对 Docker 容器运行安全扫描工具。许多现代应用程序都使用容器部署，本章将帮助你快速了解是否有容器存在漏洞，并且一如既往地，与 Ansible Tower 结合使用，如何使这成为一个持续的过程。

第九章，*自动设置实验室进行取证收集、恶意软件分析*，特别适用于恶意软件研究人员。如果你一直想使用 Cuckoo 沙箱和 MISP，但因为设置这些工具涉及的步骤太复杂而望而却步，那么本章将为你提供帮助。

第十章，*编写用于安全测试的 Ansible 模块*，介绍了我们如何扩展 Ansible 提供的功能，并学习其他项目如何使用 Ansible 提供优秀的软件解决方案。本章和下一章，带领我们进入本书的第三部分。

有时候，尽管 Ansible 自带了许多惊人的模块，但它们仍然不足以满足我们的需求。本章探讨了创建 Ansible 模块，如果我们可以这么说的话，它并不试图对方法非常正式。记住我们想要关注的是安全自动化，我们创建了一个模块来运行使用 ZAP 代理进行网站安全扫描。提供了完整的模块，这将帮助你在很短的时间内编写和使用你自己的模块。

第十一章，*Ansible 安全最佳实践、参考和进一步阅读*，介绍了如何使用 Ansible Vault 管理密码和凭证。它将帮助你建立自己的 Ansible Galaxy 实例。我们还介绍了其他使用 Ansible 剧本进行安全解决方案的项目，如 DebOps 和 Algo。我们还介绍了 AWX，这是 Ansible Tower 的免费开源版本，并向你展示如何设置和使用它。最后，我们简要讨论了预计在 2018 年第一或第二季度发布的 Ansible 2.5 版本。

# 为了阅读本书，你需要什么

Ansible 是一种用 Python2 编写的工具。对于控制机器，如果 Python2 安装了最低版本 2.6，那就没问题了。自 Ansible 2.2 起，Python3 作为技术预览版本得到支持。

# 这本书适合谁

这本书最理想的读者是那些明白自动化是实现可重复、无错误部署和基础架构、应用程序和网络配置的关键的人。不过，我们确实想要指明这一点。

如果您是负责网站、服务器和网络安全的系统管理员，那么这本书适合您。

安全顾问和分析师将通过专注于 [第三章](https://cdp.packtpub.com/security_automation_with_ansible_2/wp-admin/post.php?post=23&action=edit#post_72)，*设置具有加密自动备份的强化 WordPress*，到 [第十章](https://cdp.packtpub.com/security_automation_with_ansible_2/wp-admin/post.php?post=23&action=edit#post_442)，*编写用于安全测试的 Ansible 模块* 而受益。即使某些工作负载不适用于您，您也将了解如何使用 Ansible 为团队提供安全服务的见解。所有 DevOps 团队都愿意与将自动化视为与安全本身一样重要的人一起工作。

希望轻松部署安全服务器的应用程序开发人员尤其应该查看 [第三章](https://cdp.packtpub.com/security_automation_with_ansible_2/wp-admin/post.php?post=23&action=edit#post_72)，*设置具有加密自动备份的强化 WordPress*，到 [第七章](https://cdp.packtpub.com/security_automation_with_ansible_2/wp-admin/post.php?post=23&action=edit#post_265)，*应用程序和网络的安全强化*。

如果您是以下人员之一，您将从这本书中受益最多：

+   之前使用过 Ansible 基本命令的人

+   对 Linux 和 Windows 操作系统熟悉的人。

+   对 IP 地址、网络和软件安装程序有基本了解的人。

# 惯例

在本书中，您将找到一些区分不同类型信息的文本样式。以下是这些样式的一些示例及其含义的解释。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“`harden.yml` 执行 MySQL 服务器配置的加固” 代码块设置如下：

```
- name: deletes anonymous mysql user
  mysql_user:
    user: ""
    state: absent
    login_password: "{{ mysql_root_password }}"
    login_user: root
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```
- name: deletes anonymous mysql user
  mysql_user:
    user: ""
    state: absent
    login_password: "{{ mysql_root_password }}"
    login_user: root
```

任何命令行输入或输出写成如下格式：

```
ansible-playbook -i inventory playbook.yml
```

**新术语** 和 **重要单词** 以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中：“点击 确认安全异常 并继续执行安装步骤”。

警告或重要提示会以这种方式出现。

提示和技巧会以这种方式出现。

# 读者反馈

我们的读者的反馈意见始终受到欢迎。告诉我们您对这本书的看法 - 您喜欢或不喜欢什么。读者的反馈对我们很重要，因为它帮助我们开发您真正能充分利用的标题。要向我们发送一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在您的消息主题中提到书名。如果您在某个专题上有专业知识，并且您有兴趣撰写或贡献一本书，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您是 Packt 书籍的自豪所有者，我们有很多东西可以帮助您充分利用您的购买。

# 下载示例代码

你可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以直接将文件通过电子邮件发送给你。按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 选项卡上。

1.  单击“代码下载与勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择购买本书的位置。

1.  单击“代码下载”。

下载文件后，请确保使用以下最新版本的解压缩或提取文件夹：

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

本书的代码捆绑包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/Security-Automation-with-Ansible-2`](https://github.com/PacktPublishing/Security-Automation-with-Ansible-2)。我们还提供了来自我们丰富书籍和视频目录的其他代码捆绑包，地址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。来看看吧！

# 下载本书的彩色图像

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/SecurityAutomationwithAnsible2_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/SecurityAutomationwithAnsible2_ColorImages.pdf)下载此文件。

# 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误还是会发生。如果您在我们的书籍中发现错误——也许是文字或代码上的错误——我们将不胜感激地请您向我们报告。通过这样做，您可以避免其他读者的困扰，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击"勘误提交表格"链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该书籍的"勘误"部分的任何现有勘误列表中。要查看之前提交的勘误，请转至[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在"勘误"部分下面。

# 盗版

互联网上的版权物资盗版是跨所有媒体持续存在的问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上的任何形式上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。请通过`copyright@packtpub.com`与我们联系，并附上可疑盗版材料的链接。感谢您帮助我们保护我们的作者和我们为您带来有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，请通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
