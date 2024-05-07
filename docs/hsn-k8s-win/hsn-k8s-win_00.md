# 前言

从版本 1.14 开始，Kubernetes 带来了 2019 年最受期待的功能：对 Windows Server 容器工作负载的生产级支持。这是一个巨大的里程碑，使得所有严重依赖 Windows 技术的企业能够迁移到云原生技术。开发人员和系统运营商现在可以利用相同的工具和流水线来部署 Windows 和 Linux 工作负载，以类似的方式扩展它们，并进行高效的监控。从商业角度来看，Windows 容器的采用意味着比普通虚拟机更低的运营成本和更好的硬件利用率。

你手中拿着一本书，将指导你如何在 Microsoft Windows 生态系统中使用 Kubernetes 和 Docker 容器 - 它涵盖了混合 Windows/Linux Kubernetes 集群部署，并使用 Windows 客户端机器处理集群操作。由于 Windows 在 Kubernetes 中的支持是一个相当新的概念，你可以期待官方文档和指南仍然很少。在这本书中，我们的目标是系统化你关于涉及 Windows 的 Kubernetes 场景的知识。我们的目标是创建 Kubernetes 在 Windows 上的终极指南。

# 这本书适合谁

这本书的主要受众是需要将 Windows 容器工作负载整合到他们的 Kubernetes 集群中的 Kubernetes DevOps 架构师和工程师。如果你是 Windows 应用程序（特别是.NET Framework）开发人员，而且你还没有使用过 Kubernetes，这本书也适合你！除了关于部署混合 Windows/Linux Kubernetes 集群的策略，我们还涵盖了 Kubernetes 背后的基本概念以及它们如何映射到 Windows 环境。如果你有兴趣将现有的.NET Framework 应用程序迁移到在 Kubernetes 上运行的 Windows Server 容器，你肯定会找到如何解决这个问题的指导。

# 这本书涵盖了什么

第一章，*创建容器*，描述了目前在 Linux 和 Windows 操作系统中使用的不同容器类型。本章的主要目标是演示如何构建一个示例 Windows 容器，运行它，并执行基本操作。

第二章《在容器中管理状态》讨论了管理和持久化容器化应用程序状态的可能方法，并解释了如何在容器上挂载本地和云存储卷（Azure Files SMB 共享），以便在 Windows 容器上运行像 MongoDB 这样的集群数据库引擎。

第三章《使用容器镜像》专注于容器镜像，这是分发容器化应用程序的标准方式。本章的目标是演示如何使用 Docker Hub 和 Azure 容器注册表，以及如何在部署流水线中安全地交付容器镜像。

第四章《Kubernetes 概念和 Windows 支持》使您熟悉了核心 Kubernetes 服务，如 kubelet、kube-proxy 和 kube-apiserver，以及最常用的 Kubernetes 对象，如 Pod、Service、Deployment 和 DaemonSet。您将了解为什么 Kubernetes 中的 Windows 支持很重要，以及当前关于 Windows 容器和 Windows 节点的限制是什么。我们还将重点放在为不同用例创建简单的开发 Kubernetes 集群上。

第五章《Kubernetes 网络》描述了 Kubernetes 的网络模型和可用的 Pod 网络解决方案。您将学习如何为具有 Windows 节点的 Kubernetes 集群选择最合适的网络模式。

第六章《与 Kubernetes 集群交互》展示了如何从 Windows 机器使用 kubectl 与 Kubernetes 集群进行交互和访问。例如，我们将展示如何与本地开发集群一起工作，以及最常见和有用的 kubectl 命令是什么。

第七章《部署混合本地 Kubernetes 集群》演示了如何处理虚拟机的供应和部署混合的 Windows/Linux Kubernetes 集群，其中包括 Linux 主节点/节点和 Windows 节点。本地部署是最通用的部署类型，因为它可以使用任何云服务提供商或私人数据中心进行。

第八章《部署混合 Azure Kubernetes 服务引擎集群》概述了如何使用 AKS Engine 部署混合 Windows/Linux Kubernetes 集群的方法，并演示了一个示例部署一个 Microsoft IIS 应用程序。

