# 序言

Jenkins 是一个基于 Java 的持续集成（CI）服务器，支持在软件周期的早期发现缺陷。 由于插件数量迅速增长（目前超过 1,000 个），Jenkins 与许多类型的系统通信，构建和触发各种测试。

CI 涉及对软件进行小改动，然后构建和应用质量保证流程。 缺陷不仅出现在代码中，还出现在命名约定、文档、软件设计方式、构建脚本、将软件部署到服务器的过程等方面。 CI 迫使缺陷早日显现，而不是等待软件完全生产出来。 如果在软件开发生命周期的后期阶段发现了缺陷，那么处理过程将会更加昂贵。 一旦错误逃逸到生产环境中，修复成本将急剧增加。 估计捕捉缺陷的成本早期是后期的 100 到 1,000 倍。 有效地使用 CI 服务器，如 Jenkins，可能是享受假期和不得不加班英雄般拯救一天之间的区别。 而且你可以想象，在我作为一个有着对质量保证的渴望的高级开发人员的日常工作中，我喜欢漫长而乏味的日子，至少是对于关键的生产环境。

Jenkins 可以定期自动构建软件，并根据定义的标准触发测试，拉取结果并基于定义的标准失败。 通过构建失败早期降低成本，增加对所生产软件的信心，并有可能将主观流程转变为开发团队认为是公正的基于指标的进攻性流程。

Jenkins 不仅是一个 CI 服务器，它还是一个充满活力和高度活跃的社区。 开明的自我利益决定了参与。 有许多方法可以做到这一点：

