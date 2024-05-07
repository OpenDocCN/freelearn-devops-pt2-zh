# 前言

Kubernetes 是一个开源系统，自动化部署、扩展和管理容器化应用程序。如果您运行的不仅仅是一些容器，或者想要自动化管理容器，您就需要 Kubernetes。本书重点介绍了如何通过高级管理 Kubernetes 集群。

本书首先解释了 Kubernetes 架构背后的基本原理，并详细介绍了 Kubernetes 的设计。您将了解如何在 Kubernetes 上运行复杂的有状态微服务，包括水平 Pod 自动缩放、滚动更新、资源配额和持久存储后端等高级功能。通过真实的用例，您将探索网络配置的选项，并了解如何设置、操作和排除各种 Kubernetes 网络插件。最后，您将学习自定义资源开发和在自动化和维护工作流中的利用。本书还将涵盖基于 Kubernetes 1.10 发布的一些额外概念，如 Promethus、基于角色的访问控制和 API 聚合。

通过本书，您将了解从中级到高级水平所需的一切。

# 本书适合对象

本书适用于具有 Kubernetes 中级知识水平的系统管理员和开发人员，现在希望掌握其高级功能。您还应该具备基本的网络知识。这本高级书籍为掌握 Kubernetes 提供了一条路径。

# 充分利用本书

为了跟随每一章的示例，您需要在您的计算机上安装最新版本的 Docker 和 Kubernetes，最好是 Kubernetes 1.10。如果您的操作系统是 Windows 10 专业版，您可以启用 hypervisor 模式；否则，您需要安装 VirtualBox 并使用 Linux 客户操作系统。

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以从以下网址下载：[`www.packtpub.com/sites/default/files/downloads/MasteringKubernetesSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringKubernetesSecondEdition_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“让我们使用`get nodes`来检查集群中的节点。”

代码块设置如下：

```
type Scheduler struct { 
    config *Config 
} 
```

任何命令行输入或输出都以以下方式编写：

```
> kubectl create -f candy.yaml
candy "chocolate" created 
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“让我们点击 kubedns pod。”

警告或重要提示会以这种方式出现。提示和技巧会以这种方式出现。
