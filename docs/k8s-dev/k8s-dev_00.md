# 前言

发现自己不仅负责编写代码，还负责运行代码的责任越来越普遍。虽然许多公司仍然有一个运维组（通常更名为 SRE 或 DevOps），他们可以提供专业知识，但开发人员（就像你）经常被要求扩展自己的知识和责任范围。

将基础设施视为代码已经有一段时间了。几年前，我可能会描述边界为 Puppet 由运维人员使用，而 Chef 由开发人员使用。随着云的出现和发展，以及最近 Docker 的发展，所有这些都发生了变化。容器提供了一定程度的控制和隔离，以及非常吸引人的开发灵活性。当使用容器时，您很快就会发现自己想要一次使用多个容器，以实现责任的隔离和水平扩展。

Kubernetes 是一个由 Google 开源的项目，现在由云原生计算基金会托管。它展示了 Google 在容器中运行软件的经验，并使其对您可用。它不仅包括运行容器，还将它们组合成服务，水平扩展它们，以及提供控制这些容器如何相互交互以及如何向外界暴露的手段。

Kubernetes 提供了一个由 API 和命令行工具支持的声明性结构。Kubernetes 可以在您的笔记本电脑上使用，也可以从众多云提供商中利用。使用 Kubernetes 的好处是能够使用相同的一套工具和相同的期望，无论是在本地运行，还是在您公司的小型实验室，或者在任何数量的较大云提供商中运行。这并不完全是 Java 从过去的日子里承诺的一次编写，随处运行；更多的是，我们将为您提供一致的一套工具，无论是在您的笔记本电脑上运行，还是在您公司的数据中心，或者在 AWS、Azure 或 Google 等云提供商上运行。

这本书是您利用 Kubernetes 及其功能开发、验证和运行代码的指南。

这本书侧重于示例和样本，带您了解如何使用 Kubernetes 并将其整合到您的开发工作流程中。通过这些示例，我们关注您可能想要利用 Kubernetes 运行代码的常见任务。

# 这本书是为谁准备的

如果您是一名全栈或后端软件开发人员，对测试和运行您正在开发的代码感兴趣、好奇或被要求负责，您可以利用 Kubernetes 使该过程更简单和一致。如果您正在寻找 Node.js 和 Python 中面向开发人员的示例，以了解如何使用 Kubernetes 构建、测试、部署和运行代码，那么本书非常适合您。

# 本书涵盖内容

第一章《为开发设置 Kubernetes》涵盖了 kubectl、minikube 和 Docker 的安装，以及使用 minikube 运行 kubectl 来验证您的安装。本章还介绍了 Kubernetes 中节点、Pod、容器、ReplicaSets 和部署的概念。

第二章《在 Kubernetes 中打包您的代码》解释了如何将您的代码打包到容器中，以便在 Python 和 Node.js 中使用 Kubernetes 进行示例。

第三章《与 Kubernetes 中的代码交互》涵盖了如何在 Kubernetes 中运行容器，如何访问这些容器，并介绍了 Kubernetes 的 Services、Labels 和 Selectors 概念。

第四章《声明性基础设施》介绍了如何以声明性结构表达您的应用程序，以及如何扩展以利用 Kubernetes 的 ConfigMaps、Annotations 和 Secrets 概念。

第五章《Pod 和容器生命周期》探讨了 Kubernetes 中容器和 Pod 的生命周期，以及如何公开来自您的应用程序的钩子以影响 Kubernetes 运行您的代码，以及如何优雅地终止您的代码。

第六章《Kubernetes 中的后台处理》解释了 Kubernetes 中作业和 CronJob 的批处理概念，并介绍了 Kubernetes 如何处理持久性，包括持久卷、持久卷索赔和有状态集。

第七章《监控和指标》涵盖了 Kubernetes 中的监控，以及如何利用 Prometheus 和 Grafana 捕获和显示有关 Kubernetes 和您的应用程序的指标和简单仪表板。

第八章，*日志和跟踪*，解释了如何使用 ElasticSearch、FluentD 和 Kibana 在 Kubernetes 中收集日志，以及如何设置和使用 Jaeger 进行分布式跟踪。

第九章，*集成测试*，介绍了利用 Kubernetes 的测试策略，以及如何在集成和端到端测试中利用 Kubernetes。

第十章，*故障排除常见问题和下一步*，回顾了您在开始使用 Kubernetes 时可能遇到的一些常见痛点，并解释了如何解决这些问题，并概述了 Kubernetes 生态系统中一些可能对开发人员和开发流程感兴趣的项目。

# 为了充分利用本书

您需要满足以下软件和硬件要求：

+   Kubernetes 1.8

+   Docker 社区版

+   kubectl 1.8（Kubernetes 的一部分）

+   VirtualBox v5.2.6 或更高版本

+   minikube v0.24.1

+   MacBook 或 Linux 机器，内存为 4GB 或更多

# 下载示例代码文件

您可以从您在[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址如下：

+   [`github.com/kubernetes-for-developers/kfd-nodejs`](https://github.com/kubernetes-for-developers/kfd-nodejs)

+   [`github.com/kubernetes-for-developers/kfd-flask`](https://github.com/kubernetes-for-developers/kfd-flask)

+   [`github.com/kubernetes-for-developers/kfd-celery`](https://github.com/kubernetes-for-developers/kfd-celery)

如果代码有更新，将在现有的 GitHub 存储库中进行更新。

我们还有其他代码包来自我们丰富的书籍和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```
import signal
import sys
def sigterm_handler(_signo, _stack_frame):
sys.exit(0)
signal.signal(signal.SIGTERM, sigterm_handler) 

```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```
import signal
**import sys**
def sigterm_handler(_signo, _stack_frame):
sys.exit(0)
signal.signal(signal.SIGTERM, sigterm_handler) 
```

任何命令行输入或输出都是这样写的：

```
 kubectl apply -f simplejob.yaml 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。例如：“从管理面板中选择系统信息。”

警告或重要说明会显示为这样。提示和技巧显示为这样。
