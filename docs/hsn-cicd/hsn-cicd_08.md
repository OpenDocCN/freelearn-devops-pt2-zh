# 使用 Jenkins 构建管道

本章将详细介绍如何使用现有的 Jenkins 实例设置 Jenkins 蓝海以及如何使用 Docker 进行设置。我们将详细了解蓝海用户界面（UI），并讨论 Jenkins 经典视图和蓝海视图之间的区别。我们还将详细讨论 Pipeline Syntax，并简要讨论其用途，并解释两种不同类型的 Pipeline Syntax。

本章将涵盖以下主题：

+   Jenkins 2.0

+   Jenkins 管道

+   在 Jenkins 蓝海中导航

+   Pipeline Syntax

# 技术要求

本章需要基本了解如何与 Unix Shell 环境交互。我们还将简要讨论 Pipeline Syntax，因此如果您具有一些基本的编程技能，就能理解编程语言中关键字的用途。

# Jenkins 2.0

Jenkins 2.0 的设计方法和流程与 Jenkins 1.0 相比有所不同。不再使用自由风格的作业，而是使用了一种新的“领域特定语言”（DSL），这是 Groovy 编程语言的缩写形式。

管道视图在 Jenkins 1.0 中的功能也有所不同。管道阶段视图还帮助我们可视化管道中的各个阶段。

# 为什么要转移到 Jenkins 2.0？

那么，首先，为什么要转移到 Jenkins 2.0，而不是继续使用 Jenkins 1.0？Jenkins 经典视图被认为是混乱的，并且没有考虑到易用性。Jenkins 2.0 在更直观的方式上使用 Docker 镜像上做出了很大的推动。此外，新的 UI 包括了 Jenkins 管道编辑器，并通过引入管道视图改变了您查找构建的方式。新 UI 的目标是减少混乱，增加对使用 Jenkins 的团队中每个成员的清晰度。新 UI 还具有 GitHub 和 Bitbucket 集成以及 Git 集成。Jenkins 2.0 UI 本质上是您安装的一组插件，称为蓝海。

# 在现有实例上安装蓝海插件

如果在大多数平台上安装 Jenkins，您将不会默认安装所有相关插件的蓝海插件。您需要确保您正在运行 Jenkins 版本 2.7.x 或更高版本才能安装蓝海插件。

为了在 Jenkins 实例上安装插件，您必须具有通过基于矩阵的安全设置的管理员权限，任何 Jenkins 管理员也可以配置系统中其他用户的权限。

安装蓝海插件的步骤如下：

1.  确保您以具有管理员权限的用户登录

1.  从 Jenkins 主页或 Jenkins 经典版的仪表板上，点击仪表板左侧的“管理 Jenkins”

1.  接下来，在“管理 Jenkins”页面的中心，点击“管理插件”

1.  点击“可用”选项卡，并在过滤文本框中输入“蓝海”，以过滤插件列表，只显示名称和/或描述中包含“蓝海”的插件

请阅读第七章，*开发插件*，特别是*安装 Jenkins 插件*部分，以获取更多信息。

# 通过 Jenkins Docker 镜像安装蓝海插件

您需要确保已安装 Docker 才能获取 Jenkins CI Docker 镜像。

# Docker 先决条件

由于 Docker 利用操作系统的虚拟化技术，因此 Docker 的安装要求是特定的。

OS X 的要求是：

+   2010 年或更新型号的 Mac，带有英特尔的 MMU 虚拟化

+   OS X El Capitan 10.11 或更新版本

Windows 的要求是：

+   64 位 Windows

+   Windows 10 专业版、企业版或教育版（不是家庭版，也不是 Windows 7 或 8）来安装 Hyper-V

+   Windows 10 周年更新版或更高版本

+   访问您机器的 BIOS 以打开虚拟化

