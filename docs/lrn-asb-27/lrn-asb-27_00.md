# 前言

信息技术领域是一个快速发展的领域，始终试图加速。为了跟上这一步伐，公司需要能够快速行动并频繁迭代。直到几年前，这主要适用于软件，但现在我们开始看到以类似速度开发基础设施的必要性。未来，我们将需要以软件本身的速度改变我们运行软件的基础设施。

在这种场景下，许多技术（如软件定义的一切，例如存储、网络和计算）将至关重要，但这些技术需要以同样可扩展的方式进行管理，而这种方式将涉及使用 Ansible 和类似产品。

今天，Ansible 非常相关，因为与竞争产品不同，它是无代理的，可以实现更快的部署、更高的安全性和更好的可审计性。

# 本书适用对象

本书适用于希望使用 Ansible 2 自动化其组织基础架构的开发人员和系统管理员。不需要有关 Ansible 的先前知识。

# 本书涵盖内容

第一章，*开始使用 Ansible*，讲解了如何安装 Ansible。

第二章，*自动化简单任务*，讲解了如何创建简单的 Playbook，让您能够自动化一些您每天已经执行的简单任务。

第三章，*扩展至多个主机*，讲解了如何以易于扩展的方式处理 Ansible 中的多个主机。

第四章，*处理复杂部署*，讲解了如何创建具有多个阶段和多台机器的部署。

第五章，*走向云端*，讲解了 Ansible 如何与各种云服务集成，以及如何简化您的生活，为您管理云端。

第六章，*从 Ansible 获取通知*，讲解了如何设置 Ansible 以向您和其他利益相关者返回有价值的信息。

第七章，*创建自定义模块*，讲解了如何创建自定义模块以利用 Ansible 给予你的自由。

第八章，*调试和错误处理*，讲解了如何调试和测试 Ansible 以确保您的 Playbook 总是有效的。

第九章，*复杂环境*，讲解了如何使用 Ansible 管理多个层次、环境和部署。

第十章，*介绍企业级 Ansible*，讲解了如何从 Ansible 管理 Windows 节点，以及如何利用 Ansible Galaxy 来最大化您的生产力。

第十一章，*开始使用 AWX*，解释了 AWX 是什么以及如何开始使用它。

第十二章，*与 AWX 用户、权限和组织一起工作*，解释了 AWX 用户和权限管理的工作原理。

# 要充分利用本书

本书假定您具有 UNIX shell 的基本知识，以及基本的网络知识。

# 下载示例代码文件

您可以从 [www.packt.com](http://www.packt.com) 的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问 [www.packt.com/support](http://www.packt.com/support) 并注册，以便将文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册，网址为 [www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载完成后，请确保使用最新版本的软件解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition`](https://github.com/PacktPublishing/Learning-Ansible-2.X-Third-Edition)。如果代码有更新，将在现有的 GitHub 存储库中更新。

我们还有来自丰富书籍和视频目录的其他代码包可供使用。请查看：**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。以下是一个示例："`sudo` 命令是一个众所周知的命令，但通常以更危险的形式使用。"

代码块设置如下：

```
- hosts: all 
  remote_user: vagrant
  tasks: 
    - name: Ensure the HTTPd package is installed 
      yum: 
        name: httpd 
        state: present 
      become: True 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项将加粗显示：

```
- hosts: all 
  remote_user: vagrant
  tasks: 
    - name: Ensure the HTTPd package is installed 
      yum: 
        name: httpd 
        state: present 
      become: True
```

任何命令行输入或输出都按如下方式编写：

```
$ sudo dnf install ansible
```

警告或重要提示如下。

技巧和提示如下。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并发送邮件至`customercare@packtpub.com`。

**勘误**：尽管我们已尽一切努力确保内容准确无误，但错误难免会发生。如果您在本书中发现错误，我们将不胜感激。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何形式的非法副本，我们将不胜感激您提供位置地址或网站名称。请通过`copyright@packt.com`联系我们，并提供材料链接。

**如果您有兴趣成为作者**：如果您有专业知识并对撰写或为书籍做贡献感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评论。当您阅读并使用了本书后，为何不在您购买的网站上留下评论呢？潜在的读者可以看到并使用您的客观意见来做出购买决定，我们在 Packt 公司可以了解您对我们产品的看法，而我们的作者也可以看到您对他们的书的反馈。谢谢！

欲了解有关 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/)。