+   参与邮件列表和 Twitter（[`wiki.jenkins-ci.org/display/JENKINS/Mailing+Lists`](https://wiki.jenkins-ci.org/display/JENKINS/Mailing+Lists)）。 首先，阅读帖子，随着你了解到需要什么，然后参与讨论。 持续阅读列表将带来许多合作机会。

+   改善代码并编写插件（[`wiki.jenkins-ci.org/display/JENKINS/Help+Wanted`](https://wiki.jenkins-ci.org/display/JENKINS/Help+Wanted)）。

+   测试 Jenkins，尤其是插件，并撰写 Bug 报告，捐赠你的测试计划。

+   通过编写教程和案例研究来改善文档。

# 这本书涵盖了什么内容

第一章，*维护 Jenkins*，描述了常见的维护任务，如备份和监视。 本章中的配方概述了适当维护的方法，进而降低了故障的风险。

第二章，*增强安全性*，详细介绍如何保护 Jenkins 的安全性以及启用单点登录（SSO）的价值。本章涵盖了许多细节，从为 Jenkins 设置基本安全性，部署企业基础设施（如目录服务）到部署自动测试 OWASP 十大安全性。

第三章，*构建软件*，审查了 Jenkins 与 Maven 构建的关系以及使用 Groovy 和 Ant 进行少量脚本编写。配方包括检查许可证违规、控制报告创建、运行 Groovy 脚本以及绘制替代度量。

第四章，*通过 Jenkins 进行沟通*，审查了针对不同目标受众（开发人员、项目经理以及更广泛的公众）的有效沟通策略。Jenkins 是一个有才华的沟通者，通过电子邮件、仪表板和谷歌服务的一大群插件通知您。它通过移动设备对您叫嚷，在您经过大屏幕时辐射信息，并通过 USB 海绵导弹发射器向您射击。

第五章，*利用度量改善质量*，探讨了源代码度量的使用。为了节省成本和提高质量，您需要尽早在软件生命周期中消除缺陷。Jenkins 测试自动化创建了一张度量的安全网。本章的配方将帮助您构建这个安全网。

第六章，*远程测试*，详细介绍了建立和运行远程压力测试和功能测试的方法。本章结束时，您将对 Web 应用程序和 Web 服务运行性能和功能测试。包括两种典型的设置方案。第一种是通过 Jenkins 将 WAR 文件部署到应用服务器。第二种是创建多个从节点，准备好将测试工作从主节点转移。

第七章，*探索插件*，有两个目的。第一个是展示一些有趣的插件。第二是审查插件的工作原理。

附录，*增进质量的流程*，讨论了本书中的配方如何支持质量流程，并指向其他相关资源。这将帮助您形成一个完整的图景，了解配方如何支持您的质量流程。

# 本书所需准备的内容

本书假设您已经在运行 Jenkins 实例。

要运行本书提供的配方，您需要以下软件：

**推荐：**

+   Maven 3 ([`maven.apache.org`](http://maven.apache.org))

+   Jenkins ([`jenkins-ci.org/`](http://jenkins-ci.org/))

+   Java 版本 1.8 ([`java.com/en/download/index.jsp`](http://java.com/en/download/index.jsp))

**可选的:**

+   VirtualBox ([`www.virtualbox.org/`](https://www.virtualbox.org/))

+   SoapUI ([`www.soapui.org`](http://www.soapui.org))

+   JMeter ([`jmeter.apache.org/`](http://jmeter.apache.org/))

**有帮助的:**

+   一个本地的 subversion 或 Git 仓库

+   首选的操作系统：Linux（Ubuntu）

    ### 注意

    请注意，您可以从 Jenkins GUI (`http://localhost:8080/configure`) 中安装不同版本的 Maven、Ant 和 Java。您不需要将这些作为操作系统的一部分安装。

安装 Jenkins 有许多方法：作为 Windows 服务安装，使用 Linux 的仓库管理功能（如`apt`和`yum`），使用 Java Web Start，或直接从 WAR 文件运行。您可以选择您感觉最舒适的方法。但是，您可以从 WAR 文件运行 Jenkins，使用命令行中的 HTTPS，指向自定义目录。如果任何实验出现问题，则可以简单地指向另一个目录并重新开始。

要使用此方法，首先将`JENKINS_HOME`环境变量设置为您希望 Jenkins 在其中运行的目录。接下来，运行类似以下命令的命令：

```
Java –jar jenkins.war –httpsPort=8443 –httpPort=-1

```

Jenkins 将开始在端口`8443`上通过 https 运行。通过设置`httpPort=-1`关闭 HTTP 端口，并且终端将显示日志信息。

您可以通过执行以下命令来请求帮助：

```
Java –jar jenkins.war –help

```

可以在[`wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins`](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins)找到更广泛的安装说明。

对于在 VirtualBox 中使用 Jenkins 设置虚拟镜像的更高级菜谱描述，您可以使用第一章 *维护 Jenkins* 中的 *使用测试 Jenkins 实例* 菜谱。

# 这本书是为谁准备的

本书适用于 Java 开发人员、软件架构师、技术项目经理、构建管理器以及开发或 QA 工程师。预期具有对软件开发生命周期的基本理解，一些基本的 Web 开发知识以及对基本应用服务器概念的熟悉。还假定具有对 Jenkins 的基本理解。

# 节

在本书中，您会发现几个经常出现的标题（准备就绪，如何做，它是如何工作的，还有更多以及另请参见）。

为了清晰地说明如何完成一个菜谱，我们使用以下各节。

## 准备就绪

本节告诉您可以在菜谱中期待什么，并描述了为菜谱设置任何软件或任何预备设置所需的步骤。

## 如何做…

本节包含了遵循菜谱所需的步骤。

## 它是如何工作的…

本节通常包含了对前一节中发生的事情的详细解释。

## 还有…

本节包含有关菜谱的其他信息，以使读者更加了解菜谱。

## 另请参见

本节提供了其他有用信息的相关链接。

# 约定

在本书中，您将找到一些区分不同类型信息的文本样式。 以下是这些样式的一些示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名显示如下："然后，特定于工作的配置将存储在子目录中的`config.xml`中。"

代码块设置如下：

```
<?xml version='1.0' encoding='UTF-8'?>
<org.jvnet.hudson.plugins.thinbackup.ThinBackupPluginImpl plugin="thinBackup@1.7.4">
<fullBackupSchedule>1 0 * *  7</fullBackupSchedule>
<diffBackupSchedule>1 1 * * *</diffBackupSchedule>
<backupPath>/data/jenkins/backups</backupPath>
<nrMaxStoredFull>61</nrMaxStoredFull>
<excludedFilesRegex></excludedFilesRegex>
<waitForIdle>false</waitForIdle>
<forceQuietModeTimeout>120</forceQuietModeTimeout>
<cleanupDiff>true</cleanupDiff>
<moveOldBackupsToZipFile>true</moveOldBackupsToZipFile>
<backupBuildResults>true</backupBuildResults>
<backupBuildArchive>true</backupBuildArchive>
<backupUserContents>true</backupUserContents>
<backupNextBuildNumber>true</backupNextBuildNumber>
<backupBuildsToKeepOnly>true</backupBuildsToKeepOnly>
</org.jvnet.hudson.plugins.thinbackup.ThinBackupPluginImpl>
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```
server {
  listen   80;
  server_name  localhost;
  access_log  /var/log/nginx/jenkins _8080_proxypass_access.log;
  error_log  /var/log/nginx/jenkins_8080_proxypass_access.log;
  location / {
    proxy_pass      http://127.0.0.1:7070/;
    include         /etc/nginx/proxy.conf;
  }
}
```

任何命令行输入或输出都将显示如下：

```
sudo apt-get install jenkins

```

**新术语**和**重要词汇**以粗体显示。 您在屏幕上看到的单词，例如菜单或对话框中的单词，将以此类似的形式显示在文本中：“单击**保存**。”

### 注意

警告或重要提示将显示在此框中。

### 提示

技巧和窍门会显示如此。

# 读者反馈

我们的读者反馈始终受欢迎。 请告诉我们您对本书的看法——您喜欢或不喜欢的方面。 读者反馈对我们非常重要，因为它帮助我们开发您真正会受益的标题。

要发送给我们一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在您的消息主题中提及书名。

如果您在某个主题上拥有专业知识，并且对编写或为书籍做贡献感兴趣，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的自豪拥有者，我们有一些东西可以帮助您充分利用您的购买。

## 下载示例代码

您可以从 [`www.packtpub.com`](http://www.packtpub.com) 的帐户中下载您购买的所有 Packt Publishing 书籍的示例代码文件。 如果您在其他地方购买了此书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册以直接通过电子邮件接收文件。

## 勘误

虽然我们已经尽了一切努力确保内容的准确性，但错误确实会发生。 如果您在我们的书中发现错误——可能是文本中的错误或代码中的错误——我们将不胜感激您向我们报告。 这样做可以帮助其他读者免受挫折，并帮助我们改进本书的后续版本。 如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，单击**勘误提交表**链接，并输入您的勘误详情。 一旦验证您的勘误，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的错误部分下的任何现有勘误列表中。

要查看先前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书名。所需信息将出现在**勘误表**部分下。

## 盗版

盗版互联网上的受版权保护的材料是各种媒体持续面临的问题。在 Packt，我们非常重视对我们的版权和许可的保护。如果你在互联网上发现我们作品的任何形式的非法副本，请立即提供给我们位置地址或网站名称，以便我们采取措施解决。

请通过 `<copyright@packtpub.com>` 联系我们，并附上怀疑盗版材料的链接。

我们感谢您帮助保护我们的作者和我们为您提供有价值内容的能力。

## 问题

如果你对本书的任何方面有问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
