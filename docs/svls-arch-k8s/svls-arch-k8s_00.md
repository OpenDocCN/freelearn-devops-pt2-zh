# 前言

## 关于

本节简要介绍了作者、本书的覆盖范围、开始所需的技术技能，以及完成所有包含的活动和练习所需的硬件和软件要求。

## 关于本书

Kubernetes 已经确立了自己作为容器管理、编排和部署的标准平台。通过学习 Kubernetes，您将能够通过实施**函数即服务（FaaS）**模型来设计自己的无服务器架构。

在对无服务器架构和各种 Kubernetes 概念进行加速、实践性概述之后，您将涵盖真实开发人员面临的各种真实开发挑战，并探索克服这些挑战的各种技术。您将学习如何创建可投入生产的 Kubernetes 集群，并在其上运行无服务器应用程序。您将了解 Kubernetes 平台和无服务器框架（如 Kubeless、Apache OpenWhisk 和 OpenFaaS）如何提供您在 Kubernetes 上开发无服务器应用程序所需的工具。您还将学习如何为即将到来的项目选择适当的框架。

通过本书，您将具备技能和信心，能够利用 Kubernetes 的强大和灵活性设计自己的无服务器应用程序。

### 关于作者

**Onur Yılmaz**是一家跨国企业软件公司的高级软件工程师。他是一名持有认证的 Kubernetes 管理员（CKA），并致力于 Kubernetes 和云管理系统。他是 Docker、Kubernetes 和云原生应用等尖端技术的热情支持者。他在工程领域拥有一个硕士学位和两个学士学位。

**Sathsara Sarathchandra**是一名 DevOps 工程师，具有在云端和本地构建和管理基于 Kubernetes 的生产部署的经验。他拥有 8 年以上的经验，曾在从小型初创公司到企业的多家公司工作。他是一名持有认证的 Kubernetes 管理员（CKA）和认证的 Kubernetes 应用开发者（CKAD）。他拥有工商管理硕士学位和计算机科学学士学位。

### 学习目标

通过本书，您将能够：

+   使用 Minikube 在本地部署 Kubernetes 集群

+   使用 AWS Lambda 和 Google Cloud Functions

+   在云中创建、构建和部署由无服务器函数生成的网页。

+   创建在虚拟 kubelet 硬件抽象上运行的 Kubernetes 集群

+   创建、测试、排除 OpenFass 函数

+   使用 Apache OpenWhisk 操作创建一个示例 Slackbot

### 受众

本书适用于具有关于 Kubernetes 的基本或中级知识并希望学习如何创建在 Kubernetes 上运行的无服务器应用程序的软件开发人员和 DevOps 工程师。那些希望设计和创建在云上或本地 Kubernetes 集群上运行的无服务器应用程序的人也会发现本书有用。

### 方法

本书提供了与无服务器开发人员在实际工作中与 Kubernetes 集群一起工作的相关项目示例。您将构建示例应用程序并解决编程挑战，这将使您能够应对大型、复杂的工程问题。每个组件都设计为吸引和激发您，以便您可以在实际环境中以最大的影响力保留和应用所学到的知识。通过完成本书，您将能够处理真实世界的无服务器 Kubernetes 应用程序开发。

### 硬件要求

为了获得最佳的学生体验，我们建议以下硬件配置：

+   处理器：Intel Core i5 或同等级别

+   内存：8 GB RAM（建议 16 GB）

+   硬盘：10 GB 可用空间

+   互联网连接

### 软件要求

我们还建议您提前安装以下软件：

+   Sublime Text（最新版本）、Atom IDE（最新版本）或其他类似的文本编辑应用程序

+   Git

### 额外要求

+   Azure 账户

+   Google 云账户

+   AWS 账户

+   Docker Hub 账户

+   Slack 账户

### 约定

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：

“将`hello-from-lambda`作为函数名称，`Python 3.7`作为运行时。”

新术语和重要单词以粗体显示。例如，屏幕上看到的单词，比如菜单或对话框中的单词，会以这样的方式出现在文本中：“打开 AWS 管理控制台，在**查找服务**搜索框中输入**Lambda**，然后单击**Lambda - Run Code without Thinking about Servers**。”

代码块设置如下：

```
import json
def lambda_handler(event, context):
    return {
        'statusCode': '200',
        'body': json.dumps({"message": "hello", "platform": "lambda"}),
        'headers': {
            'Content-Type': 'application/json',
        }
    }
```

### 安装和设置

在我们可以对数据进行出色处理之前，我们需要准备好最高效的环境。在这个简短的部分中，我们将看到如何做到这一点。以下是需要满足的先决条件：

+   Docker（17.10.0-ce 或更高版本）

+   像 Virtualbox、Parallels、VMWareFusion、Hyperkit 或 VMWare 这样的 Hypervisor。更多信息请参考此链接：[`kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor`](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor)

### 其他资源

本书的代码包也托管在 GitHub 上，网址为[`github.com/TrainingByPackt/Serverless-Architectures-with-Kubernetes`](https://github.com/TrainingByPackt/Serverless-Architectures-with-Kubernetes)。我们还有其他代码包来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！
