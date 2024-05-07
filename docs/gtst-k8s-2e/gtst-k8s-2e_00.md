# 前言

本书是 Kubernetes 和整体容器管理入门指南。我们将为您介绍 Kubernetes 的特性和功能，并展示它如何融入整体运营策略。您将了解将容器从开发人员的笔记本移到更大规模时面临的障碍。您还将看到 Kubernetes 如何是帮助您自信面对这些挑战的完美工具。

# 本书内容

第一章，*Kubernetes 简介*，简要介绍容器和 Kubernetes 编排的如何、什么和为什么，探讨它如何影响您的业务目标和日常操作。

第二章，*Pods、服务、复制控制器和标签*，使用几个简单的示例来探索核心 Kubernetes 构造，即 Pod、服务、复制控制器、副本集和标签。还将涵盖基本操作，包括健康检查和调度。

第三章，*网络、负载均衡器和入口*，涵盖了 Kubernetes 的集群网络和 Kubernetes 代理。它还深入探讨了服务，最后，它展示了一些更高级别的隔离特性的简要概述，用于多租户。

第四章，*更新、渐进式部署和自动缩放*，快速了解如何在最小化停机时间的情况下推出更新和新功能。我们还将研究应用程序和 Kubernetes 集群的扩展。

第五章，*部署、任务和 DaemonSets*，涵盖了长时间运行的应用程序部署以及短期任务。我们还将研究使用 DaemonSets 在集群中的所有或子集节点上运行容器。

第六章，*存储和运行有状态的应用程序*，涵盖了跨 Pod 和容器生命周期的存储问题和持久数据。我们还将研究在 Kubernetes 中使用有状态应用程序的新构建。

第七章，*持续交付*，解释了如何将 Kubernetes 集成到您的持续交付流水线中。我们将看到如何使用 Gulp.js 和 Jenkins 来使用 k8s 集群。

第八章，*监控与日志*，教你如何在你的 Kubernetes 集群上使用和定制内置和第三方监控工具。我们将研究内置日志记录和监控、Google Cloud 监控/日志服务以及 Sysdig。

第九章，*集群联邦*，使您能够尝试新的联邦功能，并解释如何使用它们来管理跨云提供商的多个集群。我们还将涵盖前几章中核心构造的联邦版本。

第十章，*容器安全*，从容器运行时层级到主机本身，教授容器安全的基础知识。它还解释了如何将这些概念应用于运行容器，以及与运行 Kubernetes 相关的一些安全问题和做法。

第十一章，*使用 OCP、CoreOS 和 Tectonic 扩展 Kubernetes*，探讨了开放标准如何使整个容器生态系统受益。我们将介绍一些知名的标准组织，并涵盖 CoreOS 和 Tectonic，探讨它们作为主机操作系统和企业平台的优势。

第十二章，*走向生产就绪*，最后一章，展示了一些有用的工具和第三方项目，以及您可以获取更多帮助的地方。

# 本书所需内容

本书将涵盖下载和运行 Kubernetes 项目。您需要访问 Linux 系统（如果您使用 Windows，VirtualBox 也可行），并对命令行有一定了解。

此外，您应该有一个 Google Cloud Platform 账户。您可以在此处注册免费试用：

