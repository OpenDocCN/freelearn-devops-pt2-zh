# 前言

*使用 Kubernetes 进行微服务实践*是您一直在等待的书籍。它将引导您同时开发微服务并将其部署到 Kubernetes 上。微服务架构与 Kubernetes 之间的协同作用非常强大。本书涵盖了所有方面。它解释了微服务和 Kubernetes 背后的概念，讨论了现实世界中的问题和权衡，带您完成了完整的基于微服务的系统开发，展示了最佳实践，并提供了充分的建议。

本书深入浅出地涵盖了大量内容，并提供了工作代码来说明。您将学习如何设计基于微服务的架构，构建微服务，测试您构建的微服务，并将它们打包为 Docker 镜像。然后，您将学习如何将系统部署为一组 Docker 镜像到 Kubernetes，并在那里进行管理。

在学习的过程中，您将熟悉最重要的趋势，如自动化的持续集成/持续交付（CI/CD），基于 gRPC 的微服务，无服务器计算和服务网格。

通过本书，您将获得大量关于规划、开发和操作基于微服务架构部署在 Kubernetes 上的大规模云原生系统的知识和实践经验。

# 本书适合对象

本书面向希望成为大规模软件工程前沿人员的软件开发人员和 DevOps 工程师。如果您有使用容器部署在多台机器上并由多个团队开发的大规模软件系统的经验，将会有所帮助。

# 本书涵盖内容

第一章，*面向开发人员的 Kubernetes 简介*，向您介绍了 Kubernetes。您将快速了解 Kubernetes，并了解其与微服务的契合程度。

第二章，*微服务入门*，讨论了微服务架构中常见问题的各个方面、模式和方法，以及它们与其他常见架构（如单体架构和大型服务）的比较。

第三章，*Delinkcious – 示例应用*，探讨了为什么我们应该选择 Go 作为 Delinkcious 的编程语言；然后我们将看看 Go kit。

第四章《设置 CI/CD 流水线》教你了解 CI/CD 流水线解决的问题，涵盖了 Kubernetes 的 CI/CD 流水线的不同选项，最后看看如何为 Delinkcious 构建 CI/CD 流水线。

第五章《使用 Kubernetes 配置微服务》将您带入微服务配置的实际和现实世界领域。此外，我们将讨论 Kubernetes 特定的选项，特别是 ConfigMaps。

第六章《在 Kubernetes 上保护微服务》深入探讨了如何在 Kubernetes 上保护您的微服务。我们还将讨论作为 Kubernetes 上微服务安全基础的支柱。

第七章《与世界交流- API 和负载均衡器》让我们向世界开放 Delinkcious，并让用户可以在集群外与其进行交互。此外，我们将添加一个基于 gRPC 的新闻服务，用户可以使用它获取关注的其他用户的新闻。最后，我们将添加一个消息队列，让服务以松散耦合的方式进行通信。

第八章《处理有状态服务》深入研究了 Kubernetes 的存储模型。我们还将扩展 Delinkcious 新闻服务，将其数据存储在 Redis 中，而不是在内存中。

第九章《在 Kubernetes 上运行无服务器任务》深入探讨了云原生系统中最热门的趋势之一：无服务器计算（也称为函数即服务，或 FaaS）。此外，我们将介绍在 Kubernetes 中进行无服务器计算的其他方法。

第十章《测试微服务》涵盖了测试及其各种类型：单元测试、集成测试和各种端到端测试。我们还深入探讨了 Delinkcious 测试的结构。

第十一章《部署微服务》涉及两个相关但分开的主题：生产部署和开发部署。

第十二章《监控、日志和指标》关注在 Kubernetes 上运行大规模分布式系统的操作方面，以及如何设计系统以及需要考虑的因素，以确保卓越的操作姿态。

第十三章，*服务网格-使用 Istio*，审查了服务网格的热门话题，特别是 Istio。这很令人兴奋，因为服务网格是一个真正的游戏改变者。

第十四章，*微服务和 Kubernetes 的未来*，涵盖了 Kubernetes 和微服务的主题，将帮助我们学习如何决定何时是采用和投资新技术的正确时机。

# 充分利用本书

任何软件要求要么列在每章的*技术要求*部分开头，要么，如果安装特定软件是本章材料的一部分，那么您需要的任何说明将包含在章节本身中。大多数安装都是安装到 Kubernetes 集群中的软件组件。这是本书实践性的重要部分。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，文件将直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择 SUPPORT 选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的解压软件解压文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Microservices-with-Kubernetes`](https://github.com/PacktPublishing/Hands-On-Microservices-with-Kubernetes)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含了本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`static.packt-cdn.com/downloads/9781789805468_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781789805468_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“请注意，我确保它通过`chmod +x`是可执行的。”

代码块设置如下：

```
version: 2
jobs:
  build:
    docker:
    - image: circleci/golang:1.11
    - image: circleci/postgres:9.6-alpine
```

任何命令行输入或输出都是按照以下方式编写的：

```
$ tree -L 2
.
├── LICENSE
├── README.md
├── build.sh
```

**粗体**：表示一个新术语、一个重要词或者你在屏幕上看到的词。例如，菜单或对话框中的词会以这种方式出现在文本中。这是一个例子：“我们可以通过从 ACTIONS 下拉菜单中选择同步来同步它。”

警告或重要提示会出现在这样的地方。提示和技巧会出现在这样的地方。