在您的操作系统上安装 Docker，请访问 Docker 商店（[`store.docker.com/search?type=edition&offering=community`](https://store.docker.com/search?type=edition&offering=community)）网站，并点击适合您的操作系统或云服务的 Docker 社区版框。按照他们网站上的安装说明进行安装。

通过使用 Windows 命令提示符或 OS X/Linux 终端应用程序检查 Docker 版本来确保 Docker 已安装。在命令行中运行以下命令：

![](img/71166b35-4c61-4c35-8da1-d93477645440.png)

注意这里我安装了 Docker 版本 18。

# 安装 Docker 镜像

要获取 Docker 镜像，您需要确保在 Docker Hub（[`hub.docker.com/`](https://hub.docker.com/)）上有一个帐户。一旦您在 Docker Hub 上有了帐户并且安装了 Docker，获取最新的 Jenkins CI Docker 镜像就很简单。

在 Windows 命令提示符或终端中运行以下命令：

![](img/75f86bc1-49d1-46fe-b27e-6cf0c368a2ed.png)

请注意，我已经拉取了`jenkinsci/blueocean` Docker 镜像，因此该命令没有从 Docker Hub 中拉取，而是打印出了一个 SHA 哈希校验和。这表明我已经拥有了`jenkinsci/blueocean`的最新 Docker 镜像。

接下来，您需要让 Jenkins Docker 容器运行起来，并且您需要在终端或命令提示符中运行以下命令：

![](img/1095a144-6ed5-4b66-8f19-dcd414e930c8.png)

您可以通过简单地创建一个为您执行此操作的 shell 脚本或创建一个别名来简化此过程。

以下是我在文本编辑器中创建的一个 shell 脚本：

![](img/ae753d6a-1d5e-4f9c-a5ee-e10d275e3071.png)

我有一个个人的`bin`目录，我在`~/bin`中存储所有个人脚本，然后我确保将其添加到`PATH`变量中。脚本文件名为`run-jenkinsci-blueocean`。我们需要确保该脚本是可执行的，通过发出以下命令：

```
 chmod +x run-jenkinsci-blueocean
```

然后我只需要运行`~/bin/run-jenkinsci-blueocean`命令。

您也可以在 Unix 中创建类似的别名：

```
# inside ~/.zshrc

alias runJenkinsDockerImage='docker run -u root jenkins-blueocean --rm -d -p 8080:8080 -p 50000:50000 -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean'
```

请注意，我在我的`.zshrc`文件中添加了这个 shell 别名，但您也可以将其添加到`.bashrc`文件中。

Windows 用户可以创建一个批处理文件或找到其他方法来使运行 Docker 命令更容易。

为了停止 Docker 容器，您可以运行以下命令：

```
docker ps -a
```

此命令将显示系统中所有正在运行的容器；您需要查看`Container ID`、`NAMES`列，并复制与 Docker 镜像`jenkinsci/blueocean`对应的 ID。最后，要停止容器，您需要运行以下命令：

```
docker stop jenkins-blueocean
```

请注意，因为我们在 shell 脚本中的`docker run`命令中使用了`--name jenkins-blueocean`选项，Docker 创建了一个名为`jenkins-blueocean`的容器；如果我们没有这样做，那么 Docker 将为我们创建一个容器的名称。当您在终端或命令提示符中发出`docker ps -a`命令时，您可以使用容器 ID 和名称来停止容器。

一旦 Jenkins 运行起来，您可以在这里访问：`http://localhost:8080`，您需要提供为管理员生成的默认密码来解锁 Jenkins。在第五章中，*Jenkins 的安装和基础知识*，我们跳过了安装建议插件的入门步骤，但这次我建议您在入门屏幕上安装建议的插件：

![](img/1317bc5f-f095-4bfa-af7a-1ccae703e3c4.png)

通过点击安装建议插件按钮，您将获得所有建议的插件和依赖插件，这将帮助您在新的 Jenkins 2.0 流程中使用流水线等功能。

# 访问 Blue Ocean Jenkins 视图

您需要确保点击打开 Blue Ocean 按钮，看起来类似于这样：

![](img/badaaee4-6cfc-4e8b-b903-2084093bdf24.png)

点击“打开蓝色海洋”按钮后，您将被重定向到此 URL：`http://localhost:8080/blue/organizations/jenkins/pipelines`。Jenkins UI 将看起来非常不同，并且行为也不同。

这是您将看到的初始屏幕，因为我们还没有创建任何流水线：

![](img/cfc6c32e-e7bd-417b-a6e9-af003b4b6d79.png)

我们将在接下来的部分中探索流水线语法以及如何在 Jenkins 2.0 UI 中进行导航。

# Jenkins 流水线

我们将使用 Jenkins 2.0 UI 创建我们的第一个流水线，并且还将使用内置到新 Jenkins 2.0 UI 中的流水线编辑器创建一个 Jenkinsfile。

# 创建 Jenkins 流水线

我们要做的第一步是点击“创建新流水线”按钮。您将被重定向到以下屏幕：

![](img/af4a51b2-db74-49c2-a980-cd22ecf1210e.png)

对于本章的目的，我们将使用我创建的现有 GitHub 存储库，但您也可以轻松使用 Bitbucket 和您自己托管在 GitHub 或 Bitbucket 上的代码。为了使其工作，您需要确保在 GitHub 上有一个帐户，如果没有，请确保注册 GitHub（[`github.com/`](https://github.com/)）。

# 为 GitHub 提供个人访问令牌

如果您在 GitHub 上还没有个人访问令牌，您将需要创建一个。请注意，在以下截图中，有一个名为“在此处创建访问密钥”的链接：

![](img/e8dd5456-9d1a-4bc4-8b5c-1b67f9370adf.png)

点击“在此处创建访问密钥”链接后，您将被重定向到以下 GitHub 页面：

![](img/ec54ab3d-fec2-4178-ad76-4be2a213ebf9.png)

您可以保持默认选项勾选，然后点击标题为“生成令牌”的绿色按钮。确保将此个人访问令牌保存在安全的地方，因为它只会显示一次；复制它，因为我们将需要它。您需要将访问令牌粘贴到“连接到 Github”输入框中，然后点击蓝色的“连接”按钮：

![](img/186e26ea-6f1d-4a97-b138-a35240b48b9c.png)

# 选择您的 GitHub 组织

您需要选择您所属的 GitHub 组织。在接下来的截图中，我选择了 GitHub 用户名组织`jbelmont`：

![](img/64d91c75-d3ce-4f41-8a33-536cfb691e72.png)

# 选择 GitHub 存储库

您需要做的最后一步是实际选择要创建 Jenkins 流水线的 GitHub 存储库。在这里的截图中，我输入了`cucumber-examples`并选择了下拉菜单。然后蓝色的“创建流水线”按钮被启用：

![](img/78944deb-ff37-401d-87e4-9e90f1642745.png)

# 使用流水线编辑器创建流水线

在我们选择的 GitHub 存储库中，没有现有的 Jenkinsfile，因此我们被重定向到流水线编辑器屏幕，我们可以在其中创建我们的第一个 Jenkinsfile：

![](img/856f8223-249a-4543-bf92-f846bd823ff5.png)

我们需要为 Node.js 和代理添加一个 Docker 镜像，看起来类似于这样：

![](img/ffc8dbc0-9985-4bdf-9a2b-12f4f9b5bf0e.png)

请注意，我们使用`-v`选项为 Docker 挂载数据卷提供了一个图像和参数。

接下来，我们点击灰色的加号按钮，然后我们将看到以下更改：

![](img/ff130c0e-3369-43b9-ae93-69768a0a9b47.png)

接下来，在为阶段命名后，我们点击蓝色的“添加步骤”按钮。对于这个演示，我们将选择“构建”：

![](img/ae789def-1b34-4b57-b7a3-0f78388c684f.png)

接下来，我们需要选择一个步骤选项。我们将选择标题为“Shell 脚本”的选项，这将安装我们所有的 Node.js 依赖项：

![](img/5e9ff84e-31b0-409b-9a97-959ccdd4fcd0.png)

接下来，我们输入一些要在我们的 Shell 脚本中运行的命令：

![](img/f1710434-bea4-44f4-8da5-6a476bea92ea.png)

接下来，我们将再次点击灰色的加号按钮，以向我们的流水线添加一个阶段，现在看起来类似于这样：

![](img/cff137da-a10d-4d95-a16f-e60953410f47.png)

接下来，我们将为此阶段输入一个名称，并且在本章中，我们将选择`Cucumber Tests`：

![](img/5a67c3d1-7812-4b5f-a45d-bf59d8c959d0.png)

接下来，我们为此阶段添加一个步骤，并且我们将再次选择 Shell 脚本作为选项：

![](img/0c2ff88c-ccd5-4f3b-bdad-0641f36dbdba.png)

最后，我们将点击保存按钮并提供提交消息，以便将此更改推送到我们的 GitHub 存储库：

![](img/ad40105c-ecd9-4bbc-935b-eeae45804f7e.png)

点击蓝色的“保存并运行”按钮后，Jenkinsfile 将合并到主分支中，并且管道将运行。

# 在 Jenkins Blue Ocean 中导航

在 Jenkins Blue Ocean 中，您习惯使用的一些视图在 Jenkins Blue Ocean 中不可用。Jenkins Blue Ocean 背后的主要概念是使 Jenkins 内部导航更加便捷，并通过更好的图标和页面导航改进 Jenkins UI。新 Jenkins UI 的许多灵感来自强调世界已经从功能性开发人员工具转向开发人员体验的《蓝海战略》一书，并且新 UI 致力于改善 Jenkins 的开发人员体验。

# 管道视图

以下屏幕截图描述了 Jenkins Blue Ocean 的管道视图。请注意，我们为两个不同的 GitHub 存储库创建了两个不同的管道。通过单击“新管道”按钮并添加个人 base64 ([`github.com/jbelmont/decode-jwt`](https://github.com/jbelmont/decode-jwt)) Golang 库来解码 JSON Web 令牌通过命令行工具创建了第二个管道：

![](img/de1715a6-7613-40ab-80fb-645ce0617b5b.png)

此列表将根据您在 Jenkins 实例中添加的管道数量而有所不同。请注意，您可以标记一个管道，并且有标有名称、健康、分支和 PR 的列。

# 管道详细视图

如果您点击实际管道，那么您将进入一个管道详细信息页面，其中包含有关特定管道中运行的所有阶段的所有详细信息。接下来的屏幕截图是 base64 管道：

![](img/345339a9-d519-4330-a758-8637b2e4363e.png)

# 管道构建视图

您可以单击管道视图中的每个节点，并查看该阶段完成的所有工作。在第一个屏幕截图中，我们点击“构建信息”节点，以查看在该特定阶段运行的命令，其中包括拉取 GitHub 存储库的最新副本并运行`go version`和`go fmt`命令：

![](img/457938ef-74a2-4e50-930e-5ec08ce97ba5.png)

请注意，第二个节点标记为“运行测试”，当我们点击该特定节点时，我们只看到`go test`命令，该命令在 Golang 中运行我们的单元测试用例：

![](img/ca2c83c1-7e91-46b6-8089-d96409e2e838.png)

管道视图的一大优点是，您可以更清晰地查看持续集成构建中每个阶段的更好布局的可视化效果。

# 管道阶段视图

如果您点击管道中的实际阶段，由`>`符号表示，它将向您显示一个下拉视图，其中包含该特定阶段的详细信息：

![](img/454eab55-7377-4988-bacd-029e3f56dd21.png)

请注意，这里我们点击了“运行测试”阶段，以查看报告，报告显示我们用 Golang 编写的单元测试用例已通过。

# Jenkins 管道中的其他视图

还有其他视图可供使用，例如拉取请求视图，它会显示所有打开的拉取请求以及分支视图：

![](img/eb5b0717-6ce9-4442-9ead-dcccc2cce6e7.png)

Jenkins Blue Ocean 视图仍在进行中，因此任何管理任务，例如添加插件和添加安全信息，仍然在 Jenkins 经典视图中完成。

# 管道语法

管道语法有两种形式（[`jenkins.io/doc/book/pipeline/syntax/#declarative-pipeline`](https://jenkins.io/doc/book/pipeline/syntax/#declarative-pipeline)）：

+   声明性管道

+   脚本管道

两种形式之间的区别在于，声明性管道语法旨在比脚本管道更简单。脚本管道语法是 DSL，遵循 Groovy 编程语言的语义。

# 管道编辑器

在*cucumber-examples*存储库中，我们使用管道编辑器创建了一个 Jenkinsfile。您实际上可以在不使用管道编辑器的情况下编写 Jenkinsfile，尽管我建议在调试管道脚本时使用它，因为编辑器具有一些很好的功能。

# Jenkinsfile

这里有管道编辑器为我们创建的实际管道语法。它使用了声明性管道语法，这个语法有几个要讨论的项目：

```
pipeline {
  agent {
    docker {
      args '-v /Users/jean-marcelbelmont/jenkins_data'
      image 'node:10-alpine'
    }

  }
  stages {
    stage('Build') {
      steps {
        sh '''node --version

npm install'''
      }
    }
    stage('Cucumber Tests') {
      steps {
        sh 'npm run acceptance:tests'
      }
    }
  }
}
```

# 管道关键字

所有有效的声明性管道必须被包含在管道块中，正如您在前面的 Jenkinsfile 中所看到的。

# 代理关键字

代理部分指定整个管道或特定阶段将在 Jenkins 环境中执行的位置，取决于代理部分的放置位置。该部分必须在管道块的顶层内定义，但阶段级别的使用是可选的。

# 阶段关键字

stages 关键字包含一个或多个阶段指令的序列；stages 部分是管道描述的大部分工作将被放置的地方。

# 管道语法文档

如果您有兴趣了解更多关于管道语法的信息，请查看文档([`jenkins.io/doc/book/pipeline/syntax/`](https://jenkins.io/doc/book/pipeline/syntax/))。

# 总结

本章讨论了如何在现有的 Jenkins 实例中使用 Jenkins Blue Ocean 视图以及如何使用 Docker 设置 Blue Ocean 视图。我们看了许多不同的 Jenkins Blue Ocean 视图，并讨论了它们与 Jenkins 经典视图之间的一些区别。我们还讨论了管道语法和 Jenkinsfile。下一章将介绍 Travis CI 的安装和基本用法。

# 问题

1.  如果您通过 Docker 安装 Jenkins，您可以使用 Blue Ocean 视图吗？

1.  在 Blue Ocean 视图中使用管道编辑器有什么用？

1.  Jenkins 经典视图和 Blue Ocean 视图之间有哪些区别？

1.  您能详细查看管道的每个阶段吗？

1.  Blue Ocean 视图能处理 Jenkins 中的管理任务吗？

1.  阶段语法是用来做什么的？

1.  声明性管道语法需要包装在管道块中吗？

# 进一步阅读

请查看*Packt Publishing*的书籍*扩展 Jenkins*([`www.amazon.com/dp/B015CYBP2A`](https://www.amazon.com/dp/B015CYBP2A))，以了解更多关于 Jenkins 插件的信息。
