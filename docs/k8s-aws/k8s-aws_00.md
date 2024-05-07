# 前言

Docker 容器承诺彻底改变开发人员和运维在云上构建、部署和管理应用程序的方式。Kubernetes 提供了您在生产环境中实现这一承诺所需的编排工具。

《Kubernetes on AWS》指导您在**Amazon Web Services**（**AWS**）平台上部署一个生产就绪的 Kubernetes 集群。您将了解如何使用 Kubernetes 的强大功能，它是最快增长的生产容器编排平台之一，用于管理和更新您的应用程序。Kubernetes 正在成为云原生应用程序生产级部署的首选。本书从最基本的原理开始介绍 Kubernetes。您将首先学习 Kubernetes 的强大抽象——pod 和 service，这使得管理容器部署变得容易。接着，您将通过在 AWS 上设置一个生产就绪的 Kubernetes 集群的指导之旅，同时学习成功部署和管理自己应用程序的技术。

通过本书，您将在 AWS 上获得丰富的 Kubernetes 实践经验。您还将学习到一些关于部署和管理应用程序、保持集群和应用程序安全以及确保整个系统可靠且具有容错性的技巧。

# 本书适合对象

如果您是云工程师、云解决方案提供商、系统管理员、网站可靠性工程师或对 DevOps 感兴趣的开发人员，并且希望了解在 AWS 环境中运行 Kubernetes 的详尽指南，那么本书适合您。虽然不需要对 Kubernetes 有任何先前的了解，但具有 Linux 和 Docker 容器的一些经验将是一个优势。

# 本书涵盖内容

第一章，*Google 的基础设施适用于我们其余人*，帮助您了解 Kubernetes 如何为您提供谷歌的可靠性工程师使用的一些超能力，以确保谷歌的服务具有容错性、可靠性和高效性。

第二章，*启动引擎*，帮助您开始使用 Kubernetes。您将学习如何在自己的工作站上启动适用于学习和开发的集群，并开始学习如何使用 Kubernetes 本身。

第三章，《触及云端》，教您如何从头开始构建在 AWS 上运行的 Kubernetes 集群。

第四章，《管理应用程序中的更改》，深入探讨了 Kubernetes 提供的工具，用于管理在集群上运行的 Pod。

第五章，《使用 Helm 管理复杂应用程序》，教您如何使用社区维护的图表将服务部署到您的集群。

第六章，《生产规划》，让您了解在决定在生产环境中运行 Kubernetes 时可以做出的多种不同选择和决策。

第七章，《生产就绪集群》，帮助您构建一个完全功能的集群，这将作为一个基本配置，用于许多不同的用例。

第八章，《抱歉，我的应用程序吃掉了集群》，深入探讨了使用不同的服务质量配置 Pod，以便重要的工作负载能够保证它们所需的资源，但不太重要的工作负载可以在有空闲资源时利用专用资源而无需专门的资源。

第九章，《存储状态》，全都是关于使用 Kubernetes 与 AWS 原生存储解决方案弹性块存储（EBS）的深度集成。

第十章，《管理容器镜像》，帮助您了解如何利用 AWS 弹性容器注册表（ECR）服务以满足所有这些需求存储您的容器镜像。

第十一章，《监控和日志记录》，教您如何设置日志管理管道，并将帮助您了解日志的一些潜在问题和潜在问题。到本章结束时，您将已经设置了一个指标和警报系统。有关本章，请参阅[`www.packtpub.com/sites/default/files/downloads/Monitoring_and_Logging.pdf`](https://www.packtpub.com/sites/default/files/downloads/Monitoring_and_Logging.pdf)。

[第十二章](https://www.packtpub.com/sites/default/files/downloads/Best_Practices_of_Security.pdf)，*安全最佳实践*，教您如何使用 AWS 和 Kubernetes 网络原语管理 Kubernetes 集群的安全网络。 您还将学习如何保护您的主机操作系统。 有关本章，请参阅[`www.packtpub.com/sites/default/files/downloads/Best_Practices_of_Security.pdf`](https://www.packtpub.com/sites/default/files/downloads/Best_Practices_of_Security.pdf)。

# 为了充分利用本书

您需要访问 AWS 帐户以执行本书中给出的示例。

# 下载示例代码文件

您可以从您在[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。 如果您在其他地方购买了本书，可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名并按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Kubernetes-on-AWS`](https://github.com/PacktPublishing/Kubernetes-on-AWS)。 如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。 快去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄。 例如："将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。"

代码块设置如下：

```
html, body, #map {
 height: 100%; 
 margin: 0;
 padding: 0
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都以以下方式编写：

```
$ mkdir css
$ cd css
```

**粗体**：表示一个新术语，一个重要词或者屏幕上看到的词。例如，菜单或对话框中的词会在文本中显示为这样。这是一个例子：“从管理面板中选择系统信息。”

警告或重要提示会显示为这样。提示和技巧会显示为这样。
