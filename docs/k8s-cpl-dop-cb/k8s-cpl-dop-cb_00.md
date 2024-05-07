# 第一章

前言

Kubernetes 是一个开源的容器编排平台，最初由谷歌开发，并于 2014 年向公众开放。它使得基于容器的复杂分布式系统的部署对开发人员来说更加简单。自诞生以来，社区已经围绕 Kubernetes 构建了一个庞大的生态系统，涵盖了许多开源项目。本书专门设计为快速帮助 Kubernetes 管理员和可靠性工程师找到合适的工具，并快速掌握 Kubernetes。本书涵盖了从在最流行的云和本地解决方案上部署 Kubernetes 集群到帮助您自动化测试并将应用程序移出到生产环境的配方。

*Kubernetes – 一个完整的 DevOps 食谱*为您提供了清晰的，逐步的说明，以成功安装和运行您的私有 Kubernetes 集群。它充满了实用的配方，使您能够使用 Kubernetes 的最新功能以及其他第三方解决方案，并实施它们。

# 本书的受众

本书面向开发人员、IT 专业人员、可靠性工程师和 DevOps 团队和工程师，他们希望使用 Kubernetes 在其组织中管理、扩展和编排应用程序。需要对 Linux、Kubernetes 和容器化有基本的了解。

# 本书涵盖的内容

第一章，*构建生产就绪的 Kubernetes 集群*，教你如何在不同的公共云或本地配置 Kubernetes 服务，使用当今流行的选项。

第二章，*在 Kubernetes 上操作应用程序*，教你如何在 Kubernetes 上使用最流行的生命周期管理选项部署 DevOps 工具和持续集成/持续部署（CI/CD）基础设施。

第三章，*构建 CI/CD 流水线*，教你如何从开发到生产构建、推送和部署应用程序，以及在过程中检测错误、反模式和许可问题的方法。

第四章，*在 DevOps 中自动化测试*，教你如何在 DevOps 工作流中自动化测试，加快生产时间，减少交付风险，并使用 Kubernetes 中已知的测试自动化工具检测服务异常。

第五章，*为有状态的工作负载做准备*，教您如何保护应用程序的状态免受节点或应用程序故障的影响，以及如何共享数据和重新附加卷。

第六章，*灾难恢复和备份*，教您如何处理备份和灾难恢复方案，以保持应用程序在生产中高可用，并在云提供商或基本 Kubernetes 节点故障期间快速恢复服务。

第七章，*扩展和升级应用程序*，教您如何在 Kubernetes 上动态扩展容器化服务，以处理服务的变化流量需求。

第八章，*Kubernetes 上的可观察性和监控*，教您如何监控性能分析的指标，以及如何监控和管理 Kubernetes 资源的实时成本。

第九章，*保护应用程序和集群*，教您如何将 DevSecOps 构建到 CI/CD 流水线中，检测性能分析的指标，并安全地管理秘密和凭据。

第十章，*Kubernetes 上的日志记录*，教您如何设置集群以摄取日志，以及如何使用自管理和托管解决方案查看日志。

# 为了充分利用本书

要使用本书，您需要访问计算机、服务器或云服务提供商服务，您可以在其中提供虚拟机实例。为了设置实验室环境，您可能还需要更大的云实例，这将需要您启用计费。

我们假设您正在使用 Ubuntu 主机（在撰写本文时为 18.04，代号 Bionic Beaver）；本书提供了 Ubuntu 环境的步骤。

| 本书涵盖的软件/硬件 | 操作系统要求 |
| --- | --- |
| GitLab、Jenkins X、OpenShift、Rancher、kops、cURL、Python、Vim 或 Nano、kubectl、helm | Ubuntu/Windows/macOS |

您将需要 AWS、GCP 和 Azure 凭据来执行本书中的一些示例。

**如果您使用本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库访问代码（链接在下一节中提供）。这样做将有助于避免与复制/粘贴代码相关的任何潜在错误。**

## 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择支持选项卡。

1.  单击代码下载。

1.  在搜索框中输入书名，并按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址为[`github.com/k8sdevopscookbook/src`](https://github.com/k8sdevopscookbook/src)和[`github.com/PacktPublishing/Kubernetes-A-Complete-DevOps-Cookbook`](https://github.com/PacktPublishing/Kubernetes-A-Complete-DevOps-Cookbook)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

## 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781838828042_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781838828042_ColorImages.pdf)。

## 代码实例

访问以下链接，查看代码运行的视频：

[`bit.ly/2U0Cm8x`](http://bit.ly/2U0Cm8x)

## 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如："将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。"

一块代码设置如下：

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

**粗体**：表示一个新术语，一个重要词，或者您在屏幕上看到的词。例如，菜单或对话框中的单词会以这种形式出现在文本中。这是一个例子：“从管理面板中选择系统信息。”

警告或重要说明会出现在这样的形式中。提示和技巧会出现在这样的形式中。

# 章节

在这本书中，您会经常看到几个标题（*准备工作*，*如何做*，*它是如何工作的*，*还有更多*，和*另请参阅*）。

为了清晰地说明如何完成一个食谱，使用以下章节。

## 准备工作

这一部分告诉您在食谱中可以期待什么，并描述如何设置任何所需的软件或初步设置。

## 如何做…

这一部分包含了遵循食谱所需的步骤。

## 它是如何工作的…

这一部分通常包括了对前一部分发生的事情的详细解释。

## 还有更多…

这一部分包括了有关食谱的额外信息，以使您对食谱更加了解。

## 另请参阅

这一部分为食谱提供了其他有用信息的链接。
