# 前言

编写现代软件很困难，因为在软件交付中涉及许多团队，包括开发人员、质量保证、运维、产品所有者、客户支持和销售。需要有一个流程，通过该流程，软件的开发是以自动化的方式进行的。持续集成和持续交付的过程将有助于确保交付给最终用户的软件具有最高质量，并经过 CI/CD 流水线的一系列检查。在本书中，您将学习如何使用 Jenkins CI，以及如何编写自由风格脚本、插件，以及如何使用更新的 Jenkins 2.0 UI 和流水线。您将了解 Travis CI 的 UI、Travis CLI、高级日志记录和调试技术，以及 Travis CI 的最佳实践。您还将学习 Circle CI 的 UI、Circle CLI、高级日志记录和调试技术，以及 CircleCI 的最佳实践。在整本书中，我们将讨论诸如容器、安全性和部署等概念。

# 本书适合对象

本书适用于系统管理员、质量保证工程师、DevOps 和站点可靠性工程师。您应该了解 Unix 编程、基本编程概念和 Git 等版本控制系统。

# 本书涵盖内容

第一章，*自动化测试的 CI/CD*，介绍了自动化的概念，并解释了与手动流程相比自动化的重要性。

第二章，*持续集成的基础知识*，介绍了持续集成的概念，解释了软件构建是什么，并介绍了 CI 构建实践。

第三章，*持续交付的基础知识*，介绍了持续交付的概念，特别是解释了软件交付、配置管理、部署流水线和脚本编写的问题。

第四章，*CI/CD 的商业价值*，通过解释沟通问题来介绍 CI/CD 的商业价值，例如能够向团队成员传达痛点、在团队成员之间分享责任、了解您的利益相关者，并展示为什么 CI/CD 很重要。

第五章，*Jenkins 的安装和基础知识*，帮助您在 Windows、Linux 和 macOS 操作系统上安装 Jenkins CI。您还将学习如何在本地系统上运行 Jenkins 以及如何管理 Jenkins CI。

第六章，*编写自由风格脚本*，介绍了如何在 Jenkins 中编写自由风格脚本，以及如何配置 Jenkins 中的自由风格脚本，包括添加环境变量和调试自由风格脚本中的问题。

第七章，*开发插件*，解释了软件中插件的概念，如何使用 Java 和 Maven 创建 Jenkins 插件，并介绍了 Jenkins 插件生态系统。

第八章，*使用 Jenkins 构建流水线*，详细介绍了 Jenkins 2.0，并解释了如何在 Jenkins 2.0（Blue Ocean）中导航，还详细介绍了新的流水线语法。

第九章，*Travis CI 的安装和基础知识*，向您介绍了 Travis CI，并解释了 Travis CI 与 Jenkins CI 之间的区别。我们将介绍 Travis 生命周期事件和 Travis YML 语法。我们还将解释如何开始并在 GitHub 上设置。

第十章，*Travis CI CLI 命令和自动化*，向您展示如何安装 Travis CI CLI，详细解释 CLI 中的每个命令，展示如何在 Travis CI 中自动化任务，并解释如何使用 Travis API。

第十一章，“Travis CI UI 日志和调试”，详细解释了 Travis Web UI，并展示了 Travis CI 中日志和调试的高级技术。

第十二章，“CircleCI 的安装和基础知识”，帮助您在 Bitbucket 和 GitHub 上设置 CircleCI，并展示如何使用 CircleCI Web UI。我们还将解释 CircleCI YML 语法。

第十三章，“CircleCI CLI 命令和自动化”，帮助您安装 CircleCI CLI，并解释 CLI 中的每个命令。我们还将介绍 CircleCI 中的工作流程以及如何使用 CircleCI API。

第十四章，“CircleCI UI 日志和调试”，详细解释了作业日志，并展示了如何在 CircleCI 中调试缓慢的构建。我们还将介绍 CircleCI 中的日志记录和故障排除技术。

第十五章，“最佳实践”，涵盖了编写单元测试、集成测试、系统测试、CI/CD 中的验收测试的最佳实践，以及密码和秘密管理的最佳实践。我们还将介绍部署的最佳实践。

# 充分利用本书

为了充分利用本书，您需要熟悉 Unix 编程概念，比如使用 Bash shell、环境变量和 shell 脚本，并了解 Unix 的基本命令。您应该熟悉版本控制的概念，知道提交是什么意思，并且需要了解如何使用 Git。您应该了解基本的编程语言概念，因为我们将使用诸如 Golang、Node.js 和 Java 之类的语言，这些语言将作为我们在 CI/CD 流水线和示例中使用的构建语言。

本书不受操作系统限制，但是为了使用本书中的一些概念，您需要访问 Unix 环境和命令。因此，如果您使用 Windows，最好安装 Git Bash ([`git-scm.com/downloads`](https://git-scm.com/downloads))和/或 Ubuntu 子系统。您需要在系统中安装 Git ([`git-scm.com/downloads`](https://git-scm.com/downloads))、Docker ([`docs.docker.com/install/`](https://docs.docker.com/install/))、Node.js ([`nodejs.org/en/download/`](https://nodejs.org/en/download/))、Golang ([`golang.org/dl/`](https://golang.org/dl/))和 Java ([`java.com/en/download/`](https://java.com/en/download/))。最好安装文本编辑器，如 Visual Studio Code ([`code.visualstudio.com/download`](https://code.visualstudio.com/download))和终端控制台应用程序。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本的解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，网址是[`github.com/PacktPublishing/Hands-On-Continuous-Integration-and-Delivery`](https://github.com/PacktPublishing/Hands-On-Continuous-Integration-and-Delivery)，在 README 部分，您可以找到按章节划分的所有代码文件的链接。如果代码有更新，链接将在现有的 GitHub 存储库中更新。

我们还有来自我们丰富的图书和视频目录的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“Chocolatey 安装说明可以在`chocolatey.org/install`找到。”

代码块设置如下：

```
{
  "@type": "env_vars",
  "@href": "/repo/19721247/env_vars",
  "@representation": "standard",
  "env_vars": [

  ]
} 
```

任何命令行输入或输出都以以下方式编写：

```
 Rules updated
Rules updated (v6) 
```

**粗体**：表示一个新术语、一个重要词或屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“点击继续，确保点击同意按钮。”

警告或重要说明会显示为这样。提示和技巧会显示为这样。
