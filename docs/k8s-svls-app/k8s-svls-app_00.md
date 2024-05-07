# 前言

Kubernetes 是近年来突出的技术之一；它已被所有主要公共云提供商采用为容器集群和编排平台，并迅速成为行业标准。

再加上 Kubernetes 是开源的，您就拥有了在多个公共和私有提供商上托管自己的平台即服务或 PaaS 的完美基础；您甚至可以在笔记本电脑上运行它，并且由于其设计，您将在所有平台上获得一致的体验。

它的设计也使其成为运行无服务器函数的理想平台。在本书中，我们将研究几个可以部署在 Kubernetes 上并与之集成的平台，这意味着我们不仅拥有 PaaS，还有一个强大的 FaaS 平台在您的 Kubernetes 环境中运行。

# 本书适合对象

本书主要面向运维工程师、云架构师和开发人员，他们希望在 Kubernetes 集群上托管他们的无服务器函数。

# 本书内容

第一章，*无服务器景观*，解释了什么是无服务器。此外，我们将在公共云上使用 AWS Lambda 和 Azure Functions 来运行无服务器函数，获得一些实际经验。

第二章，*Kubernetes 简介*，讨论了 Kubernetes 是什么，它解决了什么问题，并且还回顾了它的背景，从谷歌的内部工程工具到开源强大工具。

第三章，*在本地安装 Kubernetes*，解释了如何通过实践经验来使用 Kubernetes。我们将使用 Minikube 安装本地单节点 Kubernetes 集群，并使用命令行客户端与其交互。

第四章，*介绍 Kubeless 函数*，解释了在本地运行 Kubernetes 后如何使用 Kubeless 启动第一个无服务器函数。

第五章，*使用 Funktion 进行无服务器应用*，解释了使用 Funktion 来调用无服务器函数的一种略有不同的方法。

第六章，*在云中安装 Kubernetes*，介绍了在 DigitalOcean、AWS、Google Cloud 和 Microsoft Azure 上启动集群的过程，以及在本地使用 Kubernetes 进行一些实践经验后的操作。

第七章，*Apache OpenWhisk 和 Kubernetes*，解释了如何在我们新推出的云 Kubernetes 集群上启动、配置和使用最初由 IBM 开发的无服务器平台 Apache OpenWhisk。

第八章，*使用 Fission 启动应用程序*，介绍了部署 Fission 的过程，这是 Kubernetes 的流行无服务器框架，并附带了一些示例函数。

第九章，*了解 OpenFaaS*，介绍了 OpenFaaS。虽然它首先是一个用于 Docker 的函数即服务框架，但也可以部署在 Kubernetes 之上。

第十章，*无服务器考虑因素*，讨论了安全最佳实践以及如何监视您的 Kubernetes 集群。

第十一章，*运行无服务器工作负载*，解释了 Kubernetes 生态系统的快速发展以及您如何跟上。我们还讨论了您应该使用哪些工具，以及为什么您希望将无服务器函数部署在 Kubernetes 上。

# 为了充分利用本书

**操作系统**：

+   macOS High Sierra

+   Ubuntu 17.04

+   Windows 10 专业版

**软件**：

在本书中，我们将安装几个命令行工具；每个工具都将在各章节中提供安装说明和其要求的详细信息。请注意，虽然提供了 Windows 系统的说明，但我们将主要使用最初设计为在 Linux/Unix 系统上运行的工具，如 Ubuntu 17.04 和 macOS High Sierra，并且本书将偏向这些系统。虽然在撰写时已尽最大努力验证这些工具在基于 Windows 的系统上的运行情况，但由于一些工具是实验性构建的，我们无法保证它们在更新的系统上仍然能够正常工作，因此，我建议使用 Linux 或 Unix 系统。

硬件：

+   **Windows 10 专业版和 Ubuntu 17.04 系统要求**：

+   使用 2011 年或之后推出的处理器（CPU），核心速度为 1.3 GHz 或更快，除了基于*Llano*和*Bobcat*微架构的英特尔 Atom 处理器或 AMD 处理器

+   最低 4 GB RAM，建议使用 8 GB RAM 或更多

+   **Apple Mac 系统要求**：

+   iMac：2009 年底或更新

+   MacBook/MacBook（Retina）：2009 年底或更新

+   **MacBook Pro**：2010 年中期或更新

+   **MacBook Air**：2010 年末或更新版本

+   **Mac mini**：2010 年中期或更新版本

+   **Mac Pro**：2010 年中期或更新版本

**至少可以访问以下公共云服务之一**：

+   **AWS**：[`aws.amazon.com/`](https://aws.amazon.com/)

+   **Google Cloud**：[`cloud.google.com/`](https://cloud.google.com/)

+   **Microsoft Azure**：[`azure.microsoft.com/`](https://azure.microsoft.com/)

+   **DigitalOcean**：[`www.digitalocean.com/`](https://www.digitalocean.com/)

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的以下软件解压缩文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Kubernetes-for-Serverless-Applications`](https://github.com/PacktPublishing/Kubernetes-for-Serverless-Applications)。我们还有其他丰富的图书和视频的代码包可供下载，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。请查看！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/KubernetesforServerlessApplications_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/KubernetesforServerlessApplications_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：指示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。这是一个例子：“这包含一个名为`index.html`的单个文件。”

一段代码设置如下：

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: cli-hello-world
  labels:
    app: nginx
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目以粗体设置：

```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: cli-hello-world
  labels:
    app: nginx
```

任何命令行输入或输出都将按照以下方式编写：

```
$ brew cask install minikube
```

**粗体**：表示一个新术语，一个重要的词，或者你在屏幕上看到的词。例如，菜单或对话框中的单词会在文本中以这种方式出现。这是一个例子：“在页面底部，您将有一个按钮，可以让您为您的帐户创建访问令牌和访问令牌密钥。”

警告或重要提示会以这种方式出现。提示和技巧会以这种方式出现。
