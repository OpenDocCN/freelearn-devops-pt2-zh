# 前言

近年来，微服务架构的趋势，将单片应用程序重构为多个微服务。容器简化了由微服务构建的应用程序的部署。容器管理、自动化和编排已成为关键问题。Kubernetes 就是为了解决这些问题而存在的。

本书是一本实用指南，提供逐步提示和示例，帮助您在私有和公共云中构建和运行自己的 Kubernetes 集群。跟着本书学习将使您了解如何在 Kubernetes 中部署和管理应用程序和服务。您还将深入了解如何扩展和更新实时容器，以及如何在 Kubernetes 中进行端口转发和网络路由。您将学习如何通过本书的实际示例构建一个稳健的高可用性集群。最后，您将通过集成 Jenkins、Docker 注册表和 Kubernetes 来构建一个持续交付流水线。

# 本书适合对象

如果您已经玩了一段时间的 Docker 容器，并希望以现代方式编排您的容器，那么这本书是您的正确选择。本书适合那些已经了解 Docker 和容器技术，并希望进一步探索以找到更好的方式来编排、管理和部署容器的人。本书非常适合超越单个容器，与容器集群一起工作，学习如何构建自己的 Kubernetes，并使其与您的持续交付流水线无缝配合。

# 本书涵盖内容

第一章《构建您自己的 Kubernetes 集群》解释了如何使用各种部署工具构建自己的 Kubernetes 集群，并在其上运行第一个容器。

第二章《走进 Kubernetes 概念》涵盖了我们需要了解的 Kubernetes 的基本和高级概念。然后，您将学习如何通过编写和应用配置文件来组合它们以创建 Kubernetes 对象。

第三章《玩转容器》解释了如何扩展和缩减容器，并执行滚动更新而不影响应用程序的可用性。此外，您将学习如何部署容器来处理不同的应用程序工作负载。它还将带您了解配置文件的最佳实践。

第四章，“构建高可用性集群”，提供了构建高可用性 Kubernetes 主节点和 etcd 的信息。这将防止 Kubernetes 组件成为单点故障。

第五章，“构建持续交付管道”，讨论了如何将 Kubernetes 集成到现有的使用 Jenkins 和私有 Docker 注册表的持续交付管道中。

第六章，“在 AWS 上构建 Kubernetes”，带您了解 AWS 的基础知识。您将学习如何在 AWS 上在几分钟内构建一个 Kubernetes 集群。

第七章，“在 GCP 上构建 Kubernetes”，带领您进入 Google Cloud Platform 世界。您将学习 GCP 的基本知识，以及如何通过几次点击启动一个托管的、可投入生产的 Kubernetes 集群。

第八章，“高级集群管理”，讨论了 Kubernetes 中的重要资源管理。本章还介绍了其他重要的集群管理，如 Kubernetes 仪表板、身份验证和授权。

第九章，“日志和监控”，解释了如何通过使用 Elasticsearch、Logstash 和 Kibana（ELK）在 Kubernetes 中收集系统和应用程序日志。您还将学习如何利用 Heapster、InfluxDB 和 Grafana 来监视您的 Kubernetes 集群。

# 充分利用本书

在整本书中，我们使用至少三台运行基于 Linux 的操作系统的服务器来构建 Kubernetes 中的所有组件。在书的开头，您可以使用一台机器，无论是 Linux 还是 Windows，来学习概念和基本部署。从可扩展性的角度来看，我们建议您从三台服务器开始，以便独立扩展组件并将集群推向生产级别。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

一旦文件下载完成，请确保您使用最新版本的解压软件解压文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Kubernetes-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Kubernetes-Cookbook-Second-Edition)。如果代码有更新，将在现有的 GitHub 存储库中更新。

我们还有其他代码包，来自我们丰富的书籍和视频目录，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 上找到。看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/KubernetesCookbookSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/KubernetesCookbookSecondEdition_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“准备以下 YAML 文件，这是一个简单的部署，启动两个 `nginx` 容器。”

代码块设置如下：

```
# cat 3-1-1_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```
Annotations:            deployment.kubernetes.io/revision=1
Selector:               env=test,project=My-Happy-Web,role=frontend
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
```

任何命令行输入或输出都以以下方式编写：

```
//install kubectl command by "kubernetes-cli" package
$ brew install kubernetes-cli
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“安装很简单，所以我们可以选择默认选项，然后点击“下一步”。”

警告或重要说明会出现在这样。提示和技巧会出现在这样。

# 章节

在本书中，您会经常看到几个标题（*准备工作*、*如何做…*、*它是如何工作的…*、*还有更多…* 和 *另请参阅*）。

为了清晰地说明如何完成一个配方，请使用以下部分：

# 准备工作

本节告诉您在配方中可以期待什么，并描述了为配方设置任何所需的软件或初步设置。

# 如何做…

这一部分包含了遵循食谱所需的步骤。

# 它是如何工作的...

这一部分通常包括对上一部分发生的事情的详细解释。

# 还有更多...

这一部分包括了有关食谱的额外信息，以使您对食谱更加了解。

# 另请参阅

这一部分提供了与食谱相关的其他有用信息的链接。