第九章《部署您的第一个应用程序》演示了如何将一个简单的 Web 应用程序以命令式和声明式的方式部署到 Kubernetes，并讨论了管理在 Kubernetes 中运行的应用程序的推荐方式。我们还将介绍如何专门在 Windows 节点上调度 Pod，并如何扩展在 Kubernetes 上运行的 Windows 应用程序。

第十章《部署 Microsoft SQL Server 2019 和 ASP.NET MVC 应用程序》描述了如何将在 Windows 容器中运行的 ASP.NET MVC 实现的样本投票应用程序部署到 AKS Engine 集群，以及 Microsoft SQL Server 2019（在 Linux 容器中运行）。您还将了解如何使用 Visual Studio 远程调试器调试在 Kubernetes 中运行的.NET 应用程序。

第十一章《配置应用程序使用 Kubernetes 功能》描述了如何实现和配置 Kubernetes 的更高级功能，包括命名空间、ConfigMaps 和 Secrets、持久存储、健康和就绪检查、自动缩放和滚动部署。本章还展示了 Kubernetes 中的基于角色的访问控制（RBAC）的工作原理。

第十二章《Kubernetes 的开发工作流程》展示了如何将 Kubernetes 作为微服务开发的平台。您将学习如何使用 Helm 打包应用程序，以及如何使用 Azure Dev Spaces 改进开发体验。此外，本章还描述了如何在 Kubernetes 中运行的容器化应用程序中使用 Azure 应用程序洞察和快照调试器。

第十三章《保护 Kubernetes 集群和应用程序》涵盖了 Kubernetes 集群和容器化应用程序的安全性。我们将讨论 Kubernetes 的一般推荐安全实践以及 Windows 特定的考虑因素。

第十四章《使用 Prometheus 监控 Kubernetes 应用程序》着重于如何监控 Kubernetes 集群，特别是运行在 Windows 节点上的.NET 应用程序。您将学习如何使用 Prometheus Helm 图表部署完整的监控解决方案，以及如何配置它来监控您的应用程序。

第十五章《灾难恢复》讨论了备份 Kubernetes 集群和灾难恢复策略。主要重点是展示哪些组件需要备份以安全恢复集群，以及如何自动化这个过程。

第十六章《运行 Kubernetes 的生产考虑因素》是针对在生产环境中运行 Kubernetes 的一些建议性建议。

# 为了充分利用本书

建议具有一些关于 Docker 和 Kubernetes 的一般知识，但不是必需的。我们将在专门的章节中涵盖 Windows 上容器化和 Kubernetes 本身的基本概念。对于那些专注于将 Windows 应用程序部署到 Kubernetes 的章节，建议您具有.NET Framework，C#和 ASP.NET MVC 的基本经验。请注意，本书中每个指南和示例都有官方 GitHub 存储库中的对应物：[`github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows`](https://github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows)

在整本书中，您将需要自己的 Azure 订阅。您可以在这里阅读有关如何获取个人使用的有限免费帐户的更多信息：[`azure.microsoft.com/en-us/free/`](https://azure.microsoft.com/en-us/free/)。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Visual Studio Code，Docker Desktop，Kubectl，带 16 GB RAM 的 Azure CLI | Windows 10 Pro，企业版或教育版（1903 版或更高版本；64 位），Windows Server 2019，Ubuntu Server 18.04 |

**如果您使用本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库访问代码（链接在下一节中提供）。这样做将有助于避免与复制和粘贴代码相关的任何潜在错误。**

# 下载示例代码文件

您可以从您在[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)注册，将文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载”。

1.  在“搜索”框中输入书名，然后按照屏幕上的说明操作。

一旦文件下载完成，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码捆绑包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows`](https://github.com/PacktPublishing/Hands-On-Kubernetes-on-Windows)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码捆绑包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781838821562_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781838821562_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```
html, body, #map {
 height: 100%; 
 margin: 0;
 padding: 0
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目以粗体显示：

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

**粗体**：表示一个新术语、一个重要词或屏幕上看到的词。例如，菜单或对话框中的单词会在文本中以这种方式出现。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会以这种方式出现。提示和技巧会以这种方式出现。