[`cloud.google.com/`](https://cloud.google.com/)

此外，本书的一些部分需要有 AWS 账户。您可以在此处注册免费试用：

[`aws.amazon.com/`](https://aws.amazon.com/)

# 本书适合谁

无论您是深入开发、深入运维，还是作为执行人员展望未来，Kubernetes 和本书都适合您。*开始使用 Kubernetes* 将帮助您了解如何将容器应用程序移入生产环境，并与现实世界的操作策略相结合的最佳实践和逐步演练。您将了解 Kubernetes 如何融入您的日常操作中，这可以帮助您为生产就绪的容器应用程序堆栈做好准备。

对 Docker 容器、一般软件开发和高级操作有一定了解会很有帮助。

# 惯例

在本书中，您将找到许多文本样式，用以区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。

文本中的代码词、文件夹名称、文件名、文件扩展名和路径名如下所示："执行一个简单的`curl`命令到 pod IP。"

URL 如下所示：

[`swagger.io/`](http://swagger.io/)

如果我们希望您用自己的值替换 URL 的一部分，则会显示如下：

`https://**<你的主机 IP>**/swagger-ui/`

资源定义文件和其他代码块设置如下：

```
apiVersion: v1
kind: Pod
metadata:
  name: node-js-pod
spec:
  containers:
  - name: node-js-pod
    image: bitnami/apache:latest
    ports:
    - containerPort: 80

```

当我们希望您用自己的值替换列表的一部分时，相关行或项目将以小于和大于符号之间的粗体设置：

```
subsets:
- addresses:
  - IP: <X.X.X.X>
  ports:
    - name: http
      port: 80
      protocol: TCP

```

任何命令行输入或输出都以如下形式编写：

```
$ kubectl get pods

```

新术语和重要单词以**粗体**显示。屏幕上看到的单词，例如在菜单或对话框中，会出现在文本中，如：“单击添加新按钮会将您移至下一个屏幕。”

文本中有几个地方涉及到键值对或屏幕上的输入对话框。在这些情况下，**键**或**输入标签**将以粗体显示，而***值***将以粗体斜体显示。例如：“在标有**超时**的框中输入***5s***。”

警告或重要注释会以此类框的形式显示。

提示和技巧以此类形式显示。

# 读者反馈

我们非常欢迎读者的反馈。让我们知道您对这本书的看法-您喜欢或不喜欢的内容。读者的反馈对我们非常重要，因为它有助于我们开发出真正能让您受益的标题。

要发送给我们一般反馈，只需发送电子邮件至`feedback@packtpub.com`，并在消息主题中提及书名。

如果有您擅长并且有兴趣撰写或贡献书籍的主题，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有很多东西可以帮助您充分利用您的购买。

# 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册后文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册到我们的网站。

1.  将鼠标指针悬停在顶部的 SUPPORT 选项卡上。

1.  单击“代码下载与勘误”。

1.  在搜索框中输入书名。

1.  选择您要下载代码文件的书籍。

1.  从下拉菜单中选择您购买本书的地方。

1.  单击“下载代码”。

下载文件后，请确保使用最新版本的以下操作解压或提取文件夹：

+   WinRAR / 7-Zip 适用于 Windows

+   Zipeg / iZip / UnRarX 适用于 Mac

+   7-Zip / PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Getting-Started-with-Kubernetes-Second-Edition`](https://github.com/PacktPublishing/Getting-Started-with-Kubernetes-Second-Edition)。我们还提供了来自丰富图书和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！

# 下载本书的彩色图像

我们还向您提供了一份 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从以下位置下载此文件：

[`www.packtpub.com/sites/default/files/downloads/GettingStartedwithKubernetesSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/GettingStartedwithKubernetesSecondEdition_ColorImages.pdf).

# 勘误

尽管我们已经竭尽全力确保内容的准确性，但错误确实会发生。如果您发现我们书籍中的错误 - 也许是文本或代码中的错误 - 如果您能向我们报告此错误，我们将不胜感激。通过这样做，您可以帮助其他读者免受挫折，帮助我们改进此书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告它们，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该书标题的勘误部分的任何现有勘误列表中。

要查看以前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索栏中输入书名。相关信息将显示在勘误部分下面。

# 盗版

互联网上盗版版权材料是所有媒体都面临的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何形式的非法复制，请立即向我们提供位置地址或网站名称，以便我们可以采取补救措施。

请通过`copyright@packtpub.com`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您帮助我们保护我们的作者以及我们为您提供有价值内容的能力。

# 问题

如果您对本书的任何方面有问题，请通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
